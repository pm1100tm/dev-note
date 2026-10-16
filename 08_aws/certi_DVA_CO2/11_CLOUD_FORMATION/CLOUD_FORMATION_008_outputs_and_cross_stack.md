# CloudFormation Outputs와 Cross-Stack Reference

## 왜 Outputs가 필요한가요?

인프라가 커지면 하나의 템플릿에 모든 리소스를 넣기보다 네트워크 스택, 애플리케이션 스택처럼 나누어 관리합니다. 그런데
애플리케이션 스택은 네트워크 스택이 만든 VPC ID나 보안 그룹 ID를 알아야 리소스를 만들 수 있습니다.

이 값을 콘솔에서 복사해 파라미터로 넣는 방식은 번거롭고 실수하기 쉽습니다. **Outputs**는 스택이 만든
리소스의 정보를 바깥으로 보여 주는 섹션이고, 여기에 **Export**를 붙이면 다른 스택이 그 값을 가져다 쓸 수
있습니다.

```shell
Network Stack                                Application Stack
  Outputs:                                     Resources:
    VpcId (Export: network-VpcId)   ──참조──▶     !ImportValue network-VpcId
```

- 공식 문서:
  [CloudFormation template Outputs syntax](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/outputs-section-structure.html)

## Outputs 섹션 구조

```yaml
Outputs:
  WebServerPublicDns:                 # 출력의 논리 ID
    Description: 웹 서버 공개 DNS 이름
    Value: !GetAtt WebServer.PublicDnsName
```

| 필드          | 필수 여부 | 설명                                                                 |
| ------------- | --------- | -------------------------------------------------------------------- |
| 논리 ID       | 필수      | 템플릿 안에서 고유한 영문·숫자 이름입니다.                           |
| `Description` | 선택      | 출력값 설명입니다. 함수나 파라미터는 쓸 수 없고 문자열만 가능합니다. |
| `Value`       | 필수      | 출력할 값입니다. `Ref`, `Fn::GetAtt` 같은 내장 함수를 쓸 수 있습니다. |
| `Export`      | 선택      | 다른 스택에서 가져갈 수 있도록 내보낼 이름(`Name`)을 지정합니다.     |
| `Condition`   | 선택      | 조건이 참일 때만 출력합니다.                                         |

Outputs는 Export하지 않아도 스택 결과를 확인하는 용도로 유용합니다. 예를 들어 웹 서버 주소나 버킷 이름을
출력해 두면 리소스를 찾으러 다른 콘솔 화면을 열 필요가 없습니다.

### 출력값 확인 방법

- 콘솔: CloudFormation 스택 상세 화면의 **Outputs** 탭에서 확인합니다.
- CLI: 아래 명령으로 확인합니다. `my-network-stack`은 예시 스택 이름입니다.

```shell
aws cloudformation describe-stacks \
  --stack-name my-network-stack \
  --query "Stacks[0].Outputs" \
  --output table
```

출력값은 스택 작업이 끝난 뒤에만 조회할 수 있고, `*_IN_PROGRESS` 상태에서는 볼 수 없습니다. 또한
Outputs의 값은 가려지지 않으므로 비밀번호나 시크릿을 출력하면 안 됩니다.

## Cross-Stack Reference 예시

### 1. 내보내는 스택: 보안 그룹 Export

```yaml
# network-stack.yaml
Resources:
  SSHSecurityGroup:
    Type: AWS::EC2::SecurityGroup
    Properties:
      GroupDescription: Allow SSH from office
      SecurityGroupIngress:
        - IpProtocol: tcp
          FromPort: 22
          ToPort: 22
          CidrIp: 203.0.113.0/24    # 예시 대역(문서용 IP). 실제 허용할 대역으로 바꿉니다.

Outputs:
  SSHSecurityGroupId:
    Description: SSH 접속용 보안 그룹 ID
    Value: !GetAtt SSHSecurityGroup.GroupId
    Export:
      Name: !Sub '${AWS::StackName}-SSHSecurityGroupId'
```

Export 이름에 `${AWS::StackName}`을 붙이면, 같은 템플릿으로 여러 스택을 만들어도 이름이 겹치지
않습니다. 이 템플릿을 `my-network-stack`이라는 이름으로 배포하면 Export 이름은
`my-network-stack-SSHSecurityGroupId`가 됩니다.

### 2. 가져오는 스택: `Fn::ImportValue`

```yaml
# app-stack.yaml
Parameters:
  LatestAmiId:
    Type: AWS::SSM::Parameter::Value<AWS::EC2::Image::Id>
    Default: /aws/service/ami-amazon-linux-latest/al2023-ami-kernel-default-x86_64

Resources:
  AppServer:
    Type: AWS::EC2::Instance
    Properties:
      InstanceType: t3.micro
      ImageId: !Ref LatestAmiId
      SecurityGroupIds:
        - !ImportValue my-network-stack-SSHSecurityGroupId
```

`!ImportValue`에는 **Export 이름**을 넣습니다. 출력의 논리
ID(`SSHSecurityGroupId`)가 아니라는 점에 주의합니다.

