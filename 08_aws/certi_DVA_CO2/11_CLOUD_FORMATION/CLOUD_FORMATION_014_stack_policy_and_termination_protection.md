# CloudFormation Stack Policy와 Termination Protection

## 왜 보호 장치가 필요한가요?

CloudFormation은 템플릿을 바꾸고 스택을 업데이트하면 필요한 리소스를 수정하거나 교체합니다. 편리한 만큼,
실수로 바꾼 속성 하나 때문에 운영 데이터베이스가 교체되거나 스택 전체가 삭제될 수도 있습니다.

이런 사고를 막기 위해 CloudFormation은 두 가지 보호 장치를 제공합니다.

- **Stack Policy**: 스택 **업데이트** 중 특정 리소스의 변경을 막습니다.
- **Termination Protection**: 스택 **삭제** 자체를 막습니다.

## Stack Policy

### 동작 원리

Stack Policy는 스택 업데이트 중 어떤 리소스에 어떤 변경을 허용할지 정하는 JSON 문서입니다.

- Stack Policy가 없으면 모든 리소스에 대한 모든 업데이트가 허용됩니다.
- Stack Policy를 설정하면 **기본적으로 모든 리소스가 보호**됩니다. 변경을 허용할 리소스에는 명시적으로
  `Allow`를 지정해야 합니다.
- `Allow`와 `Deny`가 겹치면 항상 `Deny`가 우선합니다.
- 스택 업데이트에만 적용됩니다. IAM 정책처럼 사용자별 권한을 제어하는 기능이 아니며, 스택에는 하나의 정책만
  연결할 수 있습니다.

- 공식 문서:
  [Prevent updates to stack resources](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/protect-stack-resources.html)
  (확인일: 2026-10-13)

### 작성 예시: 운영 DB만 보호하기

아래 정책은 모든 리소스의 업데이트를 허용하되, 논리 ID가 `ProductionDatabase`인 리소스의 업데이트만
거부합니다.

```json
{
  "Statement": [
    {
      "Effect": "Allow",
      "Action": "Update:*",
      "Principal": "*",
      "Resource": "*"
    },
    {
      "Effect": "Deny",
      "Action": "Update:*",
      "Principal": "*",
      "Resource": "LogicalResourceId/ProductionDatabase"
    }
  ]
}
```

| 요소        | 의미                                                                 |
| ----------- | -------------------------------------------------------------------- |
| `Effect`    | `Allow` 또는 `Deny`                                                  |
| `Action`    | 허용하거나 막을 업데이트 종류                                        |
| `Principal` | 필수 요소이며 `*`만 사용할 수 있습니다.                              |
| `Resource`  | `LogicalResourceId/<논리 ID>` 형식으로 대상 리소스를 지정합니다.     |
| `Condition` | `ResourceType` 조건으로 리소스 타입 단위 지정이 가능합니다.          |

`Action`에 사용할 수 있는 값은 다음과 같습니다.

| Action           | 의미                                                         |
| ---------------- | ------------------------------------------------------------ |
| `Update:Modify`  | 물리 ID를 유지한 채 속성을 변경하는 업데이트                 |
| `Update:Replace` | 리소스를 새로 만들고 이전 리소스를 지우는 교체 업데이트      |
| `Update:Delete`  | 템플릿에서 리소스를 제거해 삭제하는 업데이트                 |
| `Update:*`       | 위의 모든 업데이트                                           |

특정 리소스를 보호할 때는 `NotResource`와 `Allow`를 조합하지 말고, 위 예시처럼 명시적인
`Deny`를 사용합니다. 공식 문서는 `NotResource` 방식이 리소스를 안정적으로 보호하지 못한다고
안내합니다.

### 설정과 변경

정책 파일은 실행 위치에 `stack-policy.json`으로 저장했다고 가정합니다. `my-web-stack`은
예시 스택 이름입니다.

```shell
# 기존 스택에 Stack Policy 설정 또는 변경
aws cloudformation set-stack-policy \
  --stack-name my-web-stack \
  --stack-policy-body file://stack-policy.json
```

- 한 번 설정한 Stack Policy는 삭제할 수 없고 수정만 할 수 있습니다. 보호를 모두 풀려면 모든 리소스에
  `Update:*`를 허용하는 정책으로 바꿉니다.
- 스택을 만들 때는 `create-stack`의 `--stack-policy-body` 옵션으로 함께 설정할 수
  있습니다.

### 보호된 리소스를 꼭 바꿔야 할 때

보호된 리소스를 의도적으로 변경해야 한다면, 업데이트할 때만 적용되는 임시 정책을 함께 전달합니다. 임시 정책은 이번
업데이트에만 적용되고 기존 Stack Policy는 바뀌지 않습니다.

```shell
aws cloudformation update-stack \
  --stack-name my-web-stack \
  --template-body file://template.yaml \
  --stack-policy-during-update-body file://allow-all-policy.json
```

`allow-all-policy.json`은 모든 리소스에 `Update:*`를 허용하는 정책 파일의 예시 이름입니다.
이 작업에는 `cloudformation:SetStackPolicy` 권한이 필요하며, 운영 DB 교체처럼 데이터가
사라질 수 있는 변경이라면 먼저 Change Set으로 변경 내용을 확인하고 백업을 준비합니다.

