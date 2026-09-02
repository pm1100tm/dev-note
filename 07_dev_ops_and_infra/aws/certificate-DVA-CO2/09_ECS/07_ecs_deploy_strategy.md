# 🚀 ECS Deployment Strategy (배포 전략)

- 서비스를 운영하다 보면, 기존 버전(v1)을 새로운 버전(v2)로 교체해야 한다.
- 이 과정을 **Deployment(배포)** 라고 한다.
- ECS에서는 여러 가지 배포 전략을 사용할 수 있다.

---

# 전체 비교

| 전략           | 다운타임  | 배포 속도 | 롤백      | 특징                      |
| -------------- | --------- | --------- | --------- | ------------------------- |
| Rolling Update | 거의 없음 | 빠름      | 쉬움      | 기본 방식                 |
| Blue/Green     | 없음      | 매우 빠름 | 매우 쉬움 | 운영에서 가장 많이 사용   |
| Canary         | 없음      | 느림      | 매우 쉬움 | 일부 사용자에게 먼저 배포 |
| Linear         | 없음      | 보통      | 쉬움      | 일정 비율씩 점진적 배포   |
| Recreate       | 있음      | 매우 빠름 | 어려움    | 기존 종료 후 새 버전 실행 |

---

## ✏️ 1. Rolling Update

- 가장 기본적인 배포 방식이다.
- 기존 Task를 하나씩 종료하면서 새로운 Task를 하나씩 실행한다.

예를 들어, 현재

```shell
# -----------------------------------------------
# v1
Task
Task
Task

↓ 배포 시작

v1
Task
Task

v2
Task

↓

v1
Task

v2
Task
Task

↓

v2
Task
Task
Task
Task
```

기존 서비스를 유지하면서 조금씩 교체한다.

### Rolling Update 장점

- 다운타임이 거의 없다.
- 추가 서버가 많이 필요하지 않다.
- ECS 기본 배포 방식이다.

### Rolling Update 단점

배포 중에는 v1과 v2가 동시에 서비스된다.

```
50% -> v1
50% -> v2
```

버전이 다르기 때문에, DB Schema 변경이 큰 경우 문제가 생길 수 있다.

### Deployment Configuration

Rolling Update에서는 두 개의 옵션이 매우 중요하다.

```
Minimum Healthy Percent
Maximum Percent
```

#### Minimum Healthy Percent

- 배포 중 최소 몇 %의 Task를 살아있게 유지할 것인가

예를 들어

```
Task 4개
Minimum Healthy = 50%
```

이면, 최소 2개는 항상 살아 있어야 한다.

#### Maximum Percent

- 배포 중, 최대 몇 %까지 Task를 실행할 것인가

예를 들어

```
Task 4개
Maximum = 200%
```

이면, 최대 8개 까지 실행 가능하다.

---

### 예제 1

```
Desired Task = 4
Minimum = 50%
Maximum = 100%
```

- 최대 Task - 4개 까지만 가능하다
- 새 Task를 만들 공간이 없으므로, 먼저 기존 Task를 종료한다.

```
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

---

### 예제 2

```
Desired = 4
Minimum = 100%
Maximum = 150%
```

- 최소 4개 는 항상 살아 있어야 한다.
- 최대 6개 까지 실행 가능하다.

즉, 새로운 Task를 먼저 만든다.

```
Task, Task, Task, Task

↓

Task, Task, Task, Task, Task(v2), Task(v2)

↓

Health Check 통과

↓

기존 Task 종료

Task, Task, Task(v2), Task(v2), Task(v2), Task(v2)
```

다운타임이 거의 없다.

---

## 2. Blue / Green Deployment

AWS에서 가장 추천하는 방식이다. CodeDeploy와 함께 사용한다.

기존 서비스

```
Blue

Task
Task
Task
```

새 버전

```
Green

Task
Task
Task
```

두 개를 동시에 띄운다.

---

Health Check가 끝나면 ALB가 트래픽을

```
Blue
100%
```

↓

```
Green
100%
```

으로 한 번에 변경한다.

---

# Blue/Green 장점

- 다운타임 없음
- 즉시 롤백 가능
- 가장 안전한 배포

### 단점

배포 중 서버가 2배 필요하다. 비용이 증가한다.

---

### Blue/Green 롤백

문제가 발생하면 ALB를

```
Green
↓
Blue
```

로 다시 돌리면 된다. 수 초 안에 복구된다.

---

## 3. Canary Deployment

새 버전을 일부 사용자에게만 배포한다. 예를 들어

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

으로 점진적으로 늘린다.

### 장점

- 가장 안전하다.
- 문제를 빨리 발견한다.

### 단점

- 배포 시간이 오래 걸린다.

---

## 4. Linear Deployment

Canary와 비슷하지만 일정 비율씩 증가한다. 예를 들어

```
20%
↓
40%
↓
60%
↓
80%
↓
100%
```

CodeDeploy에서 많이 사용한다.

---

## 5. Recreate

가장 단순한 방식이다. 기존 서비스를 모두 종료한다.

```
Task
Task
Task
Task

↓

모두 종료

↓

새 버전 시작

↓

Task(v2)
Task(v2)
Task(v2)
Task(v2)
```

다운타임이 발생한다.

---

## 어떤 상황에서 사용할까?

| 전략       | 추천 상황                         |
| ---------- | --------------------------------- |
| Rolling    | 일반적인 웹 서비스                |
| Blue/Green | 금융, 쇼핑몰, 운영 서비스         |
| Canary     | 대규모 서비스(Amazon, Netflix 등) |
| Linear     | 점진적 배포가 필요한 경우         |
| Recreate   | 개발 서버, 내부 시스템            |

---

## AWS에서의 지원

| 서비스           | Rolling | Blue/Green                | Canary    | Linear    |
| ---------------- | ------- | ------------------------- | --------- | --------- |
| ECS 기본         | ✅      | ❌                        | ❌        | ❌        |
| ECS + CodeDeploy | ✅      | ✅                        | 일부 지원 | 일부 지원 |
| Lambda           | ❌      | ❌                        | ✅        | ✅        |
| EKS(Kubernetes)  | ✅      | Argo Rollouts 등으로 구현 | ✅        | ✅        |

---

## 시험 핵심

### ECS 기본 Deployment

→ Rolling Update

---

### Blue/Green

- → ECS + CodeDeploy
- → ALB Target Group 두 개 사용

---

### Rolling Update

- → Deployment Configuration
- - Minimum Healthy Percent
- - Maximum Percent

---

## 한 줄 암기

> Rolling Update는 **조금씩 교체**한다.

> Blue/Green은 **두 환경을 모두 띄운 뒤 트래픽을 한 번에 전환**한다.

> Canary는 **일부 사용자에게 먼저 배포**한다.

> Linear는 **일정 비율씩 점진적으로 배포**한다.

> Recreate는 **기존 서비스를 종료한 뒤 새 버전을 실행**한다.
