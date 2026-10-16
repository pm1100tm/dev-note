# CloudFormation Parameters

## 왜 Parameters가 필요한가요?

템플릿에 인스턴스 타입이나 환경 이름을 직접 적어 두면, 값이 바뀔 때마다 템플릿 파일을 고쳐 다시 업로드해야 합니다.
개발 환경은 `t3.micro`, 운영 환경은 `t3.large`처럼 값만 다르고 구조는 같은 경우에도 템플릿을 여러
벌 관리해야 합니다.

**Parameters**는 스택을 만들거나 업데이트할 때 바깥에서 값을 넣을 수 있게 하는 템플릿의 입력값입니다.
함수의 매개변수처럼 템플릿은 그대로 두고, 실행할 때 값만 바꿔 같은 템플릿을 여러 환경과 팀에서 재사용할 수
있습니다.

```shell
템플릿(구조는 고정)  +  파라미터 값(실행 시 입력)
        ↓
CloudFormation이 값을 대입해 스택 생성
```

- 공식 문서:
  [CloudFormation template Parameters syntax](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/parameters-section-structure.html)

## 언제 파라미터로 만들까요?

판단 기준은 간단합니다. **"이 설정이 앞으로 바뀔 가능성이 있는가?"** 라고 질문해 보고, 그렇다면 파라미터로
만듭니다. 파라미터로 받은 값은 템플릿을 다시 업로드하지 않고도 스택 업데이트 때 바꿀 수 있습니다.

- 회사 안에서 여러 팀이 같은 템플릿을 재사용해야 할 때
- 키 페어 이름, VPC ID처럼 템플릿을 작성하는 시점에는 값을 알 수 없을 때
- 환경(dev, prod)에 따라 인스턴스 크기나 개수가 달라질 때

반대로 리전별 AMI ID처럼 미리 정해진 값 중에서 자동으로 골라야 하는 경우에는
[Mappings](CLOUD_FORMATION_007_mappings.md)가 더 적합합니다.

## 파라미터 타입

`Type`은 파라미터에서 유일한 필수 속성입니다. 타입을 지정하면 CloudFormation이 입력값을 미리
검증하므로 잘못된 값으로 배포가 시작되는 일을 줄일 수 있습니다.

| 타입                              | 입력 예시                     | 설명                                                            |
| --------------------------------- | ----------------------------- | --------------------------------------------------------------- |
| `String`                          | `my-app`                      | 일반 문자열입니다.                                              |
| `Number`                          | `8080`                        | 숫자로 검증하지만, 템플릿 안에서 참조하면 문자열로 쓰입니다.    |
| `List<Number>`                    | `80,443`                      | 쉼표로 구분한 숫자 목록입니다.                                  |
| `CommaDelimitedList`              | `dev,test,prod`               | 쉼표로 구분한 문자열 목록입니다.                                |
| AWS 전용 타입                     | `AWS::EC2::KeyPair::KeyName`  | 계정·리전에 실제로 있는 값인지 검증합니다.                      |
| `List<AWS 전용 타입>`             | `List<AWS::EC2::Subnet::Id>`  | 실제로 있는 값의 목록입니다.                                    |
| Systems Manager 파라미터 타입     | `AWS::SSM::Parameter::Value<String>` | Parameter Store에 저장된 값을 가져옵니다.                 |

### AWS 전용 타입

`AWS::EC2::KeyPair::KeyName`, `AWS::EC2::VPC::Id`,
`AWS::EC2::Subnet::Id` 같은 타입은 현재 계정과 리전에 실제로 존재하는 값만 받습니다. 콘솔에서는
직접 입력하는 대신 드롭다운 목록에서 고를 수 있어 오타를 막아 줍니다.

### Systems Manager 파라미터 타입

