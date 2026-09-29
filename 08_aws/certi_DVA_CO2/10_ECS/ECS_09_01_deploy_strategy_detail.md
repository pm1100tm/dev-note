# ECS 배포 전략

- Rolling Update
- Blue/Green
- Canary
- Linear

## 🚀 Rolling Update

- Rolling Update는 ECS 서비스의 기본 배포 방식입니다.
- 기존 Task를 하나씩 종료하면서 새로운 Task를 하나씩 실행합니다.
- 새 태스크가 정상 상태가 될 때까지 교체를 반복합니다.

```text
배포 전:       v1  v1  v1  v1
배포 중:       v1  v1  v2  v2
배포 완료:     v2  v2  v2  v2
```

### 장점

- ECS 기본 배포 방식입니다.
- 다운타임이 거의 없습니다.
- 추가 용량을 적게 사용하면서 기본 기능만으로 배포할 수 있습니다.
- 교체 중 v1과 v2가 동시에 요청을 처리할 수 있습니다.

### 주의점

- API와 데이터베이스 변경은 두 버전이 함께 동작하도록 하위 호환성을 유지해야 합니다.
- `maximumPercent`만큼의 CPU·메모리 또는 Fargate 용량이 없으면 새 태스크가 대기하여 배포가
  느려질 수 있습니다.
- 버전이 다르기 때문에, DB Schema 변경이 큰 경우 문제가 생길 수 있습니다.

### Deployment configuration

Rolling Update의 교체 속도와 가용성은 다음 두 값으로 조절합니다.

| 설정                    | 의미                                    | 높게 설정했을 때                       |
| ----------------------- | --------------------------------------- | -------------------------------------- |
| `minimumHealthyPercent` | 배포 중 유지할 최소 정상 태스크 비율    | 기존 서비스의 여유가 커집니다.         |
| `maximumPercent`        | 배포 중 실행할 수 있는 최대 태스크 비율 | 새 태스크를 먼저 띄울 여유가 커집니다. |

#### Minimum Healthy Percent

- 배포 중 최소 몇 %의 Task를 살아있게 유지할 것인가

예를 들어

```
Task 4개
Minimum Healthy = 50%
```

이면, 최소 2개는 항상 살아 있어야 합니다.

#### Maximum Percent

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

### 추가 설명

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

<br>

---

## 🚀 Blue/Green

Blue/Green 배포는 현재 운영 중인 서비스 리비전을 Blue, 새 버전의 서비스 리비전을 Green으로 분리해
실행하는 배포 방식입니다. Green을 검증한 뒤 운영 트래픽을 Blue에서 Green으로 전환합니다.

AWS에서 가장 추천하는 방식입니다.

- 운영 중인 서비스 리비전: Blue
- 새 버전의 서비스 리비전: Green

두 버전을 동시에 실행하므로, 새 버전을 운영 트래픽에 노출하기 전에 헬스 체크와 테스트를 수행할 수 있습니다.
문제가 발생하면 Blue가 실행 중인 동안 트래픽을 되돌릴 수 있어 롤백 경로가 명확합니다.

```text
배포 전:       운영 트래픽 100% -> Blue  (v1)

Green 준비:    운영 트래픽 100% -> Blue  (v1)
                               Green (v2) 헬스 체크 및 테스트

전환 후:       운영 트래픽 100% -> Green (v2)
              Blue  (v1)는 bake time 동안 유지
```

즉, 기존 서비스(Blue)와 새 버전(Green)을, 2개를 동시에 띄웁니다.

```shell
# 기존 리비전 Blue
Task
Task
Task

# 새 리비전 Green
Task
Task
Task
```

Health Check가 끝나면 ALB가 트래픽을 한 번에 변경합니다.

```shell
Blue
100%

↓

Green
100%
```

### 동작 흐름

#### Green 환경 생성

새 태스크 정의로 서비스를 업데이트하면 ECS는 Green 서비스 리비전을 만들고, 필요한 수의 Green 태스크를
시작합니다. 이 단계에서도 Blue는 계속 운영 트래픽을 처리합니다.

Green 태스크가 컨테이너 헬스 체크와 로드 밸런서 대상 그룹의 헬스 체크를 통과해야 다음 단계로
진행할 수 있습니다.

애플리케이션이 실행 상태가 되었다고 해서 즉시 요청을 받을 준비가 된 것은 아니므로,
헬스 체크는 실제 준비 완료 상태를 확인하도록 설계해야 합니다.

#### 테스트와 트래픽 전환

