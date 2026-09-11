# ECS 구성 요소

Amazon ECS를 이해할 때는 다음 흐름을 먼저 잡으면 된다.

```text
Cluster
→ Task Definition
→ Task
→ Service
```

ECS는 컨테이너를 직접 하나씩 실행하는 도구라기보다, 컨테이너 실행 정의를 기반으로 원하는 개수의
컨테이너를 안정적으로 유지하는 관리 서비스이다.

## 1. Cluster

Cluster는 ECS 리소스를 묶는 논리적 공간이다.

Cluster 자체가 서버는 아니다. Task와 Service가 실행되는 관리 단위라고 보면 된다.

```text
ECS Cluster
  ├─ Service A
  ├─ Service B
  └─ One-off Task
```

Fargate를 사용하면 EC2 인스턴스를 직접 관리하지 않아도 Cluster 안에서 Task를 실행할 수 있다.

## 2. Task Definition

Task Definition은 컨테이너 실행 설계도이다.

어떤 이미지를 사용할지, CPU와 메모리를 얼마나 줄지, 포트는 어떻게 열지, 로그는 어디로 보낼지 등을 정의한다.

주요 설정:

```text
container image
CPU / Memory
port mapping
environment variables
secrets
log configuration
task role
task execution role
network mode
```

## 3. Container Definition

Container Definition은 Task Definition 안에 들어가는 개별 컨테이너 설정이다.

하나의 Task Definition에는 하나 이상의 Container Definition이 들어갈 수 있다.

```text
Task Definition
  ├─ app container
  └─ sidecar container
```

## 4. Task

Task는 Task Definition을 기반으로 실제 실행된 단위이다.

```text
Task Definition = 설계도
Task = 설계도로 실행된 실제 컨테이너
```

Task는 일회성 작업으로 실행할 수도 있고, Service에 의해 계속 유지될 수도 있다.

예:

```text
배치 작업 1회 실행
→ Run Task

웹 애플리케이션을 항상 2개 유지
→ Service
```

## 5. Service

Service는 지정한 개수의 Task를 계속 유지하는 구성 요소이다.

예를 들어 desired count가 3이면 ECS Service는 항상 Task 3개가 실행되도록 관리한다.

```text
Desired count = 3

Task 1 running
Task 2 running
Task 3 running
```

Task 하나가 실패하면 Service가 새 Task를 실행해 개수를 맞춘다.

```text
Task 2 stopped
→ ECS Service가 새 Task 실행
→ 다시 Task 3개 유지
```

운영 환경의 웹 API, 백엔드 서버, 워커 프로세스는 보통 Service로 실행한다.

## 6. Launch Type

Launch Type은 ECS Task를 어디에서 실행할지 정하는 방식이다.

| Launch Type | 설명                                            |
| ----------- | ----------------------------------------------- |
| Fargate     | 서버 관리 없이 Task 실행                        |
| EC2         | 사용자가 관리하는 EC2 인스턴스 위에서 Task 실행 |

DVA-C02에서는 Fargate가 자주 등장한다.

Fargate를 사용하면 인스턴스 패치, 용량 관리, Docker daemon 관리 등을 직접 하지 않아도 된다.

## 7. Capacity Provider

Capacity Provider는 ECS가 Task를 실행할 인프라 용량을 어떻게 사용할지 정하는 기능이다.

예를 들어 다음과 같은 전략을 구성할 수 있다.

```text
FARGATE
FARGATE_SPOT
EC2 Auto Scaling Group
```

## 8. ECR

Amazon ECR은 컨테이너 이미지를 저장하는 서비스이다.

ECS는 Task Definition에 적힌 이미지 URI를 보고 ECR에서 이미지를 가져와 Task를 실행한다.

```text
Docker image build
→ ECR push
→ ECS Task Definition에서 image URI 지정
→ ECS Task 실행
```

## 9. Load Balancer

ECS Service는 Application Load Balancer와 자주 함께 사용된다.

ALB는 외부 요청을 받아 정상 상태의 ECS Task로 트래픽을 분산한다.

```text
Client
→ ALB
→ Target Group
→ ECS Task
```

Health Check가 실패하면 ALB는 해당 Task로 트래픽을 보내지 않는다.

## 10. CloudWatch Logs

ECS 컨테이너 로그는 보통 CloudWatch Logs로 보낸다.

Task Definition의 log configuration에서 `awslogs` 드라이버를 설정한다.

장애 분석 시에는 다음을 확인한다.

```text
CloudWatch Logs
ECS Service Events
Stopped Task Reason
```

## 11. IAM Role

ECS에서 자주 헷갈리는 Role은 두 가지이다.

| Role                | 용도                                                       |
| ------------------- | ---------------------------------------------------------- |
| Task Execution Role | ECS가 ECR 이미지 pull, CloudWatch Logs 전송을 할 때 사용   |
| Task Role           | 컨테이너 안의 애플리케이션 코드가 AWS API를 호출할 때 사용 |

예:

```text
ECS가 ECR에서 이미지 가져오기
→ Task Execution Role

Spring Boot 앱이 S3에 파일 업로드
→ Task Role
```

## 12. 한눈에 정리

| 구성 요소            | 핵심 역할                          |
| -------------------- | ---------------------------------- |
| Cluster              | ECS 리소스를 묶는 논리적 공간      |
| Task Definition      | 컨테이너 실행 설계도               |
| Container Definition | Task 안의 개별 컨테이너 설정       |
| Task                 | 실제 실행된 컨테이너 단위          |
| Service              | 원하는 개수의 Task를 유지          |
| Launch Type          | Fargate 또는 EC2 실행 방식 선택    |
| Capacity Provider    | Task 실행 용량 공급 전략           |
| ECR                  | 컨테이너 이미지 저장소             |
| ALB                  | 외부 트래픽을 ECS Task로 분산      |
| CloudWatch Logs      | 컨테이너 로그 수집                 |
| IAM Role             | ECS와 애플리케이션의 AWS 권한 제어 |

## 시험 포인트

```text
컨테이너 실행 설정을 바꾸고 싶다
→ Task Definition 새 revision 생성

항상 N개의 컨테이너를 유지하고 싶다
→ ECS Service desired count

컨테이너 앱이 S3/DynamoDB에 접근해야 한다
→ Task Role

ECS가 ECR 이미지를 pull해야 한다
→ Task Execution Role

외부 HTTP 요청을 여러 Task로 분산하고 싶다
→ ALB + Target Group + ECS Service
```
