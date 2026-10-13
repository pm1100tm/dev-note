# 첫 실습: EC2와 보안 그룹 스택

## 실습 목표

보안 그룹 1개와 그 보안 그룹을 사용하는 EC2 인스턴스 1개를 CloudFormation 스택으로 만들고, 변경과
삭제까지 AWS CLI로 진행합니다. 문법을 모두 이해하지 못해도 괜찮습니다. "템플릿 파일 하나로 리소스가 생기고,
스택 삭제로 한 번에 정리된다"는 흐름을 직접 경험하는 것이 목표입니다. 템플릿의 각 줄이 무엇을 뜻하는지는
[Resources](CLOUD_FORMATION_005_resources.md)에서 분석합니다.

```shell
WebServerSecurityGroup (22번 포트: 내 IP만, 80번 포트: 전체 허용)
        ↓ 연결
WebServer (EC2, Apache httpd 실행)
```

> 이 문서의 명령은 작성 시점에 직접 실행해 검증하지 않았습니다. 정상 결과로
> 적은 스택 상태 값은 AWS 공식 문서 기준이며, 실제 출력 형식은 AWS CLI
> 버전에 따라 조금 다를 수 있습니다.

## 전제 조건

- AWS CLI v2가 설치되어 있고 `aws configure` 등으로 자격 증명이 설정되어 있어야 합니다.
- 사용하는 IAM 사용자 또는 역할에 CloudFormation, EC2, SSM 파라미터
  조회(`ssm:GetParameters`) 권한이 있어야 합니다.
- 실습 리전에 **기본 VPC(Default VPC)** 가 있어야 합니다. 템플릿에서 VPC와 서브넷을 지정하지
  않으므로 기본 VPC에 리소스가 만들어집니다.
- 예시 리전은 `ap-northeast-2`(서울)입니다. 다른 리전을 사용하면 명령의 리전 값을 바꿉니다.

설정을 확인하는 명령입니다.

```shell
aws --version
aws sts get-caller-identity
aws ec2 describe-vpcs --filters Name=isDefault,Values=true --region ap-northeast-2
```

- `aws --version`의 결과가 `aws-cli/2.`로 시작하면 v2입니다.
- `get-caller-identity`가 계정 ID와 ARN을 출력하면 자격 증명이 정상입니다.
- `describe-vpcs` 결과의 `Vpcs` 목록이 비어 있으면 기본 VPC가 없는 것이므로, 기본 VPC를
  만들거나 템플릿에 서브넷을 지정해야 합니다.

### 비용 주의

- EC2 인스턴스와 퍼블릭 IPv4 주소는 실행 시간만큼 요금이 발생할 수 있습니다. 프리 티어 적용 여부는 계정
  조건에 따라 다르므로 직접 확인합니다.
- 실습이 끝나면 반드시 7단계의 스택 삭제까지 진행합니다.

## 1단계: 작업 디렉터리와 템플릿 파일 만들기

로컬 터미널에서 실습용 디렉터리를 만들고 이동합니다.

```shell
mkdir -p ~/cfn-lab && cd ~/cfn-lab
```

같은 디렉터리에 `first-stack.yaml` 파일을 만들고 아래 내용을 저장합니다.

```yaml
AWSTemplateFormatVersion: "2010-09-09"
Description: First CloudFormation stack - EC2 web server with a security group

Parameters:
  SshCidr:
    Type: String
    Description: SSH 접속을 허용할 내 공인 IP (예시 형식 203.0.113.10/32)
    AllowedPattern: '^(\d{1,3}\.){3}\d{1,3}/\d{1,2}$'
    ConstraintDescription: CIDR 형식(예 203.0.113.10/32)으로 입력해야 합니다.
  LatestAmiId:
    Type: AWS::SSM::Parameter::Value<AWS::EC2::Image::Id>
    Default: /aws/service/ami-amazon-linux-latest/al2023-ami-kernel-default-x86_64

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

Outputs:
  PublicDnsName:
    Description: 웹 서버 퍼블릭 DNS 이름
    Value: !GetAtt WebServer.PublicDnsName
```

