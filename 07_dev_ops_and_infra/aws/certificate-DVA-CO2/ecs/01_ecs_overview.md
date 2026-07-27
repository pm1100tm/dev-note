# ✍️ ECS Overview

- 컨테이너란?
- ECS란?
- ECS 전체 흐름
- ECS 주요 구성요소

## 컨테이너란?

애플리케이션 실행에 필요한 코드, 런타밍, 라이브러리 설정을 하나로 묶은 실행 단위이다.
컨테이너의 장점은 환경 차이를 줄이는 것이다.

- local, dev, prd 환경에서 동일하게 실행 가능

## ECS란?

ECS(Elastic Container Service)는 AWS 컨테이너 오케스트레이션 서비스이다.
공식 설명 기준으로는 ECS는 컨테이너를 실행, 중지, 관리하는 확장 가능한 컨테이너 관리 서비스이다.

ECS는 다음과 같은 일을 한다.

```text
컨테이너 실행
컨테이너 개수 유지
컨테이너 재시작
배포 관리
로드밸런서 연결
로그 수집 연동
IAM 권한 연결
Auto Scaling 연동
```

## ECS 전체 흐름

ECS의 큰 흐름은 이렇게 보면 된다.

```text
Dockerfile 작성
→ Docker Image build
→ ECR에 Image push
→ ECS Task Definition 작성
→ ECS Service 생성
→ ECS가 Task 실행
→ ALB, CloudWatch, IAM 등과 연동
```

실제 AWS 구성으로 보면 다음과 같다.

```text
Developer
  ↓
Docker Image
  ↓
Amazon ECR
  ↓
ECS Task Definition
  ↓
ECS Service
  ↓
ECS Task
  ↓
Fargate or EC2
```

## ECS 주요 구성요소

| 구성요소             | 의미                                                  |
| -------------------- | ----------------------------------------------------- |
| Cluster              | ECS 리소스를 묶는 논리적 공간                         |
| Task Definition      | 컨테이너 실행 설계도                                  |
| Task                 | Task Definition으로 실행된 실제 컨테이너 묶음         |
| Service              | Task 개수를 유지하고 배포를 관리하는 단위             |
| Container Definition | Task 안에서 실행할 개별 컨테이너 설정                 |
| Launch Type          | 컨테이너를 Fargate에서 실행할지 EC2에서 실행할지 결정 |

### Cluster

Cluster 는

- ECS에서 Task 와 Servie 가 실행되는 논리적 그룹이다.
- 자체가 서버는 아니며, 실행 환경을 묶는 관리 단위에 가깝다.

```text
ECS Cluster
  ├─ Service A
  │   ├─ Task 1
  │   └─ Task 2
  └─ Service B
      └─ Task 1
```

### Task Definition

- ECS에서 가장 중요한 개념이다.
- 컨테이너를 어떻게 실행할지 정의하는 JSON 설계도이다.
- 포함되는 중요 설정은 아래와 같다.

**공식 문서에서도 Task Definition을 애플리케이션의 blueprint, 즉 설계도라고 설명한다.**

```text
어떤 Docker Image 를 사용할지
CPU/Memory 를 얼마나 줄지
어떤 포트를 열지
환경변수는 무엇인지
로그를 어디로 보낼지
IAM Role 은 무엇을 쓸지
Secret Manager 값을 주입할지
```

### Task

Task 는 Task Definition 기반으로 실제 실행된 단위이다.

```text
Task Definition = 설계도
Task = 설계도로 실행된 실제 컨테이너
```

하나의 Task 안에는 하나 이상의 컨테이너가 들어갈 수 있다

예:

```text
Task
  ├─ app container
  └─ sidecar container
```

### Service

Service 는 Task를 안정적으로 유지하는 역할을 한다.
예를 들어, Service 에 desired count 를 2로 설정하면, ECS는 항상 Task 2개가 실행되도록 유지한다.

만약 Task 하나가 죽으면 다음처럼 ECS Service가 새 Task를 실행한다.

```text
Task 1 stopped
Task 2 running

ECS Service가 새 Task 실행

Task 3 running
Task 2 running
```

### Launch Type

- Fargate
- EC2

ECS 컨테이너를 어디에서 실행할지 선택할 수 있다.

| 방식            | 설명                              |
| --------------- | --------------------------------- |
| Fargate         | 서버 관리 없이 컨테이너 실행      |
| EC2 Launch Type | 사용자가 EC2 인스턴스를 직접 관리 |

Fargate의 핵심은 다음과 같다.

```text
EC2 서버 관리 안 함
Task 단위로 CPU/Memory 지정
awsvpc 네트워크 모드 사용
Task마다 ENI가 붙음
```

### ECR과 ECS 관계

ECR은 Docker Image 저장소이다.

ECS는 ECR에 저장된 이미지를 가져와서 컨테이너로 실행한다

```text
ECR = 이미지 저장소
ECS = 이미지를 실행하는 서비스
```

```text
1. Spring Boot 앱 Docker Image 생성
2. ECR에 push
3. ECS Task Definition에서 image URI 지정
4. ECS가 해당 이미지를 pull
5. Task 실행
```

## 반드시 암기할 것

| 질문               | 답                               |
| ------------------ | -------------------------------- |
| ECS는 무엇인가?    | 컨테이너 실행/관리 서비스        |
| ECR은 무엇인가?    | Docker Image 저장소              |
| Task Definition은? | 컨테이너 실행 설계도             |
| Task는?            | 실제 실행된 컨테이너 단위        |
| Service는?         | Task 개수를 유지하고 배포를 관리 |
| Cluster는?         | ECS 리소스를 묶는 논리적 공간    |
| Fargate는?         | 서버리스 컨테이너 실행 환경      |
