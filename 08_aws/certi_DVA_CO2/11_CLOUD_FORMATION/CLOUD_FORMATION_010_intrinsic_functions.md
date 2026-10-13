# CloudFormation Intrinsic Functions

## 왜 내장 함수가 필요한가요?

템플릿에 모든 값을 직접 적어 두면, 리소스끼리 서로의 ID를 참조하거나 리전에 따라 값을 바꾸는 일을 할 수
없습니다. 예를 들어 EC2 인스턴스에 보안 그룹을 연결하려면 보안 그룹 ID가 필요한데, 이 ID는 스택을 만들어야
비로소 정해집니다.

**내장 함수(Intrinsic Function)** 는 이렇게 스택을 실행할 때 정해지는 값을 템플릿 안에서
가져오거나 가공하는 함수입니다. CloudFormation이 템플릿을 처리하는 시점에 함수를 계산해 실제 값으로
바꿉니다.

- 공식 문서:
  [Intrinsic function reference](https://docs.aws.amazon.com/AWSCloudFormation/latest/TemplateReference/intrinsic-function-reference.html)

## 전체 이름과 YAML 짧은 형식

YAML에서는 내장 함수를 두 가지 방식으로 쓸 수 있습니다. 의미는 같고 표기만 다릅니다.

```yaml
# 전체 이름(full form)
InstanceId:
  Ref: MyEC2Instance

# 짧은 형식(short form), YAML 태그 사용
InstanceId: !Ref MyEC2Instance
```

| 전체 이름        | 짧은 형식     |
| ---------------- | ------------- |
| `Ref`            | `!Ref`        |
| `Fn::GetAtt`     | `!GetAtt`     |
| `Fn::Sub`        | `!Sub`        |
| `Fn::Join`       | `!Join`       |
| `Fn::Base64`     | `!Base64`     |
| `Fn::FindInMap`  | `!FindInMap`  |
| `Fn::ImportValue`| `!ImportValue`|

JSON 템플릿에는 짧은 형식이 없으므로 항상 `{ "Ref": "MyEC2Instance" }`처럼 전체 이름을
씁니다.

### 짧은 형식을 바로 겹쳐 쓸 수 없는 경우

`!Base64` 같은 짧은 형식 바로 뒤에 또 다른 짧은 형식을 붙이면 유효하지 않은 문법이 됩니다. 두 함수 중
적어도 하나는 전체 이름으로 써야 합니다.

```yaml
# 잘못된 예
UserData: !Base64 !Sub "echo ${AWS::Region}"

# 올바른 예: 바깥 함수를 전체 이름으로 작성
UserData:
  Fn::Base64: !Sub "echo ${AWS::Region}"
```

- 근거:
  [`Fn::Base64` 공식 문서의 Note](https://docs.aws.amazon.com/AWSCloudFormation/latest/TemplateReference/intrinsic-function-reference-base64.html)
  (확인일: 2026-10-13)

## 함수 한눈에 보기

시험 대비 관점에서 반드시 알아야 할 함수와 알아 두면 좋은 함수를 나눴습니다.

| 구분      | 함수                                               | 하는 일                                                |
| --------- | -------------------------------------------------- | ------------------------------------------------------ |
| 필수      | `Ref`                                              | 파라미터 값 또는 리소스의 식별자 반환                  |
| 필수      | `Fn::GetAtt`                                       | 리소스의 특정 속성 값 반환                             |
| 필수      | `Fn::FindInMap`                                    | Mappings에서 키에 해당하는 값 반환                     |
| 필수      | `Fn::ImportValue`                                  | 다른 스택이 Export한 값 가져오기                       |
| 필수      | `Fn::Join`                                         | 구분자로 여러 값을 이어 붙이기                         |
| 필수      | `Fn::Sub`                                          | 문자열 안의 `${변수}`를 값으로 치환                    |
| 필수      | `Fn::Base64`                                       | 문자열을 Base64로 인코딩 (주로 EC2 `UserData`)         |
| 필수      | `Fn::If`, `Fn::Equals`, `Fn::Not`, `Fn::And`, `Fn::Or` | 조건 판단                                          |
| 참고      | `Fn::GetAZs`                                       | 리전의 가용 영역 목록 반환                             |
| 참고      | `Fn::Select`                                       | 목록에서 인덱스로 값 하나 선택                         |
| 참고      | `Fn::Split`                                        | 문자열을 구분자로 나눠 목록으로 반환                   |
| 참고      | `Fn::Cidr`                                         | CIDR 블록을 여러 서브넷 CIDR로 분할                    |
| 참고      | `Fn::Transform`                                    | 템플릿 일부에 매크로 적용                              |
| 참고      | `Fn::ForEach`, `Fn::ToJsonString`, `Fn::Length`    | 반복, JSON 문자열 변환, 목록 길이 계산                 |

`Fn::ForEach`, `Fn::ToJsonString`, `Fn::Length`는 템플릿 최상단에
`Transform: AWS::LanguageExtensions`를 선언해야 쓸 수 있습니다.

- 근거:
  [`Fn::ForEach`](https://docs.aws.amazon.com/AWSCloudFormation/latest/TemplateReference/intrinsic-function-reference-foreach.html),
  [`Fn::ToJsonString`](https://docs.aws.amazon.com/AWSCloudFormation/latest/TemplateReference/intrinsic-function-reference-ToJsonString.html),
  [`Fn::Length`](https://docs.aws.amazon.com/AWSCloudFormation/latest/TemplateReference/intrinsic-function-reference-length.html)
  (확인일: 2026-10-13)

## Ref

슬라이드에는 `Fn::Ref`로 적혀 있지만, 실제 함수 이름은 `Fn::` 접두사가 없는 **`Ref`** 입니다.
YAML 짧은 형식은 `!Ref`입니다.

`Ref`가 돌려주는 값은 참조 대상에 따라 다릅니다.

| 참조 대상 | 반환 값                                                             |
| --------- | ------------------------------------------------------------------- |
| 파라미터  | 파라미터에 입력된 값                                                |
| 리소스    | 그 리소스를 식별하는 값. 보통 물리적 ID나 이름이며 리소스 타입마다 다름 |

예를 들어 `AWS::EC2::Instance`는 인스턴스 ID, `AWS::EC2::EIP`는 IP 주소,
`AWS::S3::Bucket`은 버킷 이름을 반환합니다. 리소스별 반환 값은 각 리소스 문서의 **Return
values** 절에서 확인합니다.

```yaml
Resources:
  MyEC2Instance:
    Type: AWS::EC2::Instance
    Properties:
      ImageId: !Ref LatestAmiId        # 파라미터 값
      InstanceType: t3.micro

  MyEIP:
    Type: AWS::EC2::EIP
    Properties:
      InstanceId: !Ref MyEC2Instance   # 인스턴스 ID (예: i-0abc...)
```

- 근거:
  [`Ref` 공식 문서](https://docs.aws.amazon.com/AWSCloudFormation/latest/TemplateReference/intrinsic-function-reference-ref.html)

## Fn::GetAtt

`Ref`가 리소스의 대표 식별자 하나만 돌려준다면, `Fn::GetAtt`는 리소스가 가진 **여러 속성 중 원하는
것**을 꺼냅니다. 예를 들어 EC2 인스턴스의 퍼블릭 DNS 이름, 가용 영역, 프라이빗 IP 같은 값입니다.

```yaml
Outputs:
  WebServerDns:
    Description: 웹 서버 퍼블릭 DNS 이름
    Value: !GetAtt MyEC2Instance.PublicDnsName

  WebServerAz:
    Value: !GetAtt MyEC2Instance.AvailabilityZone
```

전체 이름으로 쓰면 `Fn::GetAtt: [MyEC2Instance, PublicDnsName]`처럼 리소스 이름과
속성 이름을 목록으로 넘깁니다. 어떤 속성을 쓸 수 있는지는 리소스 문서의 **Return values >
Fn::GetAtt** 항목에 정리되어 있습니다.

- 예시:
  [`AWS::EC2::Instance` Return values](https://docs.aws.amazon.com/AWSCloudFormation/latest/TemplateReference/aws-resource-ec2-instance.html#aws-resource-ec2-instance-return-values)

## Fn::Join과 Fn::Sub

두 함수 모두 문자열을 만들 때 씁니다. 같은 결과를 내는 예시로 차이를 비교해 보겠습니다.

```yaml
# Fn::Join: 구분자와 값 목록을 넘겨 이어 붙임
BucketName: !Join
  - "-"
  - - !Ref AWS::StackName
    - logs
    - !Ref AWS::Region

# Fn::Sub: 문자열 안에 ${...}로 변수를 넣어 치환
BucketName: !Sub "${AWS::StackName}-logs-${AWS::Region}"
```

스택 이름이 `my-app`, 리전이 `ap-northeast-2`라면 두 식 모두
`my-app-logs-ap-northeast-2`를 만듭니다.

- `Fn::Join`은 목록을 구분자로 연결할 때 편합니다.
- `Fn::Sub`은 문장 안에 변수를 끼워 넣을 때 읽기 쉽습니다. `${파라미터}`, `${리소스}`,
  `${리소스.속성}`, `${AWS::Region}` 같은 Pseudo Parameter를 모두 쓸 수 있습니다.
- 공식 문서:
  [`Fn::Join`](https://docs.aws.amazon.com/AWSCloudFormation/latest/TemplateReference/intrinsic-function-reference-join.html),
  [`Fn::Sub`](https://docs.aws.amazon.com/AWSCloudFormation/latest/TemplateReference/intrinsic-function-reference-sub.html)

## Fn::Base64

EC2 인스턴스의 `UserData`는 Base64로 인코딩된 문자열을 받습니다. 그래서 부팅 스크립트를 넣을 때
`Fn::Base64`를 함께 씁니다. 스크립트 안에서 스택 정보를 쓰려면 `Fn::Sub`과 조합합니다.

```yaml
MyEC2Instance:
  Type: AWS::EC2::Instance
  Properties:
    ImageId: !Ref LatestAmiId
    InstanceType: t3.micro
    UserData:
      Fn::Base64: !Sub |
        #!/bin/bash
        dnf install -y httpd
        echo "Hello from ${AWS::StackName}" > /var/www/html/index.html
        systemctl enable --now httpd
```

앞에서 설명한 대로 `!Base64 !Sub`처럼 짧은 형식을 바로 겹치지 않고, 바깥쪽을 `Fn::Base64:`
전체 이름으로 씁니다.

## 다른 챕터에서 다루는 함수

- `Fn::FindInMap`: Mappings에서 값을 꺼냅니다.
  [Mappings](CLOUD_FORMATION_007_mappings.md)를 참고합니다.
- `Fn::ImportValue`: 다른 스택이 Export한 값을 가져옵니다.
  [Outputs와 Cross-Stack Reference](CLOUD_FORMATION_008_outputs_and_cross_stack.md)를
  참고합니다.
- `Fn::If`, `Fn::Equals`, `Fn::Not`, `Fn::And`, `Fn::Or`: 조건을 정의하고
  적용합니다. [Conditions](CLOUD_FORMATION_009_conditions.md)를 참고합니다.

## 알아 두면 좋은 함수 예시

```yaml
# 리전의 첫 번째 가용 영역 선택
AvailabilityZone: !Select [0, !GetAZs ""]

# "a,b,c"를 나눠 두 번째 값 "b" 선택
SecondValue: !Select [1, !Split [",", "a,b,c"]]
```

- `!GetAZs ""`는 스택이 배포되는 리전의 가용 영역 목록을 돌려줍니다.
- `Fn::Select`의 인덱스는 0부터 시작합니다.

## 정리

- 내장 함수는 스택 실행 시점에 정해지는 값을 템플릿에서 가져오거나 가공합니다.
- 함수 이름은 `Ref`이며 `Fn::Ref`가 아닙니다. 파라미터는 값을, 리소스는 타입별 식별자를 반환합니다.
- `Fn::GetAtt`는 리소스의 특정 속성(예: `PublicDnsName`)을 가져옵니다.
- `Fn::Sub`은 `${}` 치환, `Fn::Join`은 구분자 연결에 씁니다.
- `UserData`에는 `Fn::Base64`가 필요하며, 짧은 형식을 바로 겹치지 않도록 주의합니다.
- `Fn::ForEach`, `Fn::ToJsonString`, `Fn::Length`는
  `AWS::LanguageExtensions` Transform이 필요합니다.

## 시험 대비 문제

### 문제 1

CloudFormation 템플릿으로 EC2 인스턴스를 만든 뒤, Outputs에 인스턴스의 퍼블릭 DNS 이름을
출력하려고 합니다. 어떤 함수를 사용해야 합니까?

- A. `!Ref MyEC2Instance`
- B. `!GetAtt MyEC2Instance.PublicDnsName`
- C. `!FindInMap [MyEC2Instance, PublicDnsName]`
- D. `!ImportValue MyEC2Instance`

#### 정답

B

#### 해설

`Ref`를 EC2 인스턴스에 쓰면 인스턴스 ID를 반환합니다. 퍼블릭 DNS 이름처럼 리소스의 특정 속성이 필요할
때는 `Fn::GetAtt`를 사용합니다. `Fn::FindInMap`은 Mappings 섹션의 값을,
`Fn::ImportValue`는 다른 스택이 Export한 값을 가져오는 함수입니다.

### 문제 2

개발자가 EC2 인스턴스의 `UserData`에 부팅 스크립트를 넣으면서 스크립트 안에 스택 이름을 포함하려고 합니다.
다음 중 올바른 템플릿 작성 방법은 무엇입니까?

- A. `UserData: !Sub "echo ${AWS::StackName}"`
- B. `UserData: !Base64 !Sub "echo ${AWS::StackName}"`
- C. `UserData:` 아래에 `Fn::Base64: !Sub "echo ${AWS::StackName}"`
- D. `UserData: !Join ["", ["echo ", !Ref AWS::StackName]]`

#### 정답

C

#### 해설

`UserData`는 Base64로 인코딩된 값이 필요하므로 A와 D는 인코딩이 빠져 있습니다. B는 짧은 형식
`!Base64` 바로 뒤에 `!Sub`를 겹쳐 써서 유효하지 않은 YAML 구문입니다. C처럼 바깥 함수를
`Fn::Base64` 전체 이름으로 쓰고 안쪽에 `!Sub`를 쓰면 스택 이름을 치환한 뒤 Base64로
인코딩합니다.
