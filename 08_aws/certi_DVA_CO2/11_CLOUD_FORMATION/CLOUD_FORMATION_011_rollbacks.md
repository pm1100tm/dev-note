# CloudFormation Rollbacks

## 왜 롤백을 알아야 하나요?

스택을 만들거나 업데이트하다 보면 권한 부족, 잘못된 속성 값, 서비스 할당량 초과 같은 이유로 일부 리소스가 실패할
수 있습니다. 이때 절반만 만들어진 인프라가 남아 있으면 무엇이 생성됐고 무엇이 실패했는지 파악하기 어렵습니다.

CloudFormation은 이런 상황에서 기본적으로 **롤백(Rollback)**, 즉 작업 전 상태로 되돌리는
동작을 수행합니다. 실패했을 때 스택이 어떤 상태가 되는지, 어떻게 복구하는지가 시험에서 자주 나옵니다.

## 스택 생성이 실패했을 때

| 설정                         | 동작                                                            | 결과 상태              |
| ---------------------------- | --------------------------------------------------------------- | ---------------------- |
| 기본값(롤백)                 | 이미 만든 리소스를 모두 삭제                                    | `ROLLBACK_COMPLETE`    |
| 롤백 비활성화                | 성공한 리소스는 남기고, 실패한 리소스는 실패 상태로 둠          | `CREATE_FAILED`        |

### 기본 동작: 전부 되돌리기

생성 중 하나라도 실패하면 CloudFormation은 그때까지 만든 리소스를 모두 삭제합니다. 실패 원인은 스택
이벤트 로그에서 확인할 수 있습니다.

`ROLLBACK_COMPLETE` 상태의 스택은 업데이트할 수 없습니다. 원인을 고친 뒤 **스택을 삭제하고 다시
생성**해야 합니다.

### 롤백 비활성화: 원인 조사용

실패한 리소스를 그대로 남겨 두고 원인을 조사하고 싶다면 롤백을 끌 수 있습니다. 콘솔에서는 스택 옵션의 **Stack
failure options**에서 **Preserve successfully provisioned
resources**를 선택합니다.

```shell
# 생성 실패 시 롤백하지 않음
aws cloudformation create-stack \
  --stack-name my-web-stack \
  --template-body file://template.yaml \
  --disable-rollback

# 같은 의미: 실패 시 아무것도 하지 않음
aws cloudformation create-stack \
  --stack-name my-web-stack \
  --template-body file://template.yaml \
  --on-failure DO_NOTHING
```

- `--on-failure`에는 `ROLLBACK`(기본값), `DO_NOTHING`, `DELETE`를 지정할 수
  있으며, `--disable-rollback`과 함께 쓸 수는 없습니다.
- 조사가 끝나면 **Retry**, **Update**, **Roll back** 중 하나로 이어서 처리합니다.
  CLI에서는 `aws cloudformation rollback-stack`으로 `CREATE_FAILED` 또는
  `UPDATE_FAILED` 스택을 마지막 안정 상태로 되돌릴 수 있습니다.