- 테스트 리스너 또는 라우팅 규칙을 구성하면 운영 트래픽을 Blue에 유지한 채 테스트 요청만 Green으로 보낼 수 있습니다.
- 예를 들어, 특정 헤더나 별도 도메인의 요청을 Green으로 라우팅해 기능을 확인할 수 있습니다.
- 검증이 끝나면 ECS는 운영 트래픽을 Green으로 한 번에 전환합니다.
- Blue/Green 전략 자체의 운영 트래픽 전환은 All-at-once 방식입니다.
- 일부 트래픽부터 점진적으로 전환하려면 `CANARY` 또는 `LINEAR` 전략을 선택합니다.

#### Bake time과 완료

Bake time은 운영 트래픽을 Green으로 전환한 뒤에도 Blue와 Green을 함께 실행하는 시간입니다.
이 기간에는 오류율, 지연 시간, 비즈니스 지표를 관찰하고 문제가 있으면 Blue로 되돌릴 수 있습니다.

Bake time이 끝나고 배포가 성공하면 ECS는 Blue 태스크를 종료합니다.
따라서 빠른 롤백이 필요할수록 관찰에 충분한 bake time을 설정해야 하지만, 그만큼 두 환경을 함께 실행하는
비용도 늘어납니다.

### 트래픽 전환에 필요한 구성

자동 트래픽 전환을 사용하려면 다음 중 하나를 구성해야 합니다.

| 구성                      | 역할                                                          | 참고 사항                                              |
| ------------------------- | ------------------------------------------------------------- | ------------------------------------------------------ |
| Application Load Balancer | HTTP/HTTPS 요청을 Blue 또는 Green 대상 그룹으로 라우팅합니다. | 테스트 리스너와 세밀한 라우팅 규칙에 적합합니다.       |
| Network Load Balancer     | TCP/UDP 수준의 요청을 전환합니다.                             | 라이프사이클 단계에 추가 대기 시간이 생길 수 있습니다. |
| Service Connect           | 서비스 간 통신의 트래픽 전환을 관리합니다.                    | 내부 서비스 통신에 적합합니다.                         |

로드 밸런서를 사용하는 경우에는 Blue와 Green 태스크를 분리해 등록할 대상 그룹이 필요합니다.
운영 요청을 받는 리스너와, 필요에 따라 Green만 검증하기 위한 테스트 리스너 또는 규칙을 준비합니다.

로드 밸런서와 Service Connect를 사용하지 않는 headless 서비스도 Blue/Green 배포를 사용할 수 있습니다.
다만 ECS가 외부 요청의 트래픽 전환을 관리하지 않으므로, 호출 측 전환 방법을 별도로 설계해야 합니다.

### 롤백과 실패 감지

Blue/Green의 핵심은 새 버전이 실패해도 기존 Blue가 남아 있다는 점입니다.

Green의 헬스 체크가 실패하거나 배포 중 설정한 alarm이 경보 상태가 되면 배포 실패로 처리하고 롤백할 수 있습니다.

- CloudWatch alarm에는 5xx 오류 비율, 지연 시간, CPU·메모리 사용률처럼 서비스 장애를 나타내는
  지표를 연결합니다.
- lifecycle hook에는 Lambda 함수 또는 일시 정지 지점을 설정해, 트래픽 전환 전후에 통합 테스트를
  실행할 수 있습니다.
- 일시 정지 지점을 사용했다면 검증을 마친 뒤 명시적으로 배포를 계속 진행해야 합니다.
  방치하면 배포가 시간 초과로 실패할 수 있습니다.
- 롤백 절차는 실제 운영 전에 테스트합니다. Blue가 종료된 뒤에는 이전 태스크 정의를 다시 배포해야 하므로
  즉시 전환할 수 없습니다.

### 장점

- Green을 운영 트래픽과 분리한 상태에서 검증할 수 있습니다.
- 트래픽 전환 전후로 Blue를 유지하므로 롤백 경로가 단순합니다.
- 배포 단계와 상태를 분리해 관찰할 수 있어 장애 원인 파악에 도움이 됩니다.
- 테스트 리스너나 lifecycle hook을 이용해 자동화된 검증 단계를 추가할 수 있습니다.

즉, 다운타임이 없고, 안전한 배포가 가능합니다.

### 단점

배포 중 서버가 2개 필요하여 비용이 증가합니다.

### 주의할 점

- Blue와 Green을 동시에 실행하므로 배포 중에는 CPU, 메모리, IP 주소 등 두 환경을 수용할 용량이 필요합니다.
- v1과 v2가 함께 실행되므로 API 계약과 데이터베이스 스키마는 두 버전에서 호환되어야 합니다.
- 세션을 태스크 메모리에만 저장하면 트래픽 전환 후 로그인 상태가 끊길 수 있습니다.
  세션 저장소를 외부화하거나 재시도 방식을 설계해야 합니다.
