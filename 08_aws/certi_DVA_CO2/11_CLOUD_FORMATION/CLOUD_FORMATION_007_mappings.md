# CloudFormation Mappings

## 왜 Mappings가 필요한가요?

같은 템플릿을 서울 리전과 버지니아 리전에 배포한다고 생각해 봅시다. EC2를 만들 때 필요한 AMI ID는 리전마다
다르기 때문에, 템플릿에 하나의 값을 적어 두면 다른 리전에서는 배포가 실패합니다.

이 값을 매번 파라미터로 입력받을 수도 있지만, 사용자가 리전에 맞는 AMI ID를 찾아 정확히 입력해야 하므로
실수하기 쉽습니다. **Mappings**는 이런 경우에 "리전이 A이면 값 X, B이면 값 Y"처럼 미리 정해 둔 값
표를 템플릿 안에 넣어 두는 기능입니다.

- Mappings의 값은 모두 템플릿에 **하드코딩된 고정값**입니다.
- 환경(dev, prod), 리전, AMI 종류처럼 미리 알 수 있는 기준에 따라 값을 고를 때 유용합니다.
- 공식 문서:
  [CloudFormation template Mappings syntax](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/mappings-section-structure.html)

## Mappings 구조

Mappings는 **맵 이름 → 1단계 키 → 2단계 키 → 값**의 3단 구조입니다.

```yaml
Mappings:
  RegionMap:                 # 맵 이름
    us-east-1:               # 1단계 키(Top Level Key)
      HVM64: ami-0aaaaaaaaaaaaaaaa   # 2단계 키(Second Level Key): 값
      ARM64: ami-0bbbbbbbbbbbbbbbb
    ap-northeast-2:
      HVM64: ami-0cccccccccccccccc
      ARM64: ami-0dddddddddddddddd
```

위 AMI ID는 구조를 보여 주기 위한 예시 값이므로 실제로 배포할 수 없습니다.

공식 문서 기준으로 Mappings에는 다음 제약이 있습니다.

- 키는 리터럴 문자열이어야 하고, 값은 `String` 또는 `List`입니다.
- Mappings 섹션 안에는 파라미터, Pseudo Parameter, 내장 함수를 넣을 수 없습니다.

## 값 꺼내기: `Fn::FindInMap`

Mappings의 값은 `Fn::FindInMap` 함수로 꺼냅니다. YAML 짧은 형식은
`!FindInMap`입니다.

```yaml
!FindInMap [ MapName, TopLevelKey, SecondLevelKey ]
```

1단계 키에 `AWS::Region` Pseudo Parameter를 넣으면, 스택이 배포되는 리전에 맞는 값을
자동으로 고를 수 있습니다.

```yaml
Resources:
  WebServer:
    Type: AWS::EC2::Instance
    Properties:
      InstanceType: t3.micro
      ImageId: !FindInMap [RegionMap, !Ref 'AWS::Region', HVM64]
```

서울 리전(`ap-northeast-2`)에 배포하면 `RegionMap → ap-northeast-2 → HVM64`
값이 선택됩니다. AMI는 리전마다 ID가 다르기 때문에 Mappings가 잘 맞는 대표적인 예입니다.

### 예시: 환경별 설정

파라미터로 환경 이름만 입력받고, 세부 값은 Mappings에서 고르게 할 수도 있습니다.

```yaml
Parameters:
  EnvType:
    Type: String
    AllowedValues: [dev, prod]
    Default: dev

Mappings:
  EnvConfig:
    dev:
      InstanceType: t3.micro
      VolumeSize: '20'
    prod:
      InstanceType: t3.large
      VolumeSize: '100'

Resources:
  AppServer:
    Type: AWS::EC2::Instance
    Properties:
      InstanceType: !FindInMap [EnvConfig, !Ref EnvType, InstanceType]
      ImageId: !Ref LatestAmiId   # LatestAmiId는 006 문서의 SSM 파라미터 예시를 사용한다고 가정합니다.
```

사용자는 `dev` 또는 `prod`만 고르면 되고, 인스턴스 타입이나 볼륨 크기 같은 세부 값은 템플릿 작성자가
관리합니다. 위 조각만으로는 배포할 수 없으며, `LatestAmiId` 파라미터는
[Parameters](CLOUD_FORMATION_006_parameters.md)의 예시를 함께 넣어야 합니다.

## Mappings와 Parameters 선택 기준