`AWS::SSM::Parameter::Value<AWS::EC2::Image::Id>` 타입을 쓰면 파라미터 값으로
Parameter Store의 키 이름을 넘기고, CloudFormation이 스택 생성·업데이트 시점의 최신 값을
읽어 옵니다. AWS가 공개하는 최신 Amazon Linux AMI ID 경로를 기본값으로 두면 리전마다 AMI ID를
따로 관리하지 않아도 됩니다.

```yaml
Parameters:
  LatestAmiId:
    Type: AWS::SSM::Parameter::Value<AWS::EC2::Image::Id>
    Default: /aws/service/ami-amazon-linux-latest/al2023-ami-kernel-default-x86_64
```

- `SecureString` 형식의 Parameter Store 값은 이 파라미터 타입으로 받을 수 없습니다. 민감한
  값은 아래 "NoEcho와 민감한 값"에서 설명하는 동적 참조를 사용합니다.
- 공식 문서:
  [CloudFormation-supplied parameter types](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/cloudformation-supplied-parameter-types.html)

## 파라미터 설정 속성

| 속성                      | 적용 타입                       | 역할                                                    |
| ------------------------- | ------------------------------- | ------------------------------------------------------- |
| `Description`             | 전체                            | 콘솔 입력 화면에 보이는 설명입니다.                     |
| `Default`                 | 전체                            | 값을 넣지 않았을 때 쓸 기본값입니다.                    |
| `AllowedValues`           | `String`, `Number`, 목록 타입   | 허용할 값의 목록입니다. 콘솔에서는 선택 상자로 보입니다. |
| `AllowedPattern`          | `String`, `CommaDelimitedList`  | 값이 맞춰야 할 정규 표현식입니다.                       |
| `MinLength` / `MaxLength` | `String`                        | 문자열 길이의 최솟값과 최댓값입니다.                    |
| `MinValue` / `MaxValue`   | `Number`                        | 숫자 범위의 최솟값과 최댓값입니다.                      |
| `ConstraintDescription`   | 전체                            | 제약 조건을 어겼을 때 보여 줄 안내 문구입니다.          |
| `NoEcho`                  | 전체                            | `true`이면 콘솔·CLI·API 조회 결과에서 값을 `*****`로 가립니다. |

`Default`가 없는 파라미터는 스택을 만들 때 반드시 값을 넣어야 하며, 넣지 않으면 스택 생성이 실패합니다.

### 예시: AllowedValues와 NoEcho

```yaml
Parameters:
  InstanceType:
    Description: EC2 인스턴스 타입
    Type: String
    Default: t3.micro
    AllowedValues:
      - t3.micro
      - t3.small
      - t3.medium
    ConstraintDescription: t3.micro, t3.small, t3.medium 중 하나를 선택해야 합니다.

  DBPassword:
    Description: 데이터베이스 관리자 비밀번호
    Type: String
    NoEcho: true
    MinLength: 8
    MaxLength: 41
    AllowedPattern: ^[a-zA-Z0-9]*$
```

- `InstanceType`은 목록에 없는 값을 넣으면 스택 생성 전에 검증 오류가 납니다.
- `DBPassword`는 스택 정보를 조회해도 값이 가려져 보입니다.

### NoEcho와 민감한 값

`NoEcho`는 **조회 결과에서 값을 가리는 기능**일 뿐, 값을 안전하게 저장하거나 암호화하는 기능은 아닙니다.
공식 문서에 따르면 `Metadata` 섹션, `Outputs` 섹션, 리소스의 `Metadata` 속성에 넣은 값은
`NoEcho`로 가려지지 않습니다.

비밀번호 같은 값은 파라미터로 직접 받기보다 Secrets Manager나 Parameter Store의
`SecureString`에 저장하고, 템플릿에서는 **동적 참조(dynamic reference)** 로 가져오는
방식을 권장합니다.