- Green의 헬스 체크가 너무 단순하면 실제 오류를 통과시킬 수 있고, 너무 엄격하면 정상 배포도 실패할 수 있습니다.
- Blue를 유지하는 bake time과 종료 시점을 정하지 않으면 비용과 롤백 가능 시간의 균형을 맞추기 어렵습니다.

### 언제 선택할까

다음과 같은 상황에서는 Blue/Green을 우선 검토합니다.

- 배포 실패 시 빠르게 이전 버전으로 돌아가야 하는 서비스입니다.
- 실제 요청과 유사한 트래픽으로 새 버전을 검증해야 합니다.
- API 변경이나 대규모 기능 변경처럼 배포 위험이 큰 경우입니다.
- 배포 중 두 서비스 리비전을 실행할 비용과 클러스터 용량이 있습니다.

반대로, 추가 용량 비용을 최소화해야 하거나 로드 밸런서와 Service Connect 없이 단순히 태스크를 교체하는
서비스라면 Rolling Update가 더 적합할 수 있습니다.

### 배포 전 점검 목록

- Blue와 Green을 동시에 실행할 클러스터 용량과 네트워크 IP 주소를 확인합니다.
- 대상 그룹, 리스너, 보안 그룹이 Green 태스크의 헬스 체크를 통과할 수 있게 설정됐는지 확인합니다.
- 테스트 트래픽의 라우팅 규칙과 운영 트래픽의 전환 규칙을 분리해서 확인합니다.
- CloudWatch alarm, lifecycle hook, bake time, 롤백 조건을 배포 전에 합의하고 테스트합니다.
- Blue와 Green이 같은 데이터베이스를 사용해도 안전한지, 특히 스키마와 데이터 마이그레이션 순서를 확인합니다.

<br>

---

## 🚀 Canary

### Canary란

Canary 배포는 새 서비스 리비전(Green)에 일부 운영 트래픽만 먼저 전달하여 검증한 뒤, 지정한 관찰 시간이
지나면 나머지 트래픽을 한 번에 전환하는 배포 전략입니다.

광산의 유독 가스를 감지하던 카나리아처럼, 적은 사용자 요청으로 새 버전의 오류를 먼저 발견한다는 의미에서
Canary라고 부릅니다.

전체 사용자에게 영향을 주기 전에 오류율과 지연 시간을 확인할 수 있어 배포의 영향 범위를 제한하는 데 적합합니다.

즉, **새 버전을 일부 사용자에게만 배포** 입니다.

```text
배포 전:          Blue(v1)  100%    Green(v2)   0%

Canary 단계:      Blue(v1)   90%    Green(v2)  10%
                  └─ canary bake time 동안 관찰 ─┘

전체 전환:        Blue(v1)    0%    Green(v2) 100%
                  └─ bake time 동안 관찰 후 Blue 종료 ─┘
```

위의 10%와 관찰 시간은 예시입니다. 서비스의 요청량과 오류를 발견하는 데 필요한 표본 수를 고려하여
비율과 시간을 정합니다.

```shell
Blue
90%

Green
10%

# 오류가 없으면
70%
30%

↓
50%
50%

↓
100%
```

으로 점진적으로 늘립니다.

### ECS에서의 Canary

현재 ECS에서는 배포 컨트롤러를 `ECS`로 설정하고, 배포 전략을 `CANARY`로 선택해 Canary 배포를
구성할 수 있습니다. ECS가 Green 서비스 리비전을 만들고 설정한 비율에 따라 트래픽을 전환합니다.

### Blue/Green과의 관계

- Canary는 Blue/Green처럼 기존 Blue와 새 Green을 함께 실행합니다.
- 차이는 운영 트래픽을 Green으로 옮기는 방법입니다.
- Blue/Green은 검증 후 운영 트래픽을 한 번에 전환하고, Canary는 일부만 먼저 전환한 뒤 관찰 시간을 둡니다.

| 항목              | Blue/Green                                     | Canary                                       | Linear                               |
| ----------------- | ---------------------------------------------- | -------------------------------------------- | ------------------------------------ |
| 트래픽 전환 횟수  | 한 번에 전환합니다.                            | 두 단계로 전환합니다.                        | 여러 단계로 나누어 전환합니다.       |
| 초기 Green 트래픽 | 0%입니다.                                      | 설정한 소량입니다.                           | 설정한 비율만큼 시작합니다.          |
| 관찰 시점         | 전체 전환 전 테스트와 전환 후 bake time입니다. | 소량 전환 후 canary bake time입니다.         | 각 비율 증가 사이의 관찰 시간입니다. |
| 적합한 경우       | 검증 후 빠르게 전체 전환할 때입니다.           | 소수의 실제 사용자로 위험을 확인할 때입니다. | 장기간 점진 전환이 필요할 때입니다.  |

