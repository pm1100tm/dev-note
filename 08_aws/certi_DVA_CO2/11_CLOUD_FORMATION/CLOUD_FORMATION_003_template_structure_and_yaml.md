# CloudFormation 템플릿 구성 요소와 YAML 기초

## 왜 템플릿 구조부터 알아야 하나요?

CloudFormation 템플릿은 정해진 이름의 섹션(section)으로 나뉜 문서입니다. 각 섹션이 어떤 역할을
하는지 알면 처음 보는 템플릿도 "무엇을 입력받아, 무엇을 만들고, 무엇을 돌려주는지" 빠르게 읽을 수 있습니다. 또한
템플릿은 대부분 YAML로 작성하므로, YAML의 기본 문법을 먼저 익혀 두면 이후 챕터의 예시를 이해하기 쉽습니다.

## 템플릿의 구성 요소

템플릿은 여러 섹션으로 이뤄지며, 이 중 **`Resources`만 필수**입니다. 나머지는 필요할 때만 작성합니다.

| 섹션                       | 필수 여부 | 역할                                                                 | 자세히 보기                                                   |
| -------------------------- | --------- | -------------------------------------------------------------------- | ------------------------------------------------------------- |
| `AWSTemplateFormatVersion` | 선택      | 템플릿 형식 버전입니다. 현재 사용할 수 있는 값은 `"2010-09-09"` 하나입니다. | -                                                             |
| `Description`              | 선택      | 템플릿에 대한 설명(주석)입니다.                                      | -                                                             |
| `Resources`                | **필수**  | 만들 AWS 리소스와 설정을 선언합니다.                                 | [Resources](CLOUD_FORMATION_005_resources.md)                 |
| `Parameters`               | 선택      | 스택을 만들 때 입력받는 값(동적 입력)입니다.                         | [Parameters](CLOUD_FORMATION_006_parameters.md)               |
| `Mappings`                 | 선택      | 템플릿 안에 고정해 둔 값의 조회표(정적 변수)입니다.                  | [Mappings](CLOUD_FORMATION_007_mappings.md)                   |
| `Outputs`                  | 선택      | 스택이 만든 값 중 밖으로 보여 주거나 다른 스택에 넘길 값입니다.      | [Outputs](CLOUD_FORMATION_008_outputs_and_cross_stack.md)     |
| `Conditions`               | 선택      | 조건에 따라 리소스나 출력의 생성 여부를 정합니다.                    | [Conditions](CLOUD_FORMATION_009_conditions.md)               |

슬라이드에서는 위 일곱 가지를 다루지만, 공식 문서에는 `Metadata`, `Rules`, `Transform`
섹션도 있습니다. `Transform`은 AWS SAM이나 매크로를 적용할 때 사용하며, 시험에서는 "SAM 템플릿에는
`Transform`이 들어간다" 정도로 기억하면 충분합니다.

- 공식 문서:
  [CloudFormation template sections](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/template-anatomy.html)
  (확인일: 2026-10-13)

### 템플릿 도우미: 참조와 함수

섹션 외에 템플릿 작성을 돕는 두 가지 도구가 있습니다.

- **참조(References)**: 템플릿 안의 다른 요소를 가리킵니다. 예를 들어
  `!Ref WebServerSecurityGroup`은 같은 템플릿에 선언한 보안 그룹을 가리킵니다.
- **함수(Functions)**: 값을 계산하거나 가공합니다. 예를 들어 `!GetAtt`는 리소스의 속성 값을,
  `!Sub`는 문자열 치환 결과를 돌려줍니다.

함수는
[Intrinsic Functions](CLOUD_FORMATION_010_intrinsic_functions.md)에서
자세히 다룹니다.

### 전체 모양 한눈에 보기

```yaml
AWSTemplateFormatVersion: "2010-09-09"
Description: 섹션 구조를 보여 주기 위한 예시 템플릿

Parameters:      # 입력값
  EnvName:
    Type: String
    Default: dev

Mappings: {}     # 고정 조회표 (이 예시에서는 비워 둠)

Conditions:      # 조건
  IsProd: !Equals [!Ref EnvName, prod]

Resources:       # 필수: 만들 리소스
  LogBucket:
    Type: AWS::S3::Bucket

Outputs:         # 결과값
  LogBucketName:
    Value: !Ref LogBucket
```

위 템플릿은 각 섹션의 위치를 보여 주기 위한 예시입니다. 실제로 배포하면 S3 버킷 하나가 만들어집니다.

## YAML 기초

CloudFormation 템플릿은 JSON 또는 YAML로 작성할 수 있습니다. JSON은 중괄호와 따옴표, 쉼표가
많아 사람이 읽고 고치기 불편하고 주석을 쓸 수 없습니다. 그래서 실무와 학습 모두 YAML을 주로 사용합니다.

### 키-값 쌍

`키: 값` 형태로 작성합니다. 콜론(`:`) 뒤에는 반드시 공백 한 칸이 필요합니다.

```yaml
Name: my-web-server
Port: 80
Enabled: true
```

### 중첩 객체

들여쓰기로 계층을 표현합니다. 들여쓰기에는 탭이 아닌 **공백**을 사용하며, 같은 계층은 같은 칸만큼 들여씁니다.