- 공식 문서:
  [Get exported outputs from a deployed CloudFormation stack](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/using-cfn-stack-exports.html)

## Cross-Stack Reference의 제약

공식 문서 기준 제약은 다음과 같습니다. (확인일: 2026-10-13)

- Export 이름은 **같은 계정·같은 리전 안에서 고유**해야 합니다.
- `Export`와 `Fn::ImportValue`는 **같은 계정·같은 리전** 안에서만 동작합니다.
- 다른 스택이 값을 Import하고 있으면, **내보낸 스택을 삭제하거나 내보낸 값을 수정할 수 없습니다.** 먼저
  모든 Import를 제거해야 합니다.
- Export의 `Name`과 `Fn::ImportValue` 안에는 리소스에 의존하는 `Ref`나 `GetAtt`를
  쓸 수 없습니다.

어떤 스택이 Export를 사용 중인지는 아래 명령으로 확인합니다.

```shell
# 현재 리전의 Export 목록
aws cloudformation list-exports

# 특정 Export를 Import하는 스택 목록
aws cloudformation list-imports --export-name my-network-stack-SSHSecurityGroupId
```

이 제약은 불편해 보이지만, 공유하는 리소스를 실수로 지워 다른 스택이 깨지는 일을 막는 안전장치입니다.

### 참고: 최근 추가된 `Fn::GetStackOutput`

슬라이드에는 없는 내용이지만, 공식 문서에는 Export 없이 다른 스택의 출력을 참조하는
`Fn::GetStackOutput` 함수가 소개되어 있습니다. 다른 계정·다른 리전도 참조할 수 있지만 **약한
참조**라서, 참조 대상 스택을 삭제해도 막아 주지 않습니다. 시험 범위는 `Export` +
`Fn::ImportValue`를 중심으로 이해하면 됩니다.

- 공식 문서:
  [Fn::GetStackOutput](https://docs.aws.amazon.com/AWSCloudFormation/latest/TemplateReference/intrinsic-function-reference-getstackoutput.html)

### 중첩 스택(Nested Stack)과의 차이

값을 한 묶음의 스택 안에서만 주고받는다면 중첩 스택이, 여러 독립 스택이 공유한다면 Export가 적합합니다.

## 실무 활용

- 네트워크 팀은 VPC, 서브넷, 보안 그룹을 담은 스택을 관리하고 ID를 Export합니다.
- 애플리케이션 팀은 그 값을 Import해 자기 스택만 관리합니다. 각 팀이 자기 영역에 집중할 수 있습니다.
- Export 이름은 `<스택 이름>-<값 이름>`처럼 규칙을 정해 두면 찾기 쉽고 충돌을 막을 수 있습니다.

## 정리

- Outputs는 스택이 만든 값을 콘솔·CLI에서 확인하거나 다른 스택에 넘기기 위한 섹션입니다.
- 다른 스택에서 쓰려면 `Export`의 `Name`으로 내보내고
  `Fn::ImportValue`(`!ImportValue`)로 가져옵니다.
- Export 이름은 계정·리전 안에서 고유해야 합니다.
- Import 중인 Export가 있으면 내보낸 스택을 삭제하거나 그 값을 수정할 수 없습니다.

## 시험 대비 문제

### 문제 1

네트워크 스택이 VPC ID를 `Outputs`로 내보내고 있고, 애플리케이션 스택이 이 값을 사용해야 합니다.
애플리케이션 템플릿에서 사용할 함수는 무엇입니까?

- A. `Fn::GetAtt`
- B. `Fn::FindInMap`
- C. `Fn::ImportValue`
- D. `Ref`

#### 정답

C

#### 해설

다른 스택이 `Export`로 내보낸 값은 `Fn::ImportValue`에 Export 이름을 넣어 가져옵니다.
`Fn::GetAtt`와 `Ref`는 같은 템플릿 안의 리소스를 참조할 때, `Fn::FindInMap`은
Mappings 값을 꺼낼 때 사용합니다.

### 문제 2

운영자가 더 이상 쓰지 않는다고 생각한 네트워크 스택을 삭제하려 했지만 삭제가 실패했습니다. 이 스택은 보안 그룹
ID를 Export하고 있습니다. 가장 가능성이 높은 원인은 무엇입니까?

- A. 스택에 `DeletionPolicy: Retain`이 설정된 리소스가 있다.
- B. 다른 스택이 `Fn::ImportValue`로 이 스택의 Export 값을 참조하고 있다.
- C. Export 이름에 `AWS::StackName`이 포함되어 있다.
- D. Outputs 섹션에 `Description`이 없다.

#### 정답

B

#### 해설

다른 스택이 Export 값을 Import하고 있으면 내보낸 스택을 삭제할 수 없습니다.
`aws cloudformation list-imports`로 참조하는 스택을 찾아 Import를 먼저 제거한 뒤
삭제해야 합니다. A의 `Retain`은 리소스를 남겨 둘 뿐 스택 삭제 자체를 막지는 않습니다.