### 동작 흐름

#### Green 환경 준비

- 새 태스크 정의로 서비스를 업데이트하면 ECS는 Green 서비스 리비전을 만들고 필요한 태스크를 시작합니다.
- 이 동안 Blue는 기존 운영 트래픽을 계속 처리합니다.

Green 태스크는 컨테이너와 로드 밸런서의 헬스 체크를 통과해야 합니다. 헬스 체크가 실패하면 Green에
운영 트래픽을 보내기 전에 배포를 중단하거나 롤백할 수 있습니다.

#### 1단계: Canary 트래픽 전환

Green이 준비되면 ECS는 `canaryPercent`에 설정한 비율만큼 운영 트래픽을 Green으로 전환합니다.
예를 들어 10%로 설정하면 Blue가 90%, Green이 10%의 운영 요청을 처리합니다.

이 단계의 목적은 Green이 실제 트래픽에서 올바르게 동작하는지 확인하는 것입니다.
단순한 헬스 체크만으로 발견하기 어려운 외부 API 호출, 캐시 동작, 인증 흐름, 성능 저하를 관찰할 수 있습니다.

##### Canary bake time

Canary bake time은 일부 트래픽을 Green에 보낸 뒤 나머지 트래픽을 전환하기 전까지 기다리는 시간입니다.
이 시간에는 Blue와 Green의 오류율, 지연 시간, 처리량, 비즈니스 지표를 비교합니다.

트래픽 비율이 너무 낮으면 드문 오류를 발견할 요청 수가 부족할 수 있습니다.
반대로 비율이 너무 높으면 오류가 발생했을 때 영향을 받는 사용자가 많아집니다.
요청량이 적은 서비스는 비율만 낮추기 보다 관찰 시간을 늘려 충분한 요청 표본을 확보해야 합니다.

#### 2단계: 나머지 트래픽 전환

Canary bake time 동안 문제가 없으면 ECS는 나머지 운영 트래픽을
Green으로 한 번에 전환합니다. 이 시점부터 Green이 100%의
운영 요청을 처리하고 Blue는 더 이상 운영 요청을 받지 않습니다.

전체 전환 뒤에도 `bakeTimeInMinutes` 동안 Blue와 Green은 함께 실행됩니다. 이 배포 bake time에는
전체 트래픽에서만 드러나는 문제를 관찰하고, 필요하면 Blue로 롤백할 수 있습니다.

### 트래픽 전환에 필요한 구성

관리형 트래픽 전환을 사용하려면 Application Load Balancer,
Network Load Balancer, Service Connect 중 하나를 구성해야 합니다.
로드 밸런서를 사용하면 Blue와 Green 태스크를 각각 연결할 대상
그룹과 운영 트래픽을 조절할 리스너 규칙이 필요합니다.

| 구성                      | Canary에서의 역할                                            | 주의점                                         |
| ------------------------- | ------------------------------------------------------------ | ---------------------------------------------- |
| Application Load Balancer | 가중치 기반으로 HTTP/HTTPS 요청을 두 대상 그룹에 분배합니다. | 사용자 요청 단위의 지표 비교에 적합합니다.     |
| Network Load Balancer     | TCP/UDP 수준의 연결을 Green으로 전환합니다.                  | 긴 연결은 전환 효과가 늦게 나타날 수 있습니다. |
| Service Connect           | 서비스 간 내부 호출의 트래픽 전환을 관리합니다.              | 호출 서비스의 재시도 정책도 함께 확인합니다.   |

`CODE_DEPLOY` 컨트롤러에서 Canary 또는 Linear를 사용할 때는
Application Load Balancer만 지원됩니다. 따라서 기존 CodeDeploy
기반 서비스라면 로드 밸런서 종류와 deployment configuration의
호환성을 먼저 확인해야 합니다.

### 모니터링과 롤백

Canary 배포는 관찰 지표가 준비되어 있을 때 효과가 있습니다. Green
전체의 평균만 보지 말고 Blue와 Green을 구분해 같은 조건에서
비교할 수 있는 지표를 수집해야 합니다.

