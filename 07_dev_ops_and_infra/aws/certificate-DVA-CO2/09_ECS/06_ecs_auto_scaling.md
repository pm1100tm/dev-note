# ECS Auto Scaling

ECS에서 Auto Scaling은 크게 두 가지로 나뉜다.

1. **ECS Service Auto Scaling**
   - Task(컨테이너)의 개수를 조절
2. **EC2 Auto Scaling**
   - EC2 인스턴스의 개수를 조절

즉,

> **Task를 늘리는 것과 EC2 서버를 늘리는 것은 서로 다른 개념이다.**

---

## 전체 구조

```text
                    ECS Cluster
                        │
        ┌───────────────┴────────────────┐
        │                                │
ECS Service Auto Scaling         EC2 Auto Scaling
(Task 개수 증가/감소)            (EC2 개수 증가/감소)
```

---

## 1. ECS Service Auto Scaling

### 역할

ECS Service Auto Scaling은

> **ECS Task(컨테이너)의 개수를 자동으로 증가 또는 감소**시킨다.

예를 들어,

현재

```text
Order Service

Task1
Task2
```

CPU 사용률이 증가하면

```text
Order Service

Task1
Task2
Task3
Task4
```

처럼 Task 개수가 늘어난다.

반대로 사용량이 감소하면 Task 개수를 줄인다.

---

### 어떤 기준으로 Scaling 하는가?

ECS는 CloudWatch Metric을 사용한다.

대표적인 Metric은 다음과 같다.

| Metric                             | 설명                 |
| ---------------------------------- | -------------------- |
| ECSServiceAverageCPUUtilization    | 평균 CPU 사용률      |
| ECSServiceAverageMemoryUtilization | 평균 메모리 사용률   |
| ALBRequestCountPerTarget           | ALB Target당 요청 수 |

---

## Scaling 방식

### 1. Target Tracking Scaling

가장 많이 사용하는 방식이다.

> 목표 CPU 또는 Memory 사용률을 유지하도록 자동으로 Scaling 한다.

예를 들어

```
목표 CPU 사용률 : 50%

현재 CPU
80%
↓
Task 증가
↓
CPU
52%

```

가 되도록 유지한다. Spring Boot 서비스에서 가장 많이 사용하는 방식이다.

---

### 2. Step Scaling

CloudWatch Alarm과 연동된다.

예를 들어

| CPU      | 동작    |
| -------- | ------- |
| 70% 이상 | Task +2 |
| 90% 이상 | Task +5 |

처럼 단계별로 Scaling한다.

---

### 3. Scheduled Scaling

시간을 기준으로 Scaling한다.

예를 들어, 매일 오전 9시

```
Task = 10
```

밤 11시

```
Task = 2
```

처럼 미리 예약할 수 있다. 쇼핑몰이나 이벤트 서비스에서 많이 사용한다.

---

### ECS Service Auto Scaling이 조절하는 것

```
Task 개수
```

만 조절한다. EC2 서버는 늘리지 않는다.

---

## ECS Service Auto Scaling 예시

```text
            ECS Cluster
        EC2 Instance
 ┌──────────────────────────┐
 │ Task                     │
 │ Task                     │
 │ Task                     │
 └──────────────────────────┘
CPU 증가
↓
Task 추가
┌──────────────────────────┐
│ Task                     │
│ Task                     │
│ Task                     │
│ Task                     │
│ Task                     │
└──────────────────────────┘
```

---

## 2. EC2 Launch Type - Auto Scaling EC2 Instances

ECS Service가 Task를 계속 늘리다 보면 EC2에 공간이 부족해질 수 있다.

예를 들어, EC2 한 대가

```
CPU 4Core
Memory 8GB
```

인데, Task가 너무 많아지면

```
CPU 부족
Memory 부족
```

상태가 된다. 그러면 Task를 더 만들 수 없다.

---

### 해결 방법

EC2를 추가한다.

기존

```text
Cluster

EC2-1
```

↓

Scaling

```text
Cluster
EC2-1
EC2-2
```

↓

Task 배치

```text
EC2-1
Task
Task

EC2-2
Task
Task
```

---

## Auto Scaling Group(ASG)

EC2 Auto Scaling은 Auto Scaling Group(ASG)이 담당한다.

예를 들어

```
CPU > 70%
```

↓

EC2 추가

↓

Cluster 용량 증가

↓

Task 배치 가능

---

## ECS Cluster Capacity Provider

최근에는

**Capacity Provider**를 사용하는 것이 권장된다.

역할은

> ECS Task가 부족한 리소스를 감지하여 EC2 Auto Scaling Group을 자동으로 확장한다.

즉,

```
Task 증가
↓
CPU 부족
↓
Capacity Provider
↓
ASG
↓
EC2 생성
↓
Task 배치
```

---

### Capacity Provider의 장점

기존에는 사람이

```
Task 증가
↓
EC2 부족
↓
ASG 설정
```

을 신경 써야 했다. Capacity Provider를 사용하면 ECS가 자동으로

```
Task 증가
↓
용량 부족 감지
↓
EC2 생성
↓
Task 실행
```

을 수행한다.

---

### Fargate에서는?

Fargate는 EC2가 존재하지 않는다.

즉,

```
Task 증가
↓
AWS가 알아서 서버 준비
↓
Task 실행
```

이 된다. 따라서, EC2 Auto Scaling을 신경 쓸 필요가 없다.

---

# EC2와 Fargate 비교

| 항목                   | EC2 Launch Type | Fargate |
| ---------------------- | --------------- | ------- |
| Task Auto Scaling      | ✅              | ✅      |
| EC2 Auto Scaling 필요  | ✅              | ❌      |
| Capacity Provider 사용 | ✅              | ❌      |
| 서버 관리              | 직접            | AWS     |
| 설정 난이도            | 높음            | 낮음    |

---

## 시험 핵심

## ECS Service Auto Scaling

✔ Task 개수를 조절한다.

사용하는 Metric

- CPU
- Memory
- ALB Request Count

지원 방식

- Target Tracking
- Step Scaling
- Scheduled Scaling

---

## EC2 Auto Scaling

- ✔ EC2 인스턴스 개수를 조절한다.
- Auto Scaling Group(ASG)을 사용한다.
- Capacity Provider를 사용하면
- Task 증가에 맞춰 EC2도 자동으로 추가된다.

---

# 한 줄 암기

> **ECS Service Auto Scaling = Task를 늘린다.**

> **EC2 Auto Scaling = EC2 서버를 늘린다.**

> **Fargate는 EC2 Auto Scaling이 필요 없다. AWS가 서버를 자동으로 준비해준다.**
