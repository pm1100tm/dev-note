# CloudFormation Custom Resources

## 왜 Custom Resource가 필요한가요?

CloudFormation은 대부분의 AWS 리소스를 지원하지만, 모든 작업을 리소스 타입으로 표현할 수는 없습니다.
예를 들어 다음 작업은 기본 리소스 타입만으로 처리하기 어렵습니다.

- 아직 CloudFormation이 지원하지 않는 AWS 리소스를 만드는 작업
- 온프레미스 시스템이나 외부(3rd party) 서비스에 설정을 등록하는 작업
- 스택 삭제 전에 S3 버킷을 비우는 것처럼, 생성·업데이트·삭제 시점에 스크립트를 실행하는 작업

Custom Resource는 이런 작업을 Lambda 함수나 SNS 토픽에 맡기고, 그 결과를 스택의 리소스처럼
다루게 해 주는 기능입니다.

- 공식 문서:
  [Create custom provisioning logic with custom resources](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/template-custom-resources.html)
  (확인일: 2026-10-13)

## 정의 방법

템플릿에서는 다음 두 타입 중 하나로 선언합니다.

- `Custom::<이름>`: 예) `Custom::EmptyBucket`. 용도가 이름에 드러나므로 권장합니다.
- `AWS::CloudFormation::CustomResource`

```yaml
Resources:
  MyCustomResource:
    Type: Custom::EmptyBucket
    Properties:
      ServiceToken: arn:aws:lambda:ap-northeast-2:123456789012:function:empty-bucket
      BucketName: my-example-bucket
```

- `ServiceToken`: CloudFormation이 요청을 보낼 대상입니다. Lambda 함수 ARN 또는
  SNS 토픽 ARN을 넣으며, **필수**입니다.
- Lambda 함수나 SNS 토픽은 스택과 **같은 리전**에 있어야 합니다.
- `ServiceToken` 외의 속성(`BucketName` 등)은 선택 입력값이며, 요청의
  `ResourceProperties`로 그대로 전달됩니다.
- 위 ARN의 리전, 계정 ID, 함수 이름과 버킷 이름은 예시 값입니다.

## 동작 흐름

```shell
스택 생성·업데이트·삭제
        ↓
CloudFormation → ServiceToken(Lambda 또는 SNS)으로 요청 전송
                 (RequestType: Create / Update / Delete, ResponseURL 포함)
        ↓
Lambda가 작업 수행 후 ResponseURL(S3 Pre-signed URL)로 SUCCESS 또는 FAILED 응답
        ↓
CloudFormation이 응답을 받고 스택 작업을 계속 진행
```

- 요청에는 `RequestType`, `ResourceProperties`, 응답을 보낼
  `ResponseURL`(S3 Pre-signed URL) 등이 들어 있습니다.
- 응답의 `Data`에 넣은 값은 템플릿에서 `Fn::GetAtt`로 읽을 수 있습니다.
- `FAILED`를 응답하거나 응답하지 않으면 스택 작업이 실패합니다.
- 응답 대기 시간의 기본값은 3600초(1시간)입니다. `ServiceTimeout` 속성으로 1~3600초 사이에서
  조정할 수 있습니다.

## 예시: 스택 삭제 시 S3 버킷 비우기

S3 버킷은 객체가 남아 있으면 삭제되지 않습니다. 그래서 스택 삭제가 `DELETE_FAILED`로 끝나는 경우가
많습니다. Custom Resource로 스택 삭제 시 Lambda가 먼저 버킷을 비우게 하면 이 문제를 해결할 수
있습니다.

> 주의: 이 예시는 스택 삭제 시 버킷의 모든 객체와 버전을 **복구할 수 없게
> 삭제**합니다. 보존해야 할 데이터가 있는 버킷에는 적용하지 말고,
> [DeletionPolicy](CLOUD_FORMATION_013_deletion_policy.md)의 `Retain`을 검토합니다.

### 전제 조건

- AWS CLI v2가 설치되어 있고 `aws sts get-caller-identity`로 자격 증명이 확인되어야
  합니다.
- IAM 역할을 만들 수 있는 권한이 필요합니다.

### 템플릿 파일 만들기

작업 디렉터리에 `empty-bucket.yaml` 파일을 만들고 다음 내용을 저장합니다.