| 관찰 항목     | 확인할 내용                                                    | 예시 롤백 기준                        |
| ------------- | -------------------------------------------------------------- | ------------------------------------- |
| 오류율        | Green의 4xx·5xx 비율이 Blue보다 증가하는지 확인합니다.         | Green 5xx 비율이 허용치를 초과합니다. |
| 지연 시간     | Green의 p95·p99 응답 시간이 악화되는지 확인합니다.             | p95 지연 시간이 기준값을 넘습니다.    |
| 처리량        | 동일한 트래픽에서 Green의 성공 처리량이 유지되는지 확인합니다. | 요청 성공 수가 급감합니다.            |
| 비즈니스 지표 | 결제·주문·로그인 같은 핵심 흐름이 정상인지 확인합니다.         | 실패 건수가 기준을 넘습니다.          |

CloudWatch alarm을 배포에 연결하고 자동 롤백을 활성화하면, 경보가
`ALARM` 상태가 되었을 때 배포를 실패로 처리하고 Blue로 되돌릴 수
있습니다. 임계값은 평상시 변동 범위를 고려해 설정해야 하며, 너무
민감한 경보는 정상 배포를 반복해서 중단시킬 수 있습니다.

lifecycle hook에는 Lambda 함수 또는 일시 정지 지점을 설정할 수
있습니다. 예를 들어 Canary 전환 전에는 통합 테스트를 실행하고,
전환 후에는 핵심 API의 synthetic test 결과를 확인할 수 있습니다.
일시 정지 지점은 명시적으로 배포를 재개하지 않으면 시간 초과로
실패할 수 있으므로 운영 절차와 담당자를 정해 둬야 합니다.

### 장점

- 오류가 있더라도 초기에는 일부 사용자에게만 영향을 줍니다.
- 실제 운영 요청으로 새 버전의 기능과 성능을 검증할 수 있습니다.
- CloudWatch alarm과 결합하면 사람이 즉시 확인하지 못해도 자동으로 롤백할 수 있습니다.
- 모든 트래픽을 한 번에 전환하는 Blue/Green보다 위험 범위를 더 작게 시작할 수 있습니다.

즉,

- 가장 안전합니다.
- 문제를 빨리 발견합니다.

### 단점

- 배포 시간이 오래 걸립니다.

### 주의할 점

- Blue와 Green을 동시에 실행하므로 두 서비스 리비전을 수용할 CPU, 메모리, IP 주소 등의 용량이 필요합니다.
- v1과 v2가 동시에 요청을 처리하므로 API 계약과 데이터베이스 스키마는 두 버전에서 호환돼야 합니다.
- 캐시, 세션, 외부 API 호출량처럼 사용자별로 달라지는 상태는 Canary 그룹에서만 오류가 나타날 수 있어
  함께 관찰해야 합니다.
- 트래픽이 매우 적으면 Canary 비율만으로는 충분한 검증 데이터를 얻기 어렵습니다.
  테스트 트래픽과 관찰 시간을 함께 설계합니다.
- 배포 중 오류가 발견됐을 때 누가 롤백을 결정하고 수행하는지 수동 대응 절차도 문서화해야 합니다.

### 언제 선택할까

다음과 같은 상황에서는 Canary를 우선 검토합니다.

- 신규 기능이 실제 사용자 행동과 성능에 미치는 영향을 확인해야 하는 경우입니다.
- 결제, 인증, 추천처럼 오류의 영향이 크지만 전체 전환 전에는 실제 요청 검증이 필요한 서비스입니다.
- 오류율과 지연 시간을 빠르게 감지할 대시보드와 alarm이 있습니다.
- 일부 사용자에게 발생할 수 있는 초기 오류를 감수하는 대신, 전체 장애 가능성을 낮추고자 하는 경우입니다.

반대로 즉시 전체 전환이 필요하거나, 트래픽이 너무 적어 Canary 관찰에 의미 있는 표본을 얻기 어렵다면
Blue/Green All-at-once 또는 Rolling Update가 더 단순하고 적합할 수 있습니다.

### 배포 전 점검 목록

- Blue와 Green을 함께 실행할 클러스터 용량과 네트워크 IP 주소를 확인합니다.
- Canary 비율과 canary bake time이 요청량에 비해 충분한 검증 표본을 만들 수 있는지 확인합니다.
- Green을 대상으로 구분해 볼 수 있는 오류율, 지연 시간, 핵심 비즈니스 지표 대시보드를 준비합니다.
- CloudWatch alarm의 임계값, 자동 롤백 여부, 수동 롤백 담당자를 배포 전에 확인합니다.
- 두 버전이 공존하는 동안 데이터베이스 스키마, 세션, 메시지 포맷이 호환되는지 확인합니다.

