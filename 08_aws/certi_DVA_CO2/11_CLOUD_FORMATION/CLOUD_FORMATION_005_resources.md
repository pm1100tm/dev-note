# CloudFormation Resources

## Resources 섹션이 하는 일

`Resources`는 템플릿에서 유일한 필수 섹션이며, 스택이 실제로 만들 AWS 리소스를 선언합니다. 다른
섹션(`Parameters`, `Mappings`, `Conditions`, `Outputs`)은 결국 이 리소스를 더
유연하게 만들거나 결과를 꺼내기 위한 보조 수단입니다.

리소스는 서로 참조할 수 있습니다. 예를 들어 EC2 인스턴스가 보안 그룹을 참조하면, CloudFormation은
참조 관계를 보고 보안 그룹을 먼저 만든 뒤 인스턴스를 만듭니다. 생성뿐 아니라 업데이트와 삭제 순서도
CloudFormation이 판단합니다.

- 공식 문서:
  [CloudFormation template Resources syntax](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/resources-section-structure.html)
  (확인일: 2026-10-13)

## 리소스 선언의 기본 형태

```yaml
Resources:
  MyBucket:                 # 논리 ID (Logical ID)
    Type: AWS::S3::Bucket   # 리소스 타입
    Properties:             # 리소스별 설정
      Tags:
        - Key: env
          Value: dev
```

### 논리 ID와 물리 ID

| 구분                      | 의미                                              | 예시                                 |
| ------------------------- | ------------------------------------------------- | ------------------------------------ |
| 논리 ID (Logical ID)      | 템플릿 안에서 리소스를 부르는 이름입니다.         | `MyBucket`, `WebServer`              |
| 물리 ID (Physical ID)     | 실제로 만들어진 리소스의 이름이나 ID입니다.       | `i-1234567890abcdef0`, 버킷 이름     |

- 논리 ID는 영문자와 숫자(`A-Za-z0-9`)만 사용할 수 있고, 템플릿 안에서 고유해야 합니다.
- 물리 ID는 리소스가 만들어진 뒤 정해집니다. 이름을 지정하지 않은 S3 버킷은 `스택이름-논리ID-고유값` 형태의
  이름이 자동으로 붙습니다.
- 위 예시의 인스턴스 ID는 공식 문서의 예시 값입니다.

### 리소스 타입 식별자

리소스 타입은 다음 형식을 따릅니다.

```shell
service-provider::service-name::data-type-name
```

| 타입                      | 의미                     |
| ------------------------- | ------------------------ |
| `AWS::EC2::Instance`      | EC2 인스턴스             |
| `AWS::EC2::SecurityGroup` | EC2 보안 그룹            |
| `AWS::S3::Bucket`         | S3 버킷                  |
| `AWS::IAM::Role`          | IAM 역할                 |

슬라이드에서는 지원 리소스 타입이 700개 이상이라고 설명하지만, 이 숫자는 새 서비스와 기능이 추가되면서 계속
늘어납니다. 정확한 목록은 공식 문서의 리소스 타입 레퍼런스에서 확인합니다.

