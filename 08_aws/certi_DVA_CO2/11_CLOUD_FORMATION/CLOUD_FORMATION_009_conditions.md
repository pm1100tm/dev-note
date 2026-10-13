# CloudFormation Conditions

## 왜 Conditions가 필요한가요?

개발 환경과 운영 환경은 대부분 같은 구성을 쓰지만, 운영 환경에만 추가 EBS 볼륨이나 더 큰 인스턴스가 필요한
경우가 많습니다. 환경마다 템플릿을 따로 만들면 공통 부분을 고칠 때마다 여러 파일을 함께 수정해야 합니다.

**Conditions**는 "이 조건이 참일 때만 이 리소스(또는 출력)를 만든다"를 템플릿 안에 선언하는
섹션입니다. 하나의 템플릿으로 환경에 따라 다른 결과를 만들 수 있습니다.

```shell
같은 템플릿
  ├─ EnvType=dev  → EC2 인스턴스만 생성
  └─ EnvType=prod → EC2 인스턴스 + 추가 EBS 볼륨 생성
```

- 공식 문서:
  [CloudFormation template Conditions syntax](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/conditions-section-structure.html)

## 조건에 자주 쓰는 기준

- 환경 구분: `dev`, `test`, `prod`
- AWS 리전: `AWS::Region` Pseudo Parameter
- 임의의 파라미터 값: 예를 들어 "버킷 이름이 비어 있지 않으면"

조건은 파라미터, Pseudo Parameter, Mappings 값, 다른 조건을 참조할 수 있습니다. 다만
**리소스의 논리 ID나 속성은 조건에서 참조할 수 없습니다.** 조건은 리소스를 만들기 전에 평가되기 때문입니다.

## 조건 정의하기

조건의 논리 ID는 원하는 이름으로 정하고, 값은 조건 함수로 작성합니다.

| 함수         | 의미                                              | 예시                                              |
| ------------ | ------------------------------------------------- | ------------------------------------------------- |
| `Fn::Equals` | 두 값이 같으면 참                                 | `!Equals [!Ref EnvType, prod]`                    |
| `Fn::Not`    | 조건의 결과를 반대로 뒤집음                       | `!Not [!Equals [!Ref BucketName, '']]`            |
| `Fn::And`    | 모든 조건이 참이면 참                             | `!And [!Condition IsProd, !Condition HasBucket]`  |
| `Fn::Or`     | 하나라도 참이면 참                                | `!Or [!Equals [!Ref EnvType, prod], !Equals [!Ref EnvType, stage]]` |
| `Fn::If`     | 조건이 참이면 첫 번째 값, 거짓이면 두 번째 값을 반환 | `!If [IsProd, t3.large, t3.micro]`            |

`Fn::If`는 `Conditions` 섹션에서 조건을 정의할 때가 아니라, 주로 `Resources`와
`Outputs`의 **속성 값**을 고를 때 사용합니다.

```yaml
Parameters:
  EnvType:
    Type: String
    AllowedValues: [dev, prod]
    Default: dev

Conditions:
  IsProd: !Equals [!Ref EnvType, prod]
```

## 조건 적용하기

### 리소스와 출력에 `Condition` 키 붙이기

리소스나 출력에 `Condition` 키를 붙이면, 조건이 참일 때만 생성됩니다.

```yaml
Parameters:
  EnvType:
    Type: String
    AllowedValues: [dev, prod]
    Default: dev
  LatestAmiId:
    Type: AWS::SSM::Parameter::Value<AWS::EC2::Image::Id>
    Default: /aws/service/ami-amazon-linux-latest/al2023-ami-kernel-default-x86_64

Conditions:
  IsProd: !Equals [!Ref EnvType, prod]

Resources:
  AppServer:
    Type: AWS::EC2::Instance
    Properties:
      ImageId: !Ref LatestAmiId
      # 운영이면 t3.large, 아니면 t3.micro
      InstanceType: !If [IsProd, t3.large, t3.micro]

  DataVolume:
    Type: AWS::EC2::Volume
    Condition: IsProd                 # 운영 환경에서만 생성
    Properties:
      Size: 100
      AvailabilityZone: !GetAtt AppServer.AvailabilityZone

  DataVolumeAttachment:
    Type: AWS::EC2::VolumeAttachment
    Condition: IsProd                 # 볼륨이 있을 때만 연결
    Properties:
      InstanceId: !Ref AppServer
      VolumeId: !Ref DataVolume
      Device: /dev/sdh

Outputs:
  DataVolumeId:
    Condition: IsProd                 # 볼륨이 없으면 출력도 만들지 않음
    Value: !Ref DataVolume
```