## Termination Protection

### 동작 원리

Termination Protection을 켜면 스택 삭제 요청이 실패하고 스택은 그대로 유지됩니다.

- 기본값은 비활성화입니다.
- 스택을 만들 때 또는 이후 언제든 켤 수 있습니다. 단, `DELETE_IN_PROGRESS`나
  `DELETE_COMPLETE` 상태에서는 설정할 수 없습니다.
- 중첩 스택(Nested Stack)은 루트 스택의 설정을 따르며, 중첩 스택에서 직접 바꿀 수 없습니다.
- 설정을 바꾸려면 `cloudformation:UpdateTerminationProtection` 권한이 필요합니다.
- 스택 업데이트는 막지 않습니다. 업데이트 보호는 Stack Policy의 역할입니다.

- 공식 문서:
  [Protect CloudFormation stacks from being deleted](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/using-cfn-protect-stacks.html)

### 설정 명령

```shell
# Termination Protection 활성화
aws cloudformation update-termination-protection \
  --enable-termination-protection \
  --stack-name my-web-stack

# 설정 확인: True가 출력되면 활성화된 상태입니다
aws cloudformation describe-stacks \
  --stack-name my-web-stack \
  --query "Stacks[0].EnableTerminationProtection"
```

정말로 스택을 지워야 할 때는 `--no-enable-termination-protection`으로 해제한 뒤
삭제합니다. 삭제하면 `DeletionPolicy`로 보존하지 않은 리소스는 함께 삭제되므로, 해제 전에 삭제 대상과
백업 여부를 확인합니다.

Termination Protection은 "롤백 비활성화"와도 다른 기능입니다. 롤백 비활성화는 생성 실패 시 자동
롤백을 멈추는 옵션이며, 자세한 내용은
[Rollbacks](CLOUD_FORMATION_011_rollbacks.md)에서 다룹니다.

## 보호 장치 비교

| 기능                   | 막는 상황                         | 적용 단위   | 설정 위치                         |
| ---------------------- | --------------------------------- | ----------- | --------------------------------- |
| Stack Policy           | 스택 업데이트 중 리소스 변경·교체·삭제 | 리소스   | 스택에 연결하는 JSON 정책         |
| Termination Protection | 스택 삭제                         | 스택        | 스택 옵션                         |
| `DeletionPolicy`       | 삭제 시 리소스와 데이터 손실      | 리소스      | 템플릿의 리소스 속성              |

세 기능은 서로 대체하지 않으므로, 운영 스택에는 함께 사용하는 경우가 많습니다. `DeletionPolicy`는
[DeletionPolicy](CLOUD_FORMATION_013_deletion_policy.md)에서 다룹니다.

## 정리

- Stack Policy는 스택 업데이트 중 특정 리소스의 변경을 막는 JSON 문서입니다.
- Stack Policy를 설정하면 모든 리소스가 기본 보호되며, 변경할 리소스에는 명시적인 `Allow`가
  필요합니다.
- Stack Policy는 삭제할 수 없고 수정만 가능하며, 업데이트 시 임시 정책으로 일시적으로 덮어쓸 수
  있습니다.
- Termination Protection은 스택 삭제를 막으며 기본값은 비활성화입니다.

## 시험 대비 문제

### 문제 1

운영 스택에는 웹 서버와 `ProductionDatabase`라는 RDS 인스턴스가 있습니다. 웹 서버는 자유롭게
업데이트하되, 스택 업데이트 중 실수로 데이터베이스가 교체되거나 삭제되는 일은 막고 싶습니다. 가장 적절한 방법은
무엇입니까?

- A. 스택에 Termination Protection을 활성화한다.
- B. 모든 리소스에 `Update:*`를 허용하고 `ProductionDatabase`에는 `Update:*`를
  거부하는 Stack Policy를 설정한다.
- C. 개발자의 IAM 정책에서 `cloudformation:UpdateStack`을 거부한다.
- D. `ProductionDatabase`에 `DeletionPolicy: Retain`만 지정한다.

#### 정답

B

#### 해설

Stack Policy는 스택 업데이트 중 특정 리소스에 대한 변경을 막는 기능입니다. A는 스택 삭제만 막고, C는
웹 서버 업데이트까지 막습니다. D는 리소스가 삭제될 때 물리 리소스를 남겨 둘 뿐, 업데이트로 인한 변경이나 교체를
막지는 않습니다.

### 문제 2

여러 팀원이 같은 AWS 계정에서 CloudFormation 콘솔을 사용합니다. 운영 스택이 실수로 삭제되는 것을 막고
싶지만, 정상적인 스택 업데이트는 계속 허용해야 합니다. 가장 간단한 방법은 무엇입니까?

- A. 스택에 Termination Protection을 활성화한다.
- B. 모든 업데이트를 거부하는 Stack Policy를 설정한다.
- C. 스택 생성 시 롤백을 비활성화한다.
- D. 템플릿을 S3 버킷에 버전 관리와 함께 저장한다.

#### 정답

A

#### 해설

Termination Protection은 스택 삭제 요청을 실패시키며, 스택 업데이트는 막지 않습니다. B는
업데이트까지 막고, C는 생성 실패 시 롤백 동작에 대한 설정으로 삭제 방지와 관계가 없습니다.