- 롤백을 끄면 실패한 스택의 리소스가 남아 비용이 계속 발생할 수 있으므로, 조사가 끝나면 정리합니다.
- 근거:
  [Stack failure options](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/stack-failure-options.html),
  [CreateStack API](https://docs.aws.amazon.com/AWSCloudFormation/latest/APIReference/API_CreateStack.html)
  (확인일: 2026-10-13)

## 스택 업데이트가 실패했을 때

업데이트 중 실패하면 CloudFormation은 자동으로 **마지막으로 정상 동작하던 상태**로 되돌립니다.
되돌리기가 끝나면 스택은 `UPDATE_ROLLBACK_COMPLETE` 상태가 되며, 이 상태에서는 템플릿을 고쳐
다시 업데이트할 수 있습니다.

```shell
UPDATE_IN_PROGRESS
        ↓ (리소스 하나 실패)
UPDATE_ROLLBACK_IN_PROGRESS
        ↓
UPDATE_ROLLBACK_COMPLETE   ← 이전 정상 상태로 복구 완료
```

생성 실패와 마찬가지로, 무엇이 왜 실패했는지는 스택 이벤트 로그와 오류 메시지로 확인합니다.

## 롤백마저 실패했을 때

드물지만 되돌리는 작업 자체가 실패할 수 있습니다. 예를 들어 누군가 CloudFormation 밖에서 콘솔로 리소스를
직접 삭제해, 원래 상태로 돌아갈 대상이 없어진 경우입니다. 이때 스택은 `UPDATE_ROLLBACK_FAILED`
상태가 됩니다.

### 복구 순서

- 이벤트 로그에서 롤백에 실패한 리소스와 원인을 확인합니다.
- 원인이 된 리소스를 직접 고칩니다. 예를 들어 지워진 리소스를 다시 만들거나 권한을 추가합니다.
- 콘솔의 **Continue update rollback** 또는 CLI의
  `continue-update-rollback`으로 롤백을 이어서 진행합니다.

```shell
aws cloudformation continue-update-rollback \
  --stack-name my-web-stack
```

도저히 복구할 수 없는 리소스는 `--resources-to-skip`으로 건너뛸 수 있습니다. 다만 건너뛴 리소스는
템플릿과 실제 상태가 달라지므로, 다음 업데이트 전에 둘을 맞춰야 합니다.

```shell
aws cloudformation continue-update-rollback \
  --stack-name my-web-stack \
  --resources-to-skip MyBrokenResource
```

- `MyBrokenResource`는 예시 논리 ID이므로 이벤트 로그에서 확인한 실제 논리 ID로 바꿉니다.
- 근거:
  [AWS CLI `continue-update-rollback`](https://docs.aws.amazon.com/cli/latest/reference/cloudformation/continue-update-rollback.html)
  (확인일: 2026-10-13)

## 실패 원인 확인하기

```shell
aws cloudformation describe-stack-events \
  --stack-name my-web-stack \
  --query "StackEvents[?contains(ResourceStatus, 'FAILED')].[LogicalResourceId,ResourceStatus,ResourceStatusReason]" \
  --output table
```

이 명령은 이벤트 중 상태에 `FAILED`가 들어간 항목만 골라 논리 ID, 상태, 실패 이유를 표로 보여 줍니다.
가장 먼저 실패한 이벤트가 실제 원인인 경우가 많고, 뒤이은 실패는 그 결과로 생긴 경우가 많습니다.

## 정리

- 생성 실패 시 기본값은 전부 롤백이며, 결과 상태 `ROLLBACK_COMPLETE` 스택은 삭제 후 다시 만들어야
  합니다.
- 원인 조사가 필요하면 `--disable-rollback` 또는 `--on-failure DO_NOTHING`으로
  롤백을 끕니다.
- 업데이트 실패 시 마지막 정상 상태로 자동 롤백되어 `UPDATE_ROLLBACK_COMPLETE`가 됩니다.
- 롤백이 실패해 `UPDATE_ROLLBACK_FAILED`가 되면, 리소스를 직접 고친 뒤
  `ContinueUpdateRollback`을 실행합니다.

## 시험 대비 문제

### 문제 1

개발자가 새 CloudFormation 스택을 만들었는데 생성 도중 실패하고, 이미 만든 리소스가 모두 삭제되었습니다.
개발자는 다음 시도에서 실패한 리소스를 남겨 둔 채 원인을 조사하고 싶습니다. 어떻게 해야 합니까?

- A. 스택 정책(Stack Policy)으로 모든 리소스의 업데이트를 거부한다.
- B. 스택 생성 시 롤백을 비활성화(`--disable-rollback`)한다.
- C. 모든 리소스에 `DeletionPolicy: Retain`을 지정한다.
- D. 종료 보호(Termination Protection)를 활성화한다.

#### 정답

B

#### 해설

스택 생성 실패 시 기본 동작은 만든 리소스를 모두 삭제하는 롤백입니다. 롤백을 비활성화하면 성공한 리소스와 실패한
리소스가 남아 있어 원인을 조사할 수 있습니다. 스택 정책은 업데이트 보호, 종료 보호는 스택 삭제 방지 기능이며,
생성 실패 시 롤백 동작과는 관계가 없습니다.

### 문제 2

스택 업데이트가 실패한 뒤 자동 롤백도 실패해 스택이 `UPDATE_ROLLBACK_FAILED` 상태가 되었습니다.
확인해 보니 누군가 스택이 관리하던 리소스를 콘솔에서 직접 삭제했습니다. 가장 적절한 조치는 무엇입니까?

- A. 스택을 즉시 삭제하고 새로 만든다.
- B. 같은 템플릿으로 `update-stack`을 다시 실행한다.
- C. 원인이 된 리소스 문제를 직접 해결한 뒤 `ContinueUpdateRollback`을 실행한다.
- D. 스택에 Termination Protection을 활성화한다.

#### 정답

C

#### 해설

`UPDATE_ROLLBACK_FAILED` 상태에서는 일반 업데이트를 진행할 수 없습니다. 롤백을 막은 원인을 직접
고친 뒤 콘솔의 Continue update rollback 또는 CLI의
`continue-update-rollback`으로 롤백을 마무리합니다. 롤백이 끝나
`UPDATE_ROLLBACK_COMPLETE`가 되면 다시 스택을 업데이트할 수 있습니다.