```yaml
WebServer:
  Type: AWS::EC2::Instance
  Properties:
    InstanceType: t3.micro
```

### 배열(목록)

`-`로 시작하는 줄이 목록의 항목입니다. 항목 안에 다시 키-값을 둘 수 있습니다.

```yaml
SecurityGroupIngress:
  - IpProtocol: tcp
    FromPort: 80
    ToPort: 80
    CidrIp: 0.0.0.0/0
  - IpProtocol: tcp
    FromPort: 443
    ToPort: 443
    CidrIp: 0.0.0.0/0
```

짧은 목록은 `[a, b]`처럼 한 줄로도 쓸 수 있습니다.

### 여러 줄 문자열

EC2 `UserData` 스크립트처럼 긴 문자열은 `|` 또는 `>`로 작성합니다.

- `|`: 줄바꿈을 그대로 유지합니다. 셸 스크립트나 설정 파일 내용에 사용합니다.
- `>`: 줄바꿈을 공백으로 바꿔 한 줄로 이어 붙입니다. 긴 설명 문장에 사용합니다.

```yaml
Script: |
  #!/bin/bash
  echo "line 1"
  echo "line 2"

Summary: >
  이 문장은 여러 줄에 걸쳐 썼지만
  실제 값은 한 줄로 이어집니다.
```

셸 스크립트를 `>`로 쓰면 명령이 한 줄로 합쳐져 실행되지 않으므로, 스크립트에는 `|`를 사용합니다.

### 주석

`#` 뒤의 내용은 주석입니다. JSON과 달리 템플릿 안에 설명을 남길 수 있습니다.

```yaml
# 웹 서버용 보안 그룹
WebServerSecurityGroup:
  Type: AWS::EC2::SecurityGroup   # 타입 뒤에도 주석을 둘 수 있습니다.
```

## 같은 템플릿을 JSON과 YAML로 비교하기

S3 버킷 하나를 만드는 같은 템플릿입니다.

```json
{
  "AWSTemplateFormatVersion": "2010-09-09",
  "Description": "S3 bucket example",
  "Resources": {
    "MyBucket": {
      "Type": "AWS::S3::Bucket",
      "Properties": {
        "Tags": [
          { "Key": "env", "Value": "dev" }
        ]
      }
    }
  }
}
```

```yaml
AWSTemplateFormatVersion: "2010-09-09"
Description: S3 bucket example
Resources:
  MyBucket:
    Type: AWS::S3::Bucket
    Properties:
      Tags:
        - Key: env
          Value: dev
```

YAML 쪽이 기호가 적어 구조가 더 잘 보입니다. 다만 YAML은 들여쓰기가 곧 문법이므로, 칸이 하나만 어긋나도
다른 의미가 되거나 오류가 납니다. 템플릿을 배포하기 전에
`aws cloudformation validate-template` 명령이나 `cfn-lint` 같은 도구로 문법을
확인하는 습관을 들이는 것이 좋습니다.

- [cfn-lint (AWS CloudFormation Linter)](https://github.com/aws-cloudformation/cfn-lint)

## 정리

- 템플릿에서 필수 섹션은 `Resources` 하나이며, 나머지 섹션은 선택입니다.
- `AWSTemplateFormatVersion`의 값은 `"2010-09-09"`이고, `Description`은
  템플릿 설명입니다.
- `Parameters`는 동적 입력, `Mappings`는 정적 조회표, `Outputs`는 결과값,
  `Conditions`는 생성 조건입니다.
- YAML은 들여쓰기로 계층을 표현하고, `-`로 목록을, `|`와 `>`로 여러 줄 문자열을, `#`으로 주석을
  씁니다.

## 시험 대비 문제

### 문제 1

CloudFormation 템플릿에서 반드시 포함해야 하는 섹션은 무엇입니까?

- A. `Parameters`
- B. `Outputs`
- C. `Resources`
- D. `AWSTemplateFormatVersion`

#### 정답

C

#### 해설

`Resources`는 스택이 만들 AWS 리소스를 선언하는 섹션이며, 모든 템플릿에 반드시 있어야 합니다.
`AWSTemplateFormatVersion`은 자주 쓰이지만 선택 섹션입니다. `Parameters`와
`Outputs`도 필요할 때만 작성합니다.

### 문제 2

개발자가 리전별로 다른 AMI ID를 템플릿 안에 미리 적어 두고, 스택이 배포되는 리전에 따라 알맞은 값을 고르려고
합니다. 값은 배포할 때 사용자가 입력하지 않습니다. 어떤 섹션을 사용해야 합니까?

- A. `Parameters`
- B. `Mappings`
- C. `Outputs`
- D. `Description`

#### 정답

B

#### 해설

`Mappings`는 템플릿 안에 고정해 둔 값의 조회표로, 리전이나 환경처럼 미리 알 수 있는 키에 따라 값을 고를
때 사용합니다. `Parameters`는 스택을 만들 때 사용자가 값을 입력하는 섹션이므로 문제의 조건과 맞지
않습니다. `Outputs`는 스택이 만든 결과값을 내보내는 섹션입니다.
