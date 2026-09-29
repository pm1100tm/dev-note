# 🚀 ECS Deployment Strategy (배포 전략)

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