각 부분의 역할은 다음과 같습니다.

- `SshCidr`: SSH(22번 포트)를 허용할 IP 범위입니다. 기본값을 두지 않아 배포할 때 반드시 입력하게
  했습니다.
- `LatestAmiId`: AWS가 공개한 SSM 파라미터에서 최신 Amazon Linux 2023 AMI ID를
  읽어 옵니다. AMI ID는 리전마다 다르기 때문에 직접 적지 않습니다.
- `WebServerSecurityGroup`: 22번은 내 IP만, 80번은 모든 IP에서 접근할 수 있게 합니다.
- `WebServer`: 위 보안 그룹을 사용하는 EC2 인스턴스이며, 시작할 때 `UserData` 스크립트로
  Apache httpd를 설치하고 실행합니다.
- `Outputs`: 생성된 인스턴스의 퍼블릭 DNS 이름을 출력합니다.

> 22번 포트를 `0.0.0.0/0`으로 열면 전 세계에서 SSH 접속 시도가 들어옵니다. 예제나
> 블로그 템플릿에 그렇게 되어 있더라도 내 IP(`/32`)로 제한합니다.

### 슬라이드 예제와 다른 점

슬라이드의 예제는 Elastic IP도 함께 만들지만, 이 실습에서는 제외했습니다. 현재 AWS는 퍼블릭 IPv4
주소에 시간당 요금을 부과하므로, 실습에 꼭 필요하지 않은 고정 IP는 만들지 않습니다. 참고용으로 Elastic
IP를 인스턴스에 연결하는 리소스는 다음과 같이 선언합니다.

```yaml
  WebServerEIP:
    Type: AWS::EC2::EIP
    Properties:
      InstanceId: !Ref WebServer
```