- [AWS resource and property types reference](https://docs.aws.amazon.com/AWSCloudFormation/latest/TemplateReference/aws-template-resource-type-ref.html)

## 리소스 문서 읽는 법

모든 리소스를 외울 수는 없으므로, 필요한 리소스의 공식 문서를 찾아 읽는 방법을 익히는 것이 중요합니다.
[AWS::EC2::Instance 문서](https://docs.aws.amazon.com/AWSCloudFormation/latest/TemplateReference/aws-resource-ec2-instance.html)를
예로 들면 다음 순서로 읽습니다.

### Syntax: 전체 모양

JSON과 YAML로 사용할 수 있는 속성 목록이 나옵니다. 처음에는 필요한 속성만 골라 템플릿에 옮깁니다.

### Properties: 속성별 설명

각 속성마다 다음 세 가지를 확인합니다.

- **Required**: 반드시 넣어야 하는지(`Yes`), 넣지 않아도 되는지(`No`), 다른 속성에 따라
  달라지는지(`Conditional`)를 나타냅니다.
- **Type**: 값의 형식입니다. `String`, `Integer`, `Array of String`처럼
  표시됩니다.
- **Update requires**: 이 속성을 바꿔 스택을 업데이트할 때 리소스에 어떤 영향이 있는지 나타냅니다.

`Update requires`는 운영 환경에서 특히 중요합니다.

| 값                  | 의미                                                           | EC2 인스턴스 속성 예시   |
| ------------------- | -------------------------------------------------------------- | ------------------------ |
| `No interruption`   | 리소스 중단 없이 변경됩니다.                                   | `Tags`                   |
| `Some interruptions`| 재시작처럼 일시적인 중단이 생길 수 있습니다.                   | `InstanceType`, `UserData` |
| `Replacement`       | 새 리소스를 만들고 기존 리소스를 삭제합니다. 물리 ID가 바뀝니다. | `ImageId`                |

예를 들어 `ImageId`를 바꾸면 인스턴스가 교체되므로, 인스턴스 안에만 저장된 데이터는 사라질 수 있습니다.
업데이트 전에 [변경 세트](CLOUD_FORMATION_004_hands_on_first_stack.md)로
`Replacement` 여부를 확인하는 습관이 필요합니다.

- 공식 문서:
  [Update behaviors of stack resources](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/using-cfn-updating-stacks-update-behaviors.html)

### Return values: 다른 곳에서 꺼내 쓸 수 있는 값

- **Ref**: `!Ref 논리ID`가 돌려주는 값입니다. EC2 인스턴스는 인스턴스 ID를 돌려줍니다.
- **Fn::GetAtt**: `!GetAtt 논리ID.속성`으로 꺼낼 수 있는 속성 목록입니다. EC2 인스턴스는
  `PublicDnsName`, `PublicIp`, `PrivateIp`, `VpcId` 등을 제공합니다.

`Ref`가 무엇을 돌려주는지는 리소스 타입마다 다르므로, 반드시 해당 리소스 문서의 Return values를
확인합니다.

## 첫 실습 템플릿 분석

[첫 실습](CLOUD_FORMATION_004_hands_on_first_stack.md)에서 사용한 템플릿의
`Resources` 부분을 위 기준으로 다시 읽어 봅니다.

```yaml
Resources:
  WebServerSecurityGroup:
    Type: AWS::EC2::SecurityGroup
    Properties:
      GroupDescription: Allow SSH from my IP and HTTP from anywhere
      SecurityGroupIngress:
        - IpProtocol: tcp
          FromPort: 22
          ToPort: 22
          CidrIp: !Ref SshCidr
        - IpProtocol: tcp
          FromPort: 80
          ToPort: 80
          CidrIp: 0.0.0.0/0

  WebServer:
    Type: AWS::EC2::Instance
    Properties:
      ImageId: !Ref LatestAmiId
      InstanceType: t3.micro
      SecurityGroupIds:
        - !GetAtt WebServerSecurityGroup.GroupId
      UserData: !Base64 |
        #!/bin/bash
        dnf install -y httpd
        echo "<h1>Hello from CloudFormation</h1>" > /var/www/html/index.html
        systemctl enable --now httpd
```

- `GroupDescription`:
  [보안 그룹 문서](https://docs.aws.amazon.com/AWSCloudFormation/latest/TemplateReference/aws-resource-ec2-securitygroup.html)에서
  필수 속성이므로 반드시 넣었습니다.
- `SecurityGroupIngress`: 인바운드 규칙 목록입니다. YAML 배열로 규칙 두 개를 선언했습니다.
- `CidrIp: !Ref SshCidr`: 리소스가 아니라 파라미터를 참조합니다. `Ref`는 파라미터와 리소스
  모두에 사용할 수 있습니다.
- `ImageId`: 공식 문서에서 조건부 필수(`Conditional`)이며, 바꾸면 인스턴스가
  교체(`Replacement`)됩니다.
- `SecurityGroupIds`: 보안 그룹 ID 목록을 받습니다.
  `!GetAtt WebServerSecurityGroup.GroupId`로 같은 템플릿의 보안 그룹 ID를 넣었고,
  이 참조 때문에 보안 그룹이 먼저 만들어집니다.
- `UserData`: 공식 문서에서 Base64로 인코딩된 텍스트를 요구하므로 `!Base64` 함수로 감쌌습니다.

슬라이드 예제에 있는 Elastic IP는
[AWS::EC2::EIP 문서](https://docs.aws.amazon.com/AWSCloudFormation/latest/TemplateReference/aws-resource-ec2-eip.html)를
보고 `InstanceId: !Ref WebServer`처럼 인스턴스를 연결해 선언합니다.

## 자주 묻는 질문

### 리소스 개수를 동적으로 만들 수 있나요?

기본 템플릿 문법에는 반복문이 없어서, 리소스 3개가 필요하면 3번 선언해야 합니다. 동적으로 만들려면 템플릿을 배포
전에 변환하는 **매크로(Macro)** 와 `Transform` 섹션을 사용합니다.

AWS가 제공하는 `AWS::LanguageExtensions` 변환을 선언하면 `Fn::ForEach`로 목록 값만큼
리소스를 반복 생성할 수 있습니다.

```yaml
Transform: AWS::LanguageExtensions
Resources:
  Fn::ForEach::Buckets:
    - Name
    - [Logs, Backup]
    - ${Name}Bucket:
        Type: AWS::S3::Bucket
```

위 예시는 `LogsBucket`, `BackupBucket` 두 버킷을 선언한 것과 같습니다. 시험 범위에서는 "동적
생성은 매크로와 Transform으로 가능하다" 정도로 기억합니다.

- 공식 문서:
  [Fn::ForEach](https://docs.aws.amazon.com/AWSCloudFormation/latest/TemplateReference/intrinsic-function-reference-foreach.html)

### 모든 AWS 서비스를 지원하나요?

대부분 지원하지만, 새로 나온 기능이나 일부 서비스는 아직 리소스 타입이 없을 수 있습니다. 이때는 Lambda 함수로
원하는 작업을 수행하는 **Custom Resource**로 해결할 수 있습니다.
[Custom Resources](CLOUD_FORMATION_015_custom_resources.md)에서
다룹니다.

## 정리

- `Resources`는 필수 섹션이며, 리소스는 논리 ID, `Type`, `Properties`로 선언합니다.
- 리소스 타입은 `AWS::서비스::데이터타입` 형식이며, 지원 목록은 공식 레퍼런스에서 확인합니다.
- 리소스 문서에서는 Required, Type, Update requires, Return values를 확인합니다.
- `Replacement` 속성을 바꾸면 리소스가 새로 만들어지고 물리 ID가 바뀝니다.
- 동적 리소스 생성은 매크로와 `Transform`, 지원하지 않는 기능은 Custom Resource로 해결합니다.

## 시험 대비 문제

### 문제 1

CloudFormation 스택에 있는 EC2 인스턴스의 AMI(`ImageId`)를 새 버전으로 바꿔 스택을
업데이트하려고 합니다. 리소스 문서에서 `ImageId`의 `Update requires` 값은
`Replacement`입니다. 업데이트 시 어떤 일이 일어납니까?

- A. 기존 인스턴스를 그대로 두고 AMI 정보만 바뀐다.
- B. 기존 인스턴스를 재부팅한 뒤 AMI가 바뀐다.
- C. 새 인스턴스를 만들고 기존 인스턴스를 삭제하며, 인스턴스 ID가 바뀐다.
- D. 업데이트가 거부되어 스택을 삭제 후 다시 만들어야 한다.

#### 정답

C

#### 해설

`Replacement`는 새 물리 리소스를 만든 뒤 기존 리소스를 삭제하는 업데이트입니다. 그래서 물리 ID가
바뀌고, 기존 인스턴스에만 저장된 데이터는 사라질 수 있습니다. B는 `Some interruptions`, A는
`No interruption`에 가까운 설명입니다.

### 문제 2

개발자가 CloudFormation에서 아직 리소스 타입으로 지원하지 않는 외부 시스템 설정을 스택 생성 시 함께
처리하려고 합니다. 가장 적절한 방법은 무엇입니까?

- A. `Mappings` 섹션에 외부 시스템 설정 값을 적는다.
- B. Lambda 함수 기반의 Custom Resource를 정의한다.
- C. `Outputs` 섹션에서 외부 시스템 API를 호출한다.
- D. 지원되지 않으므로 CloudFormation으로는 처리할 수 없다.

#### 정답

B

#### 해설

Custom Resource를 사용하면 스택 생성·업데이트·삭제 시 CloudFormation이 Lambda 함수 등에
요청을 보내고, 함수가 원하는 작업을 수행합니다. `Mappings`는 고정 값 조회표이고, `Outputs`는
결과값을 내보내는 섹션이므로 외부 작업을 실행하지 않습니다.