```yaml
AWSTemplateFormatVersion: '2010-09-09'
Description: Custom Resource example - empty S3 bucket before deletion

Resources:
  DataBucket:
    Type: AWS::S3::Bucket

  EmptyBucketFunctionRole:
    Type: AWS::IAM::Role
    Properties:
      AssumeRolePolicyDocument:
        Version: '2012-10-17'
        Statement:
          - Effect: Allow
            Principal:
              Service: lambda.amazonaws.com
            Action: sts:AssumeRole
      ManagedPolicyArns:
        - arn:aws:iam::aws:policy/service-role/AWSLambdaBasicExecutionRole
      Policies:
        - PolicyName: EmptyBucket
          PolicyDocument:
            Version: '2012-10-17'
            Statement:
              - Effect: Allow
                Action:
                  - s3:ListBucket
                  - s3:ListBucketVersions
                Resource: !GetAtt DataBucket.Arn
              - Effect: Allow
                Action:
                  - s3:DeleteObject
                  - s3:DeleteObjectVersion
                Resource: !Sub '${DataBucket.Arn}/*'

  EmptyBucketFunction:
    Type: AWS::Lambda::Function
    Properties:
      Runtime: python3.13
      Handler: index.handler
      Timeout: 300
      Role: !GetAtt EmptyBucketFunctionRole.Arn
      Code:
        ZipFile: |
          import boto3
          import cfnresponse

          def handler(event, context):
              bucket_name = event['ResourceProperties']['BucketName']
              try:
                  if event['RequestType'] == 'Delete':
                      bucket = boto3.resource('s3').Bucket(bucket_name)
                      # 버전 관리 여부와 관계없이 모든 객체 버전과 삭제 마커를 지운다
                      bucket.object_versions.all().delete()
                  cfnresponse.send(event, context, cfnresponse.SUCCESS, {},
                                   physicalResourceId=f'empty-{bucket_name}')
              except Exception as e:
                  # 실패해도 반드시 응답해야 스택이 1시간 동안 멈추지 않는다
                  cfnresponse.send(event, context, cfnresponse.FAILED, {},
                                   physicalResourceId=f'empty-{bucket_name}',
                                   reason=str(e))

  EmptyBucketOnDelete:
    Type: Custom::EmptyBucket
    Properties:
      ServiceToken: !GetAtt EmptyBucketFunction.Arn
      ServiceTimeout: 600
      BucketName: !Ref DataBucket

Outputs:
  BucketName:
    Value: !Ref DataBucket
```

템플릿의 핵심은 다음과 같습니다.

- `EmptyBucketOnDelete`는 `!Ref DataBucket`으로 버킷을 참조하므로, 스택을 삭제할 때
  버킷보다 **먼저** 삭제됩니다. 이때 Lambda가 `Delete` 요청을 받아 버킷을 비웁니다.
- Create와 Update 요청에서는 아무 작업 없이 `SUCCESS`만 응답합니다.
- `physicalResourceId`를 고정값으로 보냅니다. 값이 바뀌면 CloudFormation이 업데이트를
  교체로 판단해 이전 리소스에 `Delete` 요청을 보내기 때문입니다.
- `cfnresponse` 모듈은 `ZipFile` 속성으로 코드를 템플릿 안에 직접 작성할 때만 제공됩니다. 코드를
  S3에 올려 배포하면 응답 전송 코드를 직접 작성해야 합니다.
