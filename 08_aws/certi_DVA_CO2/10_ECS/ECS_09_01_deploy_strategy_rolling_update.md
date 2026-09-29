# ECS 배포 전략 - 롤링 업데이트

## 🚀 Rolling Update

- Rolling Update는 ECS 서비스의 기본 배포 방식입니다.
- 기존 Task를 하나씩 종료하면서 새로운 Task를 하나씩 실행합니다.
- 새 태스크가 정상 상태가 될 때까지 교체를 반복합니다.

```text
배포 전:       v1  v1  v1  v1
배포 중:       v1  v1  v2  v2
배포 완료:     v2  v2  v2  v2
```

## 장점

- ECS 기본 배포 방식입니다.
- 다운타임이 거의 없습니다.
- 추가 용량을 적게 사용하면서 기본 기능만으로 배포할 수 있습니다.
- 교체 중 v1과 v2가 동시에 요청을 처리할 수 있습니다.

## 주의점

- API와 데이터베이스 변경은 두 버전이 함께 동작하도록 하위 호환성을 유지해야 합니다.
- `maximumPercent`만큼의 CPU·메모리 또는 Fargate 용량이 없으면 새 태스크가 대기하여 배포가
  느려질 수 있습니다.
- 버전이 다르기 때문에, DB Schema 변경이 큰 경우 문제가 생길 수 있습니다.

## Deployment configuration

Rolling Update의 교체 속도와 가용성은 다음 두 값으로 조절합니다.

| 설정                    | 의미                                    | 높게 설정했을 때                       |
| ----------------------- | --------------------------------------- | -------------------------------------- |
| `minimumHealthyPercent` | 배포 중 유지할 최소 정상 태스크 비율    | 기존 서비스의 여유가 커집니다.         |
| `maximumPercent`        | 배포 중 실행할 수 있는 최대 태스크 비율 | 새 태스크를 먼저 띄울 여유가 커집니다. |

### Minimum Healthy Percent

- 배포 중 최소 몇 %의 Task를 살아있게 유지할 것인가

예를 들어

```
Task 4개
Minimum Healthy = 50%
```

이면, 최소 2개는 항상 살아 있어야 합니다.

### Maximum Percent

- 배포 중, 최대 몇 %까지 Task를 실행할 것인가

예를 들어

```
Task 4개
Maximum = 200%
```

이면, 최대 8개 까지 실행 가능합니다.

### 예제 1

```shell
Desired Task = 4
Minimum = 50%
Maximum = 100%

# 최소 Task - 2개는 살아있어야 한다.
# 최대 Task - 4개 까지만 가능하다.
# 새 Task를 만들 공간이 없으므로, 먼저 기존 Task 종료 -> 새 Task 생성 -> 기존 Task 종료.. 반복

# ==============================================
Task
Task
Task
Task

↓
종료

Task
Task
Task

↓
새 Task 생성

Task
Task
Task
Task(v2)

↓
계속 반복
메모리는 적게 사용하지만 서비스 여유가 적다.
```

### 예제 2

```shell
Desired = 4
Minimum = 100%
Maximum = 150%

# 최소 Task - 4개는 살아있어야 한다.
# 최대 Task - 6개 까지 실행 가능하다.
# 즉, 새로운 Task를 먼저 만든다.
# ==============================================
Task, Task, Task, Task

↓

Task, Task, Task, Task, Task(v2), Task(v2)

↓

Health Check 통과

↓

기존 Task 종료

Task, Task, Task(v2), Task(v2), Task(v2), Task(v2)
```

## 추가 설명

- `minimumHealthyPercent`가 50이면, ECS는 배포 중 정상 태스크를 최소 2개 이상 유지하려고 합니다.
  `maximumPercent`가 100이면 최대 4개만 실행할 수 있으므로, 새 태스크를 만들기 전에 기존 태스크를
  일부 중지해야 합니다.

- `minimumHealthyPercent`가 100이고 `maximumPercent`가 150이면 정상 태스크 4개를 유지하면서
  최대 6개까지 실행할 수 있습니다. 새 태스크를 먼저 시작하고 헬스 체크가 통과한 후 기존 태스크를 중지할 수 있어,
  여유 용량이 있다면 더 안전합니다.

### 실패 감지와 롤백

`ECS` 컨트롤러에서는 deployment circuit breaker를 켜서 새 태스크가 실행 상태가 되지 않거나,
헬스 체크에 반복 실패할 때 배포를 실패로 판단하게 할 수 있습니다.
`rollback`도 함께 켜면 가장 최근에 완료된 배포로 자동 되돌릴 수 있습니다.

- Circuit breaker는 Rolling Update에만 적용됩니다.
- Blue/Green, Canary, Linear는 선택한 ECS 전략의 실패 감지 설정을 확인해야 합니다.
- `CODE_DEPLOY` 컨트롤러를 사용한다면 CodeDeploy의 자동 롤백 설정과 CloudWatch alarm으로
  실패 감지 조건을 구성합니다.
