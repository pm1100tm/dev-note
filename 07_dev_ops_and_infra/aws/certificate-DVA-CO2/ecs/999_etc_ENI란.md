# ✍️ ENI(Elastic Network Interface)

ECS를 공부할 때 나오는 ENI는 Elastic Network Interface의 약자입니다.

초보자라면 “EC2나 ECS Task에 꽂는 랜선(Network Card)” 이라고 생각하면 가장 이해하기 쉽습니다.

즉, ENI가 있어야 네트워크 통신을 할 수 있습니다.

쉽게 비유하면,

```text
- EC2 인스턴스 = 컴퓨터
- ENI = 컴퓨터에 꽂혀 있는 랜카드(Network Interface Card)
- Private IP = 랜카드에 부여된 IP 주소

인터넷
   │
   ▼
VPC
   │
   ▼
Subnet
   │
   ▼
ENI (랜카드)
   │
   ▼
EC2 또는 ECS Task
```

## ENI에는 무엇이 들어있을까?

ENI에는 다음과 같은 네트워크 정보가 저장됩니다.

- Private IP 주소
- Security Group
- MAC Address
- Public IP(필요한 경우)

즉, 네트워크 설정을 담고 있는 객체입니다.

---

## ECS에서는 왜 ENI가 중요할까?

특히 Fargate에서는 Task 하나마다 ENI가 하나씩 생성됩니다.

예를 들어,

```text
ECS Service
   ├── Task 1
   │      └── ENI 1
   │            └── Private IP : 10.0.1.10
   ├── Task 2
   │      └── ENI 2
   │            └── Private IP : 10.0.1.11
   └── Task 3
          └── ENI 3
                └── Private IP : 10.0.1.12
```

**Task마다 고유한 IP를 가지며, 각각 독립적으로 Security Group을 적용할 수 있습니다.**

## EC2 모드와 Fargate 모드 차이

### EC2 Launch Type

```
EC2
 └── ENI 하나
      ├── Task A
      ├── Task B
      └── Task C
```

여러 Task가 EC2의 네트워크를 공유하는 경우가 많습니다.

### Fargate Launch Type (awsvpc)

```
Task A ── ENI A ── IP A
Task B ── ENI B ── IP B
Task C ── ENI C ── IP C
```

Task마다 독립적인 네트워크를 가집니다.

## 한 줄 정리

AWS 시험에서 기억할 한 줄

ENI(Elastic Network Interface)는 EC2나 ECS Task가 네트워크와 통신하기 위한 가상의 네트워크 카드이며, IP 주소와 Security Group 등의 네트워크 정보를 가지고 있다.

ECS를 공부하다 보면 awsvpc 네트워크 모드와 함께 ENI가 자주 등장하는데, 이 둘은 거의 항상 함께
이해하면 됩니다. 특히 Fargate에서는 Task마다 ENI가 생성된다는 점은 AWS 자격증에서도 매우 자주
출제되는 핵심 포인트입니다.