```yaml
Resources:
  MyDB:
    Type: AWS::RDS::DBInstance
    Properties:
      Engine: mysql
      DBInstanceClass: db.t3.micro
      AllocatedStorage: '20'
      # MyDbSecret은 Secrets Manager에 미리 만든 시크릿 이름(예시 값)입니다.
      MasterUsername: '{{resolve:secretsmanager:MyDbSecret:SecretString:username}}'
      MasterUserPassword: '{{resolve:secretsmanager:MyDbSecret:SecretString:password}}'
```

- `MyDbSecret`은 예시 이름이므로 실제 시크릿 이름으로 바꿉니다. 시크릿 안에 `username`,
  `password` 키가 있어야 합니다.
- 공식 문서:
  [Dynamic references](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/dynamic-references.html),
  [Secrets Manager 동적 참조](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/dynamic-references-secretsmanager.html)

## 파라미터 참조하기: `Ref`

정의한 파라미터는 `Ref` 함수로 참조하며, YAML에서는 짧은 형식인 `!Ref`를 주로 씁니다. 슬라이드에는
`Fn::Ref`로 적혀 있지만, 실제 함수 이름은 **`Ref`** 입니다. 다른 내장 함수와 달리 `Fn::`
접두사가 붙지 않습니다.

```yaml
Resources:
  WebServer:
    Type: AWS::EC2::Instance
    Properties:
      InstanceType: !Ref InstanceType   # 짧은 형식
      ImageId:
        Ref: LatestAmiId                # 전체 형식
```

`Ref`는 파라미터뿐 아니라 템플릿 안의 다른 리소스도 참조할 수 있습니다. 리소스를 참조하면 보통 그 리소스의 물리
ID가 반환되며, 자세한 내용은
[Intrinsic Functions](CLOUD_FORMATION_010_intrinsic_functions.md)에서
다룹니다.

## CLI로 파라미터 값 전달하기

```shell
# create-stack: ParameterKey/ParameterValue 형식
aws cloudformation create-stack \
  --stack-name my-web-stack \
  --template-body file://template.yaml \
  --parameters \
    ParameterKey=InstanceType,ParameterValue=t3.small \
    ParameterKey=EnvName,ParameterValue=dev

# deploy: Key=Value 형식
aws cloudformation deploy \
  --stack-name my-web-stack \
  --template-file template.yaml \
  --parameter-overrides InstanceType=t3.small EnvName=dev
```

- `my-web-stack`, `template.yaml`, `EnvName`은 예시 값입니다. 템플릿에 실제로
  정의한 파라미터 이름을 사용합니다.
- `update-stack`에서 기존 값을 그대로 쓰려면
  `ParameterKey=InstanceType,UsePreviousValue=true`처럼 지정합니다.

## Pseudo Parameters

**Pseudo Parameters**는 따로 선언하지 않아도 모든 템플릿에서 바로 쓸 수 있는, AWS가 미리
제공하는 파라미터입니다. 계정 ID나 리전처럼 배포 위치에 따라 달라지는 값을 하드코딩하지 않게 해 줍니다.

| Pseudo Parameter          | 반환 값 예시                                                    | 설명                                              |
| ------------------------- | --------------------------------------------------------------- | ------------------------------------------------- |
| `AWS::AccountId`          | `123456789012`                                                  | 스택이 생성되는 계정 ID                           |
| `AWS::Region`             | `ap-northeast-2`                                                | 스택이 생성되는 리전                              |
| `AWS::StackId`            | `arn:aws:cloudformation:ap-northeast-2:123456789012:stack/MyStack/...` | 스택 ID(ARN)                               |
| `AWS::StackName`          | `MyStack`                                                       | 스택 이름                                         |
| `AWS::NotificationARNs`   | `[arn:aws:sns:...:MyTopic]`                                     | 스택 이벤트 알림을 받는 SNS 토픽 ARN 목록         |
| `AWS::NoValue`            | 값 없음                                                         | `Fn::If`와 함께 써서 속성 자체를 제거             |
| `AWS::Partition`          | `aws`                                                           | 파티션(중국 리전은 `aws-cn`)                      |
| `AWS::URLSuffix`          | `amazonaws.com`                                                 | AWS 도메인 접미사                                 |

