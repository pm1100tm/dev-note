# CloudFormation Service Role과 Capabilities

## 이 챕터에서 다루는 것

CloudFormation이 리소스를 만들 때 **누구의 권한으로** 작업하는지, 그리고 위험할 수 있는 템플릿을
배포할 때 **사용자가 무엇을 명시적으로 승인해야 하는지**를 정리합니다. 둘 다 권한과 보안에 관한 내용이라 시험에
자주 나옵니다.

## Service Role

### 배경: 기본은 사용자 권한으로 동작합니다

별도 설정이 없으면 CloudFormation은 스택을 실행한 사용자의 자격 증명으로 만든 임시 세션을 사용해 리소스를
만듭니다. 그래서 S3 버킷을 만드는 템플릿을 배포하려면, 사용자 자신에게도 S3 버킷 생성 권한이 있어야 합니다.

이 방식은 개발자가 스택을 배포하려면 템플릿에 들어 있는 모든 서비스의 권한을 직접 가져야 한다는 문제가 있습니다.
최소 권한 원칙(Least Privilege)을 지키기 어렵습니다.

### Service Role이란?

**Service Role**은 CloudFormation이 스택 리소스를 생성·변경·삭제할 때 대신 사용하는 IAM
역할입니다. Service Role을 지정하면 CloudFormation은 사용자 권한 대신 이 역할의 권한으로
작업합니다.

```shell
사용자 권한: cloudformation:* + iam:PassRole   (S3 권한 없음)
        ↓ 스택 생성 요청 (--role-arn 지정)
CloudFormation이 Service Role을 맡음
        ↓ Service Role 권한: s3:*Bucket
S3 버킷 생성
```

사용자는 S3 권한이 없어도, 허용된 Service Role을 통해 S3 버킷이 들어 있는 스택을 배포할 수 있습니다.

### 필요한 권한

| 대상         | 필요한 권한                                                                  |
| ------------ | ---------------------------------------------------------------------------- |
| 사용자       | 스택 작업 권한(`cloudformation:CreateStack` 등)과 역할을 넘길 수 있는 `iam:PassRole` |
| Service Role | 템플릿의 리소스를 만들 권한(예: `s3:CreateBucket`)                          |
| Service Role 신뢰 정책 | `cloudformation.amazonaws.com`이 역할을 맡을 수 있도록 허용        |

`iam:PassRole`은 "이 역할을 AWS 서비스에 넘겨 사용하게 할 수 있다"는 권한입니다. 이 권한이 없으면
사용자는 Service Role을 지정할 수 없습니다.

Service Role의 신뢰 정책(Trust Policy) 예시는 다음과 같습니다.

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": { "Service": "cloudformation.amazonaws.com" },
      "Action": "sts:AssumeRole"
    }
  ]
}
```

### CLI에서 지정하기

```shell
aws cloudformation create-stack \
  --stack-name my-s3-stack \
  --template-body file://template.yaml \
  --role-arn arn:aws:iam::123456789012:role/<CloudFormation 서비스 역할 이름>
```

- 계정 ID `123456789012`와 `<CloudFormation 서비스 역할 이름>`은 실제 계정 ID와 역할
  이름으로 바꿉니다.

### 주의 사항

- 스택에 Service Role을 지정하면 CloudFormation은 이후 그 스택의 **모든 작업**에 이 역할을
  사용합니다.
- 스택 생성 후에는 연결된 Service Role을 제거할 수 없습니다.
- 스택 작업 권한만 있으면 `iam:PassRole`이 없는 다른 사용자도 이미 연결된 역할로 작업할 수 있습니다.
  역할 권한이 넓으면 의도치 않게 권한이 상승할 수 있으므로, Service Role에도 최소 권한만 부여합니다.
- 근거:
  [CloudFormation service role](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/using-iam-servicerole.html)
  (확인일: 2026-10-13)

## Capabilities

### 배경: 위험한 템플릿은 명시적으로 승인합니다

IAM 사용자나 역할을 만드는 템플릿은 계정의 권한 구조를 바꿀 수 있습니다. 또 매크로를 쓰는 템플릿은 배포 직전에
내용이 바뀔 수 있습니다.

그래서 CloudFormation은 이런 템플릿을 배포할 때 사용자가 **"이 템플릿이 무엇을 하는지 알고 있다"**
고 확인하도록 요구합니다. 이 확인 값을 **Capabilities**라고 합니다.

### Capabilities 종류

| 값                       | 언제 필요한가                                                                        |
| ------------------------ | ------------------------------------------------------------------------------------ |
| `CAPABILITY_IAM`         | 템플릿이 IAM 리소스(User, Role, Group, Policy, AccessKey, InstanceProfile 등)를 만들거나 수정할 때 |
| `CAPABILITY_NAMED_IAM`   | IAM 리소스에 `RoleName`, `UserName`처럼 **이름을 직접 지정**했을 때                  |
| `CAPABILITY_AUTO_EXPAND` | 템플릿에 매크로(`AWS::Serverless`, `AWS::Include` Transform 포함)가 있고, 변경 세트로 검토하지 않고 바로 스택을 만들 때 |

- 이름 없는 IAM 리소스만 있으면 `CAPABILITY_IAM`과 `CAPABILITY_NAMED_IAM` 중
  아무것이나 지정해도 됩니다.
- 이름을 지정한 IAM 리소스가 있으면 반드시 `CAPABILITY_NAMED_IAM`을 지정해야 합니다.
- 매크로와 중첩 스택을 함께 쓰는 템플릿은 `CAPABILITY_AUTO_EXPAND`를 지정해 템플릿에서 바로
  스택을 만들어야 합니다.
- 근거:
  [CreateStack API의 Capabilities](https://docs.aws.amazon.com/AWSCloudFormation/latest/APIReference/API_CreateStack.html),
  [Acknowledging IAM resources](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/control-access-with-iam.html#using-iam-capabilities)
  (확인일: 2026-10-13)

### 승인하지 않으면 생기는 오류

필요한 Capabilities를 지정하지 않으면 CloudFormation은 배포를 거부하고
`InsufficientCapabilities` 오류를 반환합니다. SDK에서는
`InsufficientCapabilitiesException` 예외로 나타납니다. 실수로 IAM 리소스가 만들어지는
것을 막는 보안 장치입니다.

콘솔에서는 **I acknowledge that this template may create IAM
resources** 체크박스가 같은 역할을 합니다.

### 예시

다음 템플릿은 이름을 지정한 IAM 역할을 만들므로 `CAPABILITY_NAMED_IAM`이 필요합니다.

```yaml
Resources:
  AppRole:
    Type: AWS::IAM::Role
    Properties:
      RoleName: my-app-role          # 이름을 직접 지정
      AssumeRolePolicyDocument:
        Version: "2012-10-17"
        Statement:
          - Effect: Allow
            Principal:
              Service: ec2.amazonaws.com
            Action: sts:AssumeRole