<br>

---

## 🚀 Linear

### Linear란

Linear 배포는 새 서비스 리비전(Green)으로 전달하는 운영 트래픽을
같은 비율만큼 여러 번 나누어 늘리는 전략입니다. 각 전환 사이에 관찰 시간을 두므로,
전체 트래픽을 한 번에 전환하지 않고 성능과 오류를 확인할 수 있습니다.

```text
배포 전:          Blue(v1) 100%    Green(v2)   0%

1단계:            Blue(v1)  80%    Green(v2)  20%
                  └─ step bake time 동안 관찰 ─┘

2단계:            Blue(v1)  60%    Green(v2)  40%
                  └─ step bake time 동안 관찰 ─┘

... 반복 ...

마지막 전환:       Blue(v1)   0%    Green(v2) 100%
                  └─ deployment bake time 후 Blue 종료 ─┘
```

위 비율은 예시입니다.

- `stepPercent`가 20이면 매 단계마다 Green 트래픽이 20%p씩 증가합니다.
- 100%에 도달한 뒤에는 단계 사이의 대기 시간 대신, 이전 리비전을 유지하는 별도의 배포 bake time이
  적용됩니다.

### Canary와의 차이

- Canary는 소량의 트래픽을 한 번 검증한 뒤 나머지를 한 번에 전환합니다.
- 반면 Linear는 같은 증가 폭으로 여러 차례 전환합니다.
- 따라서 트래픽이 늘어날수록 발생하는 부하나 오류를 단계별로 관찰할 수 있습니다.

| 항목        | Canary                                    | Linear                                         |
| ----------- | ----------------------------------------- | ---------------------------------------------- |
| 트래픽 전환 | 소량 전환 뒤 나머지를 한 번에 전환합니다. | 같은 비율로 여러 번 늘립니다.                  |
| 관찰 시점   | Canary 단계와 전체 전환 후입니다.         | 매 단계 사이와 전체 전환 후입니다.             |
| 적합한 경우 | 초기 소량 검증이 중요한 경우입니다.       | 증가하는 부하를 계속 확인해야 하는 경우입니다. |
| 배포 시간   | 비교적 짧게 끝날 수 있습니다.             | 단계 수와 대기 시간만큼 길어집니다.            |

### ECS에서의 동작

- 현재 ECS 관리형 Linear 배포는 서비스의 배포 컨트롤러를 `ECS`로 설정하고,
  배포 전략을 `LINEAR`로 선택해 사용합니다.
- ECS는 Blue와 Green 서비스 리비전을 함께 실행한 뒤, 설정한 비율과 간격에 따라
  운영 트래픽을 Green으로 이동합니다.

트래픽 전환을 ECS가 관리하려면,

- Application Load Balancer(ALB),
- Network Load Balancer(NLB),
- Service Connect

중 하나가 필요합니다.

ALB와 NLB를 사용할 때는 Blue와 Green 리비전을 분리할 두 대상 그룹과 운영 트래픽을 제어할
리스너 규칙을 준비합니다.

기존 `CODE_DEPLOY` 컨트롤러에서도 Linear 구성은 가능합니다.
다만 CodeDeploy의 미리 정의된 deployment configuration을 사용하며, Linear 트래픽 전환은
ALB에서만 지원합니다.

새 ECS 관리형 배포와 기존 CodeDeploy 배포의 설정 방식은 다르므로,
서비스의 배포 컨트롤러를 먼저 확인해야 합니다.

### 주요 설정값

| 설정                              | 의미                                               | ECS 관리형 Linear의 범위·기본값               |
| --------------------------------- | -------------------------------------------------- | --------------------------------------------- |
| `linearConfiguration.stepPercent` | 매 단계에서 Green으로 옮길 운영 트래픽 비율입니다. | 3.0~100.0, 0.1 단위, 기본값은 10.0입니다.     |
| `stepBakeTimeInMinutes`           | 다음 트래픽 전환까지 기다리는 시간입니다.          | 0~1,440분, 기본값은 6분입니다.                |
| `bakeTimeInMinutes`               | 100% 전환 뒤 Blue를 유지하며 관찰하는 시간입니다.  | 서비스 배포 설정에서 별도로 지정합니다.       |
| CloudWatch alarm                  | 경보 시 배포 실패 처리와 자동 롤백에 사용합니다.   | 오류율·지연 시간 등 서비스 지표를 연결합니다. |

예를 들어 `stepPercent`를 10, 단계 bake time을 6분으로 설정하면
10%, 20%, 30%처럼 10%p씩 트래픽이 증가합니다.