| 구분             | Mappings                                              | Parameters                                     |
| ---------------- | ----------------------------------------------------- | ---------------------------------------------- |
| 값을 정하는 시점 | 템플릿 작성 시 미리 모두 적어 둠                      | 스택 생성·업데이트 시 입력                     |
| 값을 정하는 사람 | 템플릿 작성자                                         | 스택을 배포하는 사용자                         |
| 잘 맞는 경우     | 리전, AZ, 계정, 환경처럼 다른 값에서 유추할 수 있는 값 | 키 페어 이름, 사용자별 설정처럼 미리 알 수 없는 값 |
| 장점             | 사용자가 잘못된 값을 넣을 여지가 적어 더 안전함        | 템플릿 수정 없이 유연하게 값을 바꿀 수 있음    |
| 값 변경 방법     | 템플릿 수정 후 스택 업데이트                          | 새 값으로 스택 업데이트                        |

정리하면, **가능한 값을 미리 모두 알고 있고 리전이나 환경 같은 변수로 결정할 수 있다면 Mappings**,
**사용자마다 정말 다른 값이라면 Parameters**를 사용합니다.

## 실무에서의 주의점: AMI ID 하드코딩

Mappings에 AMI ID를 적어 두면 시간이 지나 새 AMI가 나올 때마다 템플릿을 직접 고쳐야 합니다. 공식
문서도 AMI ID는 가능하면 Systems Manager Parameter Store 값을 쓰도록 권장합니다.

```yaml
Parameters:
  LatestAmiId:
    Type: AWS::SSM::Parameter::Value<AWS::EC2::Image::Id>
    Default: /aws/service/ami-amazon-linux-latest/al2023-ami-kernel-default-x86_64
```

이 방식은 스택 생성·업데이트 시점의 최신 AMI ID를 리전에 맞게 가져옵니다. 자세한 내용은
[Parameters](CLOUD_FORMATION_006_parameters.md)를 참고합니다. 다만 시험에서는
"리전마다 다른 AMI ID를 고르는 방법"으로 Mappings와 `Fn::FindInMap`이 여전히 자주 나오므로
두 방식을 모두 알아 둡니다.

## 정리

- Mappings는 템플릿 안에 넣어 두는 고정값 표입니다.
- 구조는 맵 이름 → 1단계 키 → 2단계 키 → 값이며,
  `!FindInMap [MapName, TopLevelKey, SecondLevelKey]`로 꺼냅니다.
- `!Ref 'AWS::Region'`과 함께 쓰면 리전별 값을 자동으로 고를 수 있습니다.
- 미리 알 수 있는 값은 Mappings, 사용자마다 다른 값은 Parameters가 적합합니다.
- 실무에서 최신 AMI ID가 필요하면 SSM 파라미터 타입을 우선 검토합니다.

## 시험 대비 문제

### 문제 1

하나의 CloudFormation 템플릿으로 여러 리전에 EC2 인스턴스를 배포하려고 합니다. AMI ID는 리전마다
다르며, 배포하는 사용자가 AMI ID를 직접 입력하지 않게 하고 싶습니다. 가장 적절한 방법은 무엇입니까?

- A. 리전별 AMI ID를 `Mappings`에 정의하고 `!FindInMap`과 `AWS::Region`으로 값을
  선택한다.
- B. `Parameters`에 `AmiId` 파라미터를 만들고 사용자가 리전에 맞는 값을 입력한다.
- C. `Outputs`에 AMI ID를 내보내고 다른 스택에서 가져온다.
- D. `Conditions`에 리전마다 조건을 만들고 리소스를 리전 수만큼 복사한다.

#### 정답

A

#### 해설

리전별 AMI ID처럼 미리 알 수 있고 리전에 따라 결정되는 값은 Mappings에 적어 두고,
`!FindInMap [RegionMap, !Ref 'AWS::Region', HVM64]`처럼 꺼내 쓰는 방식이
정석입니다. B는 사용자가 값을 잘못 넣을 수 있고, D는 템플릿이 불필요하게 복잡해집니다.

### 문제 2

`!FindInMap [EnvConfig, !Ref EnvType, InstanceType]` 에서
`!Ref EnvType`이 하는 역할은 무엇입니까?

- A. 맵 이름을 지정한다.
- B. 1단계 키(Top Level Key)를 지정한다.
- C. 2단계 키(Second Level Key)를 지정한다.
- D. 키를 찾지 못했을 때 사용할 기본값을 지정한다.

#### 정답

B

#### 해설

`Fn::FindInMap`의 인자 순서는 맵 이름, 1단계 키, 2단계 키입니다. 이 예시에서는
`EnvConfig`가 맵 이름, `EnvType` 파라미터 값(`dev` 또는 `prod`)이 1단계 키,
`InstanceType`이 2단계 키입니다.