```

```shell
aws cloudformation deploy \
  --stack-name my-iam-stack \
  --template-file iam-role.yaml \
  --capabilities CAPABILITY_NAMED_IAM
```

- 이름을 지정한 IAM 리소스는 계정 안에서 이름이 유일해야 합니다. 같은 템플릿으로 여러 리전에 스택을 만들면
  이름이 충돌하거나 IAM 리소스를 공유하게 되므로 주의합니다.

## 정리

- Service Role은 CloudFormation이 스택 리소스를 다룰 때 사용하는 IAM 역할이며, 최소 권한
  원칙을 지키는 데 씁니다.
- Service Role을 지정하려면 사용자에게 `iam:PassRole` 권한이 필요하고, CLI에서는
  `--role-arn`으로 지정합니다.
- IAM 리소스를 만들면 `CAPABILITY_IAM`, 이름을 지정했으면 `CAPABILITY_NAMED_IAM`이
  필요합니다.
- 매크로나 중첩 스택을 바로 배포하려면 `CAPABILITY_AUTO_EXPAND`가 필요합니다.
- 필요한 Capabilities가 없으면 `InsufficientCapabilities` 오류가 발생합니다.

## 시험 대비 문제

### 문제 1

보안팀은 개발자에게 S3 권한을 직접 주지 않으면서도, 개발자가 S3 버킷이 포함된 CloudFormation 스택을
배포할 수 있게 하려고 합니다. 가장 적절한 방법은 무엇입니까?

- A. 개발자에게 `AdministratorAccess` 정책을 부여한다.
- B. S3 권한을 가진 CloudFormation Service Role을 만들고, 개발자에게
  `cloudformation:*`와 `iam:PassRole` 권한을 준다.
- C. 템플릿에 `CAPABILITY_IAM`을 지정한다.
- D. S3 버킷 정책에 개발자 IAM 사용자를 추가한다.

#### 정답

B

#### 해설

Service Role을 사용하면 CloudFormation이 사용자 대신 역할의 권한으로 리소스를 만듭니다. 개발자는
스택 작업 권한과 역할을 넘길 수 있는 `iam:PassRole`만 있으면 되므로 최소 권한 원칙을 지킬 수 있습니다.
`CAPABILITY_IAM`은 IAM 리소스 생성을 승인하는 값으로, 권한을 위임하는 기능이 아닙니다.

### 문제 2

개발자가 `RoleName` 속성으로 이름을 지정한 IAM 역할이 들어 있는 템플릿을 AWS CLI로 배포했더니
`InsufficientCapabilities` 오류가 발생했습니다. 어떻게 해결해야 합니까?

- A. `--capabilities CAPABILITY_IAM`을 지정한다.
- B. `--capabilities CAPABILITY_NAMED_IAM`을 지정한다.
- C. `--capabilities CAPABILITY_AUTO_EXPAND`를 지정한다.
- D. `--role-arn`으로 관리자 권한 역할을 지정한다.

#### 정답

B

#### 해설

템플릿에 IAM 리소스가 있으면 Capabilities로 이를 승인해야 합니다. 특히 IAM 리소스에 이름을 직접
지정한 경우에는 `CAPABILITY_IAM`으로는 부족하고 `CAPABILITY_NAMED_IAM`을 지정해야
합니다. `CAPABILITY_AUTO_EXPAND`는 매크로나 중첩 스택을 위한 값이고, Service Role을
지정해도 Capabilities 승인 요구는 사라지지 않습니다.