100% 전환까지 9번의 중간 대기를 거치므로, Green 준비 시간과 마지막 bake time을
제외해도 최소 54분이 걸립니다.

### 배포 흐름

#### Green 리비전 준비

- 새 태스크 정의로 서비스를 업데이트하면 ECS가 Green 태스크를
  시작하고 대체 대상 그룹에 등록합니다.
- 컨테이너와 대상 그룹 헬스 체크가 통과할 때까지 Blue가 기존
  운영 트래픽을 계속 처리합니다.
- 필요하면 테스트 리스너나 lifecycle hook으로 Green을 먼저
  검증합니다.

애플리케이션 프로세스가 시작된 상태와 요청을 받을 준비가 된 상태는 다를 수 있습니다.
헬스 체크는 데이터베이스 연결, 필수 설정 로딩처럼 실제 준비 완료에 필요한 조건을 반영해야 합니다.

#### 단계별 트래픽 전환

Green이 준비되면 ECS가 `stepPercent`만큼 운영 트래픽을 Green에 전달합니다.
`stepBakeTimeInMinutes` 동안 지표가 정상이라면 같은 폭으로 다음 단계로 진행합니다.

`PRE_PRODUCTION_TRAFFIC_SHIFT` 또는 `PRODUCTION_TRAFFIC_SHIFT` lifecycle hook은
각 운영 트래픽 전환 단계마다 호출됩니다.

Lambda hook으로 자동 검증을 실행할 수 있고, pause hook을 사용하면
`ContinueServiceDeployment`를 호출할 때까지 배포를 멈출 수 있습니다.

#### 전체 전환과 완료

Green이 100%의 운영 트래픽을 처리하면 마지막 단계 bake time은
끝납니다. 이후 `bakeTimeInMinutes` 동안에도 Blue와 Green을
함께 유지하며 전체 부하에서의 오류를 관찰합니다.

이 시간이 끝나고 배포가 성공하면 ECS가 Blue 태스크를 종료합니다.
Blue가 종료된 뒤에는 대상 그룹 가중치만 되돌려 즉시 복구할 수 없으므로, 이전 태스크 정의를 다시 배포하는
절차도 준비해야 합니다.

### 모니터링과 롤백

- Linear 배포는 단계마다 자동으로 안전해지는 기능이 아닙니다.
- Blue와 Green을 구분한 지표와, 경보에 따른 중단·롤백 조건을 함께 준비해야 합니다.

| 관찰 항목     | 확인할 내용                                          | 예시 롤백 기준                           |
| ------------- | ---------------------------------------------------- | ---------------------------------------- |
| 오류율        | Green의 5xx 비율이 Blue보다 증가하는지 확인합니다.   | Green 5xx 비율이 허용치를 초과합니다.    |
| 지연 시간     | Green의 p95·p99 응답 시간이 악화되는지 확인합니다.   | p95 지연 시간이 기준값을 넘습니다.       |
| 처리량        | 증가한 트래픽에서도 성공 처리량이 유지되는지 봅니다. | 성공 요청 수가 급감합니다.               |
| 비즈니스 지표 | 결제·주문·로그인 등 핵심 흐름이 정상인지 확인합니다. | 핵심 작업의 실패 건수가 기준을 넘습니다. |

- CloudWatch alarm을 배포에 연결하고 자동 롤백을 활성화하면, 경보가
  `ALARM` 상태일 때 ECS가 배포를 실패로 처리할 수 있습니다.
- 경보는 각 트래픽 단계에서 문제를 발견할 수 있을 만큼 빠르게 평가하되,
  평상시 변동만으로 오작동하지 않게 임계값을 정해야 합니다.

### 설정 예시

다음은 ECS 관리형 Linear 배포의 핵심 설정 예시입니다. ARN과
역할 이름은 실제 환경의 값으로 바꿔야 하며, 예시 값을 그대로
운영 환경에 사용하면 안 됩니다.

```json
{
  "deploymentController": {
    "type": "ECS"
  },
  "deploymentConfiguration": {
    "strategy": "LINEAR",
    "linearConfiguration": {
      "stepPercent": 10.0,
      "stepBakeTimeInMinutes": 6
    },
    "bakeTimeInMinutes": 15,
    "alarms": {
      "alarmNames": ["green-5xx-rate-alarm"],
      "enable": true,
      "rollback": true
    }
  }
}
```

이 설정은 10%p씩 6분 간격으로 Green 트래픽을 늘리고, 100% 전환
후 15분 동안 Blue를 유지합니다. 실제 서비스에서는 요청량과
오류 감지에 필요한 표본 수를 기준으로 비율과 시간을 정합니다.

