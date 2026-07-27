# ECS Cluster, Service, Task

여러분이 카페를 운영한다고 생각해보겠습니다.

## 1. 건물 = Cluster

먼저 카페를 열려면 건물이 있어야 합니다.

```text
건물
 ├── 주방
 ├── 창고
 └── 손님 공간
```

이 건물이 바로 ECS의 Cluster입니다.

Cluster는 컨테이너(Task)가 실행될 공간 입니다. 혼자서는 아무 일도 하지 않습니다.

## 2. 직원 = Task

카페에서 실제 일을 하는 것은 직원입니다.

```text
직원
 ├── 커피 만들기
 ├── 계산하기
 └── 청소하기
```

Task 하나가 Spring Boot, Node.js, Redis 일 수 있습니다.

즉, 실제 프로그램 하나가 실행되는 단위입니다.

## 3. 매니저 = Service

문제는 직원이 아플 수도 있습니다.

```text
직원 한 명이 퇴사
↓
카페 운영 불가능
```

그래서 매니저가 필요합니다.

```text
매니저

직원은 항상 3명 유지!
한 명 퇴사?
↓
새 직원 채용
```

## ECS에서는 이렇게 동작합니다.

Spring Boot 서버를 항상 3개 실행하고 싶다고 해봅시다.

```shell
Spring Boot
1
2
3

그런데 하나가 죽었습니다.
1
2
X

Service가 이를 감지합니다.
"어? 하나 죽었네"
↓
새로운 Task 생성

결국 아래와 같이 다시 복구됩니다.
1
2
3
```

Service의 가장 큰 역할은 **원하는 개수(Task)를 계속 유지하는 것** 입니다.

## Cluster 안에는 무엇이 있을까?

```
Cluster
│
├── Service A
│      ├── Task
│      ├── Task
│      └── Task
│
├── Service B
│      ├── Task
│      └── Task
│
└── Service C
       ├── Task
       ├── Task
       ├── Task
       └── Task
```

- Cluster 하나 안에는 여러 개의 Service가 들어갈 수 있습니다.
- 그리고, 각 Service는 여러 개의 Task를 관리합니다.

### 실제 웹 서비스라면,

예를 들어 쇼핑몰이 있다고 해봅시다.
그 안에는,

```
회원 서비스
주문 서비스
결제 서비스
상품 서비스
```

ECS 에서는

```text
Cluster
├── User Service
│      ├── Task
│      ├── Task
│      └── Task
├── Order Service
│      ├── Task
│      ├── Task
│      └── Task
├── Payment Service
│      ├── Task
│      └── Task
└── Product Service
       ├── Task
       ├── Task
       └── Task
```

### 왜 Service가 필요한가?

만약 Service 없이 Task만 실행했다면,

```text
Task 실행

---
서버 오류
↓
Task 종료

```

그러면 끝입니다. 아무도 다시 실행해 주지 않습니다. 하지만 Service가 있으면,

```text
Task 종료
↓
Service 발견
↓
새 Task 생성
```

이 자동으로 이루어집니다. 그래서 운영 환경에서는 대부분 Service를 사용합니다.

### Cluster에는 서버가 있는 걸까?

여기서 또 하나 헷갈리는 부분입니다. Cluster는 실제 서버 자체가 아닙니다.
Cluster는 컨테이너를 실행하기 위한 논리적인 그룹(묶음) 입니다.
실제 컨테이너는 다음과 같은 환경에서 실행됩니다.

```
Cluster
↓
EC2 여러 대
↓
Docker Container
```

EC2 인스턴스 위에서 컨테이너가 실행됩니다.

#### Fargate 방식

```
Cluster
↓
Fargate
↓
Container
```

Fargate를 사용하면 EC2를 직접 관리할 필요 없이 AWS가 컨테이너 실행 환경을 제공합니다.
즉, Cluster는 EC2나 Fargate 같은 실행 환경을 묶어서 관리하는 논리적인 단위입니다.