- `Runtime` 값은 작성 시점의 예시입니다. 배포 전
  [Lambda 지원 런타임](https://docs.aws.amazon.com/lambda/latest/dg/lambda-runtimes.html)에서
  지원 중인 Python 버전인지 확인합니다.

- 공식 문서:
  [cfn-response module](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/cfn-lambda-function-code-cfnresponsemodule.html)

### 배포와 확인

`empty-bucket.yaml`이 있는 디렉터리에서 실행합니다. 템플릿이 IAM 역할을 만들므로
`--capabilities CAPABILITY_IAM`이 필요합니다. 이유는
[Service Role과 Capabilities](CLOUD_FORMATION_012_service_role_and_capabilities.md)에서
다룹니다.

```shell
aws cloudformation deploy \
  --stack-name cfn-empty-bucket-demo \
  --template-file empty-bucket.yaml \
  --capabilities CAPABILITY_IAM

# 생성된 버킷 이름 확인
BUCKET=$(aws cloudformation describe-stacks \
  --stack-name cfn-empty-bucket-demo \
  --query "Stacks[0].Outputs[?OutputKey=='BucketName'].OutputValue" \
  --output text)

# 테스트 객체 업로드
echo "hello" > hello.txt
aws s3 cp hello.txt "s3://${BUCKET}/hello.txt"
```

`deploy` 명령이 성공하면 스택 상태가 `CREATE_COMPLETE`가 되고,
`aws s3 ls "s3://${BUCKET}"`에서 `hello.txt`가 보여야 합니다.

### 정리와 검증

다음 명령은 스택과 버킷, 버킷 안의 테스트 객체를 모두 삭제합니다.

```shell
aws cloudformation delete-stack --stack-name cfn-empty-bucket-demo
aws cloudformation wait stack-delete-complete --stack-name cfn-empty-bucket-demo
```

`wait` 명령이 오류 없이 끝나면 버킷이 비워진 뒤 정상적으로 삭제된 것입니다. 실패하면
`aws cloudformation describe-stack-events --stack-name cfn-empty-bucket-demo`와
Lambda의 CloudWatch Logs에서 원인을 확인합니다.

## 실무 활용과 주의 사항

- Lambda 코드에서 예외가 나더라도 반드시 `SUCCESS` 또는 `FAILED`를 응답합니다. 응답이 없으면
  스택이 `ServiceTimeout`(기본 1시간) 동안 진행 중 상태로 멈춥니다.
- Lambda 실행 역할에는 실제 작업에 필요한 최소 권한만 부여합니다.
- VPC 안의 Lambda는 S3 Pre-signed URL로 응답을 보낼 수 있도록 네트워크 경로를 확인합니다.
- 응답 `Data`에 민감한 값을 넣으면 `DescribeStackEvents` 같은 API로 노출될 수 있으므로
  `NoEcho`를 사용합니다.
- 반복해서 쓰는 외부 리소스라면 Custom Resource 대신 CloudFormation Registry에 리소스
  타입을 등록하는 방법도 있습니다.

## 정리

- Custom Resource는 CloudFormation이 직접 처리하지 못하는 작업을 Lambda 함수나 SNS
  토픽에 맡기는 기능입니다.
- 타입은 `Custom::<이름>`(권장) 또는 `AWS::CloudFormation::CustomResource`로
  선언합니다.
- `ServiceToken`은 필수이며, 같은 리전의 Lambda 또는 SNS ARN을 지정합니다.
- 처리 결과는 S3 Pre-signed URL(`ResponseURL`)로 응답하며, 응답이 없으면 기본 1시간 후
  실패합니다.
- 대표 사례는 스택 삭제 전에 S3 버킷을 비우는 작업입니다.

## 시험 대비 문제

### 문제 1

CloudFormation 스택에 S3 버킷이 포함되어 있습니다. 스택을 삭제할 때마다 버킷에 객체가 남아 있어
`DELETE_FAILED`가 발생합니다. 스택 삭제 과정에서 자동으로 버킷을 비우고 싶습니다. 가장 적절한 방법은
무엇입니까?

- A. 버킷에 `DeletionPolicy: Retain`을 지정한다.
- B. Lambda 함수를 사용하는 Custom Resource를 만들어 `Delete` 요청 시 버킷의 객체를
  삭제한다.
- C. 스택에 Termination Protection을 활성화한다.
- D. 버킷에 `DeletionPolicy: Snapshot`을 지정한다.

#### 정답

B

#### 해설

Lambda 기반 Custom Resource는 스택 삭제 시 `Delete` 요청을 받아 버킷을 비우는 스크립트를
실행할 수 있습니다. A는 버킷을 지우지 않고 남겨 두는 설정이며, D의 `Snapshot`은 S3 버킷이 지원하지
않습니다. C는 스택 삭제 자체를 막습니다.

### 문제 2

개발자가 템플릿에 `Custom::Inventory` 타입의 Custom Resource를 선언하고 있습니다.
CloudFormation이 생성·업데이트·삭제 요청을 보낼 대상을 지정하려면 어떤 속성이 필요합니까?

- A. `ServiceToken`에 같은 리전의 Lambda 함수 ARN을 지정한다.
- B. `ServiceToken`에 다른 리전의 SQS 큐 URL을 지정한다.
- C. `Handler`에 Lambda 함수 이름을 지정한다.
- D. `DependsOn`에 Lambda 함수의 논리 ID를 지정한다.

#### 정답

A

#### 해설

Custom Resource의 필수 속성은 `ServiceToken`이며, 요청을 받을 Lambda 함수 ARN 또는
SNS 토픽 ARN을 지정합니다. 대상은 스택과 같은 리전에 있어야 하고, SQS는 지원 대상이 아닙니다.