### 장점

- 전체 사용자에게 영향을 주기 전에 증가하는 실제 트래픽에서
  새 버전을 검증할 수 있습니다.
- 오류가 발견되면 다음 단계로 영향 범위가 커지기 전에 배포를
  중단하거나 롤백할 수 있습니다.
- CloudWatch alarm과 lifecycle hook을 연결해 검증과 대응을
  자동화할 수 있습니다.
- Canary보다 더 세밀하게 트래픽 증가 속도를 조절할 수 있습니다.

### 주의할 점

- Blue와 Green이 함께 실행되므로 두 리비전을 수용할 CPU,
  메모리, IP 주소 여유가 필요합니다.
- v1과 v2가 공존하는 동안 API 계약, 데이터베이스 스키마,
  메시지 포맷은 서로 호환돼야 합니다.
- 긴 연결이나 세션 고정이 있으면 설정한 가중치와 실제 요청
  비율이 바로 일치하지 않을 수 있습니다.
- 단계가 작고 대기 시간이 길수록 안전하게 관찰할 수 있지만,
  배포 시간과 두 환경의 실행 비용이 늘어납니다.
- 트래픽이 적은 서비스는 한 단계의 요청 수가 부족할 수 있으므로
  비율만 줄이지 말고 관찰 시간을 늘리거나 별도 테스트를 합니다.

### 언제 선택할까

다음과 같은 경우에 Linear 배포를 우선 검토합니다.

- 트래픽 증가에 따라 성능 저하가 나타날 수 있는 변경입니다.
- 결제·인증처럼 장애 영향이 크고, 전체 전환 전에 여러 수준의
  실제 요청 검증이 필요한 서비스입니다.
- Blue와 Green을 비교할 대시보드, CloudWatch alarm, 롤백 절차가
  준비되어 있습니다.
- 추가 용량 비용과 긴 배포 시간을 감수하는 대신 전환 속도를
  세밀하게 제어하려는 경우입니다.

반대로 빠른 전환이 필요하면 Blue/Green All-at-once가, 소량만
검증한 뒤 빠르게 전환해도 되면 Canary가 더 단순할 수 있습니다.

### 배포 전 점검 목록

- Blue와 Green을 동시에 실행할 클러스터 용량과 네트워크 IP
  주소가 있는지 확인합니다.
- 단계별 요청 수가 오류를 판단하기에 충분한지 계산하여
  `stepPercent`와 단계 bake time을 정합니다.
- Green을 분리해 확인할 오류율, 지연 시간, 핵심 비즈니스 지표와
  CloudWatch alarm을 준비합니다.
- ALB 또는 NLB 대상 그룹과 리스너 규칙, 또는 Service Connect의
  트래픽 전환 구성이 올바른지 확인합니다.
- 두 버전의 데이터베이스 스키마, 세션, 메시지 포맷이 공존 중에도
  호환되는지와 각 단계에서의 롤백 절차를 검증합니다.

<br>

## 한 줄 암기

> Rolling Update는 **조금씩 교체**한다.

> Blue/Green은 **두 환경을 모두 띄운 뒤 트래픽을 한 번에 전환**한다.

> Canary는 **일부 사용자에게 먼저 배포**한다.

> Linear는 **일정 비율씩 점진적으로 배포**한다.

> Recreate는 **기존 서비스를 종료한 뒤 새 버전을 실행**한다.

<br>

## 참고 자료

- [Amazon ECS blue/green deployments](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/deployment-type-blue-green.html)
- [Required resources for Amazon ECS blue/green deployments](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/blue-green-deployment-implementation.html)
- [Amazon ECS blue/green service deployments workflow](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/blue-green-deployment-how-it-works.html)
- [Amazon ECS service deployment controllers and strategies](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/ecs_service-options.html)
- [Amazon ECS canary deployments](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/canary-deployment.html)
- [Creating an Amazon ECS canary deployment](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/deploy-canary-service.html)
- [Required resources for Amazon ECS canary deployments](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/canary-deployment-implementation.html)
- [Deployment configurations for an Amazon ECS compute platform](https://docs.aws.amazon.com/codedeploy/latest/userguide/deployment-configurations-ecs.html)
- [Amazon ECS linear deployments](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/deployment-type-linear.html)
- [Creating an Amazon ECS linear deployment](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/deploy-linear-service.html)
- [Required resources for Amazon ECS linear deployments](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/linear-deployment-implementation.html)
- [LinearConfiguration API reference](https://docs.aws.amazon.com/AmazonECS/latest/APIReference/API_LinearConfiguration.html)

확인일: 2026-09-29