- 근거:
  [Amazon VPC 요금 - 퍼블릭 IPv4 주소](https://aws.amazon.com/vpc/pricing/)

## 2단계: 템플릿 문법 검사

`~/cfn-lab` 디렉터리에서 실행합니다.

```shell
aws cloudformation validate-template \
  --template-body file://first-stack.yaml \
  --region ap-northeast-2
```

YAML 문법이나 템플릿 구조에 문제가 없으면 템플릿의 `Parameters`와 `Description` 정보가
출력됩니다. 오류가 있으면 `ValidationError`와 함께 원인이 표시됩니다. 이 명령은 문법만 확인하므로,
속성 값이 올바른지(예: 존재하지 않는 인스턴스 타입)는 실제 배포에서 확인됩니다.

## 3단계: 스택 생성

내 공인 IP를 조회해 `SshCidr` 파라미터로 넘기면서 스택을 만듭니다.

```shell
aws cloudformation deploy \
  --stack-name cfn-first-stack \
  --template-file first-stack.yaml \
  --parameter-overrides SshCidr=$(curl -s https://checkip.amazonaws.com)/32 \
  --region ap-northeast-2
```

- `cfn-first-stack`은 예시 스택 이름입니다. 같은 리전에 같은 이름의 스택이 있으면 다른 이름을
  사용합니다.
- `https://checkip.amazonaws.com`은 요청한 쪽의 공인 IP를 돌려주는 AWS
  엔드포인트입니다.
- 회사 프록시나 VPN을 사용하면 실제 SSH 접속 IP와 조회된 IP가 다를 수 있습니다.
- `deploy` 명령은 스택 생성이 끝날 때까지 기다린 뒤 성공 또는 실패 메시지를 출력합니다.

## 4단계: 생성 결과 확인

스택 상태를 확인합니다.

```shell
aws cloudformation describe-stacks \
  --stack-name cfn-first-stack \
  --query "Stacks[0].StackStatus" \
  --output text \
  --region ap-northeast-2
```

정상이면 `CREATE_COMPLETE`가 출력됩니다. `ROLLBACK_COMPLETE`라면 생성에 실패한 것이므로,
아래 명령으로 실패 이벤트를 확인합니다.

```shell
aws cloudformation describe-stack-events \
  --stack-name cfn-first-stack \
  --query "StackEvents[?contains(ResourceStatus, 'FAILED')].[LogicalResourceId,ResourceStatusReason]" \
  --output table \
  --region ap-northeast-2
```

생성에 성공했다면 `Outputs`에서 퍼블릭 DNS 이름을 확인하고 웹 서버에 요청합니다.

```shell
aws cloudformation describe-stacks \
  --stack-name cfn-first-stack \
  --query "Stacks[0].Outputs" \
  --output table \
  --region ap-northeast-2

# 위 출력의 PublicDnsName 값으로 바꿔서 실행
curl http://<PublicDnsName>
```

- `<PublicDnsName>`은 자리표시자입니다. 출력된 `ec2-...amazonaws.com` 형태의 값으로
  바꿉니다.
- `UserData` 스크립트는 인스턴스 부팅 후 실행되므로, 스택이 `CREATE_COMPLETE`가 된 직후에는
  응답이 없을 수 있습니다. 1~2분 뒤 다시 시도합니다.
- 정상이면 `<h1>Hello from CloudFormation</h1>`이 응답으로 돌아옵니다.

콘솔에서도 CloudFormation → 스택 → `cfn-first-stack`의 **Resources** 탭에서
만들어진 보안 그룹과 인스턴스를 확인할 수 있습니다.

## 5단계: 스택 업데이트

템플릿을 고친 뒤 같은 명령으로 다시 배포하면 스택이 업데이트됩니다. `first-stack.yaml`의
`WebServer` 리소스 `Properties` 아래에 `Tags`를 추가합니다.

```yaml
  WebServer:
    Type: AWS::EC2::Instance
    Properties:
      ImageId: !Ref LatestAmiId
      InstanceType: t3.micro
      SecurityGroupIds:
        - !GetAtt WebServerSecurityGroup.GroupId
      Tags:
        - Key: Name
          Value: cfn-first-web-server
      UserData: !Base64 |
        #!/bin/bash
        dnf install -y httpd
        echo "<h1>Hello from CloudFormation</h1>" > /var/www/html/index.html
        systemctl enable --now httpd
```

바로 반영하기 전에 어떤 변경이 일어나는지 먼저 보고 싶다면 `--no-execute-changeset`을 붙여 변경
세트(Change Set)만 만듭니다.

```shell
aws cloudformation deploy \
  --stack-name cfn-first-stack \
  --template-file first-stack.yaml \
  --parameter-overrides SshCidr=$(curl -s https://checkip.amazonaws.com)/32 \
  --no-execute-changeset \
  --region ap-northeast-2
```

명령이 끝나면 변경 세트를 확인하는 `describe-change-set` 명령이 안내됩니다. 변경 세트의
`Changes`에서 `WebServer`의 `Action`이 `Modify`이고 `Replacement`가
`False`인지 확인합니다. `Tags`는 공식 문서에서 업데이트 시 중단이 없는(`No interruption`)
속성으로 안내되어 있으므로 인스턴스를 교체하지 않습니다.

검토가 끝나면 `--no-execute-changeset` 없이 같은 `deploy` 명령을 다시 실행하거나, 안내된
변경 세트 ARN으로 `execute-change-set`을 실행합니다.

```shell
aws cloudformation execute-change-set \
  --change-set-name <변경 세트 ARN> \
  --region ap-northeast-2
```

업데이트가 끝나면 스택 상태가 `UPDATE_COMPLETE`가 되고, EC2 콘솔에서 인스턴스 이름이
`cfn-first-web-server`로 표시됩니다.

- 공식 문서:
  [AWS::EC2::Instance 속성과 업데이트 동작](https://docs.aws.amazon.com/AWSCloudFormation/latest/TemplateReference/aws-resource-ec2-instance.html)

## 6단계: 콘솔에서 직접 바꾸면 안 되는 이유

스택이 만든 리소스를 EC2 콘솔에서 직접 수정하면 템플릿과 실제 상태가 달라집니다. 이를 **드리프트(drift)**
라고 합니다. 드리프트가 생기면 다음 스택 업데이트에서 예상하지 못한 변경이나 실패가 생길 수 있으므로, 변경은 항상
템플릿을 고쳐 배포합니다.

## 7단계: 스택 삭제

> 스택을 삭제하면 이 스택이 만든 EC2 인스턴스와 보안 그룹이 함께 삭제되며
> 되돌릴 수 없습니다. 실습용 스택 이름이 맞는지 확인한 뒤 실행합니다.

```shell
aws cloudformation delete-stack \
  --stack-name cfn-first-stack \
  --region ap-northeast-2

aws cloudformation wait stack-delete-complete \
  --stack-name cfn-first-stack \
  --region ap-northeast-2
```

`wait` 명령은 삭제가 끝날 때까지 기다렸다가 아무 출력 없이 종료됩니다. 삭제가 끝난 뒤 스택을 조회하면 스택이
존재하지 않는다는 오류가 나오는 것이 정상입니다.

```shell
aws cloudformation describe-stacks \
  --stack-name cfn-first-stack \
  --region ap-northeast-2
```

EC2 콘솔에서도 인스턴스가 `terminated` 상태인지 확인합니다. 삭제가 `DELETE_FAILED`로 끝나면
`describe-stack-events`로 원인을 확인한 뒤 다시 삭제합니다.

## 정리

- 템플릿 파일 하나로 보안 그룹과 EC2를 만들고, 스택 삭제로 한 번에 정리할 수 있습니다.
- `validate-template`은 문법 검사, `deploy`는 생성과 업데이트, `delete-stack`은
  삭제에 사용합니다.
- `--no-execute-changeset`으로 변경 세트를 먼저 만들면 업데이트 전에 교체 여부를 확인할 수
  있습니다.
- AMI ID는 SSM 공개 파라미터로 받아 리전 의존성을 줄이고, SSH 포트는 내 IP로 제한합니다.
- 스택이 만든 리소스는 콘솔에서 직접 고치지 않고 템플릿으로 변경합니다.

## 시험 대비 문제

### 문제 1

개발자가 운영 중인 스택의 템플릿을 수정했습니다. 실제로 반영하기 전에 어떤 리소스가 추가·수정·교체되는지 미리
확인하려고 합니다. 가장 적절한 방법은 무엇입니까?

- A. `validate-template`을 실행해 변경 내용을 확인한다.
- B. 변경 세트(Change Set)를 만들어 변경 내용을 검토한 뒤 실행한다.
- C. 스택을 삭제하고 새 템플릿으로 다시 생성한다.
- D. EC2 콘솔에서 먼저 리소스를 수정해 본다.

#### 정답

B

#### 해설

변경 세트는 현재 스택과 새 템플릿을 비교해 어떤 리소스가 추가·수정·삭제되고, 교체(`Replacement`)가
일어나는지 보여 줍니다. 검토 후 실행해야 실제로 반영됩니다. `validate-template`은 템플릿 문법만
확인하며, 실제 리소스에 어떤 변화가 생길지는 알려 주지 않습니다.

### 문제 2

CloudFormation으로 만든 EC2 인스턴스의 보안 그룹 규칙을 운영자가 EC2 콘솔에서 직접 수정했습니다.
이후 스택 업데이트에서 예상하지 못한 결과가 생겼습니다. 이 상황을 가리키는 용어와 올바른 대응은 무엇입니까?

- A. 롤백(Rollback) - 스택을 삭제하고 다시 만든다.
- B. 드리프트(Drift) - 변경은 템플릿을 수정해 스택 업데이트로 반영한다.
- C. 변경 세트(Change Set) - 콘솔 변경 내용을 변경 세트로 저장한다.
- D. 스택 정책(Stack Policy) - 보안 그룹에 IAM 정책을 붙인다.

#### 정답

B

#### 해설

스택 밖에서 리소스를 직접 바꿔 템플릿과 실제 상태가 달라진 것을 드리프트라고 합니다. CloudFormation은
드리프트 감지 기능으로 차이를 확인할 수 있으며, 근본적으로는 모든 변경을 템플릿으로 관리해야 합니다. 롤백은 배포
실패 시 이전 상태로 되돌리는 동작이고, 스택 정책은 스택 업데이트 중 특정 리소스 변경을 막는 기능입니다.