조건부 리소스(`DataVolume`)를 참조하는 다른 리소스와 출력에도 같은 조건을 붙여야 합니다. 조건이 거짓이면
`DataVolume`이 만들어지지 않으므로, 이를 참조하는 쪽이 조건 없이 남아 있으면 참조 대상을 찾지 못해 스택
작업이 실패합니다.

### 속성 자체를 빼기: `AWS::NoValue`

`Fn::If`의 결과로 `AWS::NoValue`를 반환하면, 조건이 거짓일 때 **해당 속성을 아예 지정하지 않은
것**처럼 처리됩니다.

```yaml
Conditions:
  UseSnapshot: !Not [!Equals [!Ref SnapshotId, '']]

Resources:
  MyDB:
    Type: AWS::RDS::DBInstance
    Properties:
      # ...다른 속성 생략...
      DBSnapshotIdentifier: !If
        - UseSnapshot
        - !Ref SnapshotId
        - !Ref AWS::NoValue
```

스냅샷 ID를 입력하면 스냅샷에서 DB를 복원하고, 비워 두면 `DBSnapshotIdentifier` 속성 없이 새
DB를 만듭니다. 이 조각에는 `SnapshotId` 파라미터와 RDS의 나머지 필수 속성이 생략되어 있습니다.

## 조건이 평가되는 시점

- CloudFormation은 스택을 **생성하거나 업데이트할 때** 조건을 평가합니다.
- 업데이트 때 조건이 참에서 거짓으로 바뀌면, 그 조건이 붙은 리소스는 **삭제**됩니다. 운영 데이터가 있는
  리소스라면 조건 변경 전에 영향을 확인해야 합니다.
- 조건만 단독으로 바꾸는 업데이트는 할 수 없고, 리소스를 추가·수정·삭제하는 변경과 함께 반영해야 합니다.

비슷한 이름의 `Rules` 섹션은 리소스 생성 여부가 아니라 파라미터 입력값의 조합을 검증하는 기능이므로
Conditions와 구분합니다.

## 정리

- Conditions는 파라미터나 리전 같은 값에 따라 리소스·출력의 생성 여부를 정합니다.
- 조건은 `Fn::Equals`, `Fn::And`, `Fn::Or`, `Fn::Not`으로 정의하고, 속성 값은
  `Fn::If`로 고릅니다.
- 리소스와 출력에는 `Condition` 키로 조건을 적용합니다.
- `Fn::If`와 `AWS::NoValue`를 함께 쓰면 조건에 따라 속성 자체를 뺄 수 있습니다.
- 조건은 리소스의 논리 ID나 속성을 참조할 수 없습니다.

## 시험 대비 문제

### 문제 1

하나의 템플릿으로 개발·운영 환경을 배포합니다. `EnvType` 파라미터가 `prod`일 때만 추가 EBS 볼륨을
만들고 싶습니다. 가장 적절한 방법은 무엇입니까?

- A. `Conditions`에 `IsProd: !Equals [!Ref EnvType, prod]`를 정의하고,
  EBS 볼륨 리소스에 `Condition: IsProd`를 지정한다.
- B. `Mappings`에 환경별로 볼륨 생성 여부를 `true`/`false`로 적는다.
- C. `Outputs`에 `Condition`을 지정해 볼륨을 만든다.
- D. 운영용 템플릿을 별도로 복사해 볼륨을 추가한다.

#### 정답

A

#### 해설

리소스 생성 여부는 `Conditions` 섹션에서 조건을 정의하고 리소스에 `Condition` 키를 붙여
제어합니다. B의 Mappings는 값을 고르는 기능이므로 리소스 생성 여부를 직접 정할 수 없습니다. D는 동작은
하지만 템플릿을 여러 벌 관리해야 하므로 Conditions의 목적에 맞지 않습니다.

### 문제 2

다음 중 CloudFormation의 조건 함수(Condition Functions)에 **해당하지 않는** 것은
무엇입니까?

- A. `Fn::And`
- B. `Fn::Equals`
- C. `Fn::If`
- D. `Fn::GetAtt`

#### 정답

D

#### 해설

조건 함수는 `Fn::And`, `Fn::Equals`, `Fn::If`, `Fn::Not`, `Fn::Or`입니다.
`Fn::GetAtt`는 리소스의 속성 값을 가져오는 내장 함수이며, 조건 정의에서는 리소스 속성을 참조할 수도
없습니다.