표의 계정 ID와 스택 이름은 예시 값입니다.

```yaml
Resources:
  LogBucket:
    Type: AWS::S3::Bucket
    Properties:
      # 예: my-web-stack-logs-123456789012-ap-northeast-2
      BucketName: !Sub '${AWS::StackName}-logs-${AWS::AccountId}-${AWS::Region}'
```

S3 버킷 이름은 전 세계에서 고유해야 하므로, 스택 이름과 계정 ID, 리전을 조합하면 이름 충돌을 줄일 수
있습니다. `AWS::NoValue`의 사용법은
[Conditions](CLOUD_FORMATION_009_conditions.md)에서 다룹니다.

- 공식 문서:
  [Pseudo parameters reference](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/pseudo-parameter-reference.html)

## 정리

- Parameters는 스택 생성·업데이트 시 값을 입력받아 템플릿을 재사용하게 해 줍니다.
- 앞으로 바뀔 가능성이 있는 설정은 파라미터로 만듭니다.
- AWS 전용 타입과 `AllowedValues`, `AllowedPattern` 같은 제약 조건으로 잘못된 입력을
  배포 전에 막습니다.
- 파라미터는 `Ref`(`!Ref`)로 참조합니다. `Fn::Ref`가 아니라 `Ref`가 정확한 이름입니다.
- `NoEcho`는 표시만 가리므로, 비밀 값은 Secrets Manager나 Parameter Store 동적
  참조를 사용합니다.
- Pseudo Parameters(`AWS::AccountId`, `AWS::Region` 등)는 선언 없이 바로
  사용할 수 있습니다.

## 시험 대비 문제

### 문제 1

개발자가 여러 리전과 계정에 배포할 템플릿을 작성하고 있습니다. IAM 정책의 리소스 ARN에 현재 계정 ID와 리전을
넣어야 하지만, 템플릿에 값을 하드코딩하거나 사용자에게 입력받고 싶지 않습니다. 가장 적절한 방법은 무엇입니까?

- A. `Parameters` 섹션에 `AccountId`와 `Region` 파라미터를 만들고 기본값을 지정한다.
- B. `Mappings` 섹션에 계정별·리전별 값을 모두 적어 둔다.
- C. `AWS::AccountId`와 `AWS::Region` Pseudo Parameters를 `!Sub`나
  `!Ref`로 참조한다.
- D. `Outputs` 섹션에서 계정 ID를 내보낸 뒤 `Fn::ImportValue`로 가져온다.

#### 정답

C

#### 해설

Pseudo Parameters는 선언하지 않아도 모든 템플릿에서 쓸 수 있으며, 스택이 배포되는 계정과 리전 값을
CloudFormation이 자동으로 채워 줍니다. A는 사용자가 값을 잘못 넣을 수 있고, B는 계정과 리전이 늘어날
때마다 템플릿을 고쳐야 합니다.

### 문제 2

데이터베이스 비밀번호를 CloudFormation 파라미터로 받으면서, 콘솔과 `describe-stacks` 조회
결과에 비밀번호가 보이지 않게 하려고 합니다. 파라미터에 설정해야 하는 속성은 무엇입니까?

- A. `AllowedPattern`
- B. `NoEcho`
- C. `ConstraintDescription`
- D. `Default`

#### 정답

B

#### 해설

`NoEcho: true`로 설정하면 스택 정보나 이벤트를 조회할 때 값이 `*****`로 가려집니다. 다만
`Outputs`나 `Metadata`에 넣은 값은 가려지지 않으며 값을 암호화해 주지도 않습니다. 그래서 실무에서는
Secrets Manager나 Parameter Store의 `SecureString`을 동적 참조로 가져오는 방식을
함께 권장합니다.
