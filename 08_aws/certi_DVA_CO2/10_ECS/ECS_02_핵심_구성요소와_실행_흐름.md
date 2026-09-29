# ECS 핵심 구성 요소와 실행 흐름

> 확인일: 2026-09-29

ECS는 컨테이너 실행 정의를 기반으로 태스크를 배치하고, 원하는 개수를 안정적으로 유지하는 컨테이너
오케스트레이션 서비스입니다.

이 문서는 ECS의 구성 요소뿐 아니라, 이미지가 태스크로 실행되고 서비스로 운영되는 흐름, 네트워크와
IAM 역할까지 함께 다룹니다.

Amazon ECS의 핵심 구성 요소는 아래와 같습니다.

- Cluster
- Task Definition
  - Container Definition
- Task
- Service
- Launch Type 또는 Capacity Provider

ECR, 로드 밸런서, CloudWatch Logs, IAM 과 같은 것은, ECS와 자주 함께 쓰이는 주변 서비스와
설정입니다. ECS의 필수 구성 요소 자체는 아닙니다.

아래는 생성 순서가 아니라 구성 요소 사이의 관계를 보여 줍니다.

```text
Task Definition ── 인스턴스화 ──> Task
       │                              │
       │                              └─ Cluster의 EC2 또는 Fargate에서 실행
       │
       └─ Service가 참조 ──> Desired Count만큼 Task를 유지
```

## 1. Cluster

Cluster는 다음을 묶는 논리적 경계입니다.

- ECS에서 Task와 Service가 실행되는 논리적 그룹입니다.
- 자체가 서버는 아니며, 실행 환경을 묶는 관리 단위에 가깝습니다.

즉, Task와 Service가 실행되는 관리 단위라고 보면 됩니다.

```text
ECS Cluster
  ├─ Service A
  │   ├─ Task 1
  │   └─ Task 2
  └─ Service B
      └─ Task 1
```

> Fargate를 사용하면 EC2 인스턴스를 직접 관리하지 않아도 Cluster
> 안에서 Task를 실행할 수 있습니다.

## 2. Task Definition

- 애플리케이션을 구성하는 하나 이상의 컨테이너와 실행 조건을 적은
  JSON 형식의 설계도입니다.
- 한 번 등록한 revision은 변경할 수 없습니다. 이미지를 바꾸거나
  설정을 수정할 때는 새 revision을 등록하고 Service에 적용합니다.

다음과 같은 값을 정의할 수 있습니다.

- 어떤 Docker Image 를 사용할지
- CPU/Memory 를 얼마나 줄지
- 어떤 포트를 열지
- 환경변수는 무엇인지
- 로그는 어디로 보낼지
- IAM Role 은 무엇을 쓸지
- Secret Manager 값을 주입할지
- network mode, 볼륨, 컨테이너 시작 명령은 무엇인지

AWS는 Task Definition을 애플리케이션의 blueprint, 즉 설계도로 설명합니다.
필요한 CPU와 메모리, 이미지, 네트워크, 로그, IAM 역할은 Launch Type과 워크로드에 맞게 선택해야 합니다.

## 3. Task

Task는 Task Definition을 기반으로 실제 실행된 단위입니다.

```text
Task Definition = 설계도
Task = 설계도로 실행된 실제 컨테이너
```

- Task는 `RunTask`로 한 번 실행하는 배치·관리 작업일 수도 있고,
  Service가 계속 유지하는 장기 실행 작업일 수도 있습니다.
- 하나의 Task 안에는 하나 이상의 컨테이너가 들어갈 수 있습니다.

예:

```text
Task
  ├─ app container
  └─ sidecar container
```

## 4. Service

- Service는 지정한 Task Definition을 바탕으로 태스크를 배포하고, 원하는 개수를 유지하는 구성 요소입니다.
- `REPLICA` 방식에서 Desired Count를 3으로 설정하면, ECS는 Task 3개가 실행되도록 유지합니다.

예를 들어 desired count가 3이면 ECS Service는 항상 Task 3개가 실행되도록 관리합니다.

```shell
Desired count = 3

Task 1 running
Task 2 running
Task 3 running
```

Task 하나가 실패하면 Service가 새 Task를 실행해 개수를 맞춥니다.

```shell
Task 1 stopped
Task 2 running
Task 3 running

ECS Service가 새 Task 실행

Task 4 running
Task 2 running
Task 3 running
```

운영 환경의 웹 API, 백엔드 서버, 워커 프로세스는 보통 Service로 실행합니다.
반대로 데이터 일괄 처리처럼 끝나면 종료되는 작업은 독립 Task로 실행할 수 있습니다.

## 5. Container Definition

Container Definition은 Task Definition 안에 들어가는 개별 컨테이너
설정입니다. 이미지, 포트 매핑, 환경 변수, 로그 드라이버, 헬스 체크,
CPU·메모리 제한 등을 컨테이너마다 설정합니다.

하나의 Task Definition에는 하나 이상의 Container Definition이 들어갈 수 있습니다.

```text
Task Definition
  ├─ app container
  └─ sidecar container
```

## 6. Launch Type

Launch Type은 ECS Task가 어떤 컴퓨팅 환경에서 실행될지를 나타냅니다.

| Launch Type | 설명                                              |
| ----------- | ------------------------------------------------- |
| Fargate     | AWS가 실행 서버를 관리하는 서버리스 방식입니다.   |
| EC2         | 사용자가 관리하는 EC2 인스턴스 위에서 실행합니다. |

DVA-C02에서는 EC2와 Fargate의 운영 책임 차이를 구분하는 문제가 자주 등장합니다.

Fargate는 EC2, 운영체제, ECS Agent를 직접 관리하지 않지만, 태스크 정의, 서브넷, 보안 그룹, IAM,
로그와 태스크 수는 계속 관리해야 합니다.

Fargate는 `awsvpc` 네트워크 모드를 반드시 사용하며, 태스크마다 ENI와 사설 IP가 할당됩니다.

반면 EC2 Linux 태스크는 `awsvpc`, `bridge`, `host`, `none` 등의 네트워크 모드를 선택할 수 있습니다.

- [Fargate Task Definition 차이점](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/fargate-tasks-services.html)
- [EC2 Task 네트워킹 옵션](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/task-networking.html)

## 7. Capacity Provider

- Capacity Provider는 ECS가 태스크를 실행할 컴퓨팅 용량과 확장 전략을 선택하는 기능입니다.
- 새 구성에서는 Launch Type을 직접 지정하기보다 Capacity Provider 전략 사용을 AWS가 권장합니다.

대표적인 Capacity Provider는 다음과 같습니다.

| Capacity Provider     | 용도                                               |
| --------------------- | -------------------------------------------------- |
| `FARGATE`             | Fargate 온디맨드 용량에서 태스크를 실행합니다.     |
| `FARGATE_SPOT`        | 중단될 수 있는 Fargate Spot 용량을 사용합니다.     |
| ASG Capacity Provider | 연결한 EC2 Auto Scaling Group의 용량을 사용합니다. |

`EC2 Auto Scaling Group` 자체가 Capacity Provider는 아닙니다. ASG를
연결하여 만든 **ASG Capacity Provider**가 ECS와 ASG의 확장을 연결합니다.

따라서 EC2 기반 서비스에서 Task만 늘리고 EC2 용량을 늘리지 않으면 태스크가 `PENDING` 상태에
머무를 수 있습니다.

## 8. ECR

- Amazon ECR은 컨테이너 이미지를 저장하는 AWS 레지스트리입니다.
- ECS는 Task Definition의 이미지 URI를 사용해 ECR 또는 다른 호환
  레지스트리에서 이미지를 가져와 컨테이너를 실행할 수 있습니다.

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

## 9. Load Balancer

- ECS Service는 Application Load Balancer(ALB) 또는 Network Load Balancer(NLB)와 함께
  사용할 수 있습니다.
- HTTP 경로 기반 라우팅이 필요한 웹 API에는 보통 ALB를 사용합니다.

ALB는 외부 요청을 받아 정상 상태의 ECS Task로 트래픽을 분산합니다.

```text
Client
→ ALB
→ Target Group
→ ECS Task
```

- ALB Target Group의 헬스 체크가 실패하면 ALB는 해당 태스크로 새 요청을 보내지 않습니다.
- `awsvpc`를 사용하는 Fargate 태스크는 Target Group의 Target Type을 `ip`로 지정해야 합니다.

## 10. 네트워크와 보안 그룹

Fargate 태스크와 `awsvpc` 모드의 EC2 태스크는 태스크별 ENI를 가집니다.
Service 또는 `RunTask`를 실행할 때 서브넷과 보안 그룹을 지정합니다.
이 덕분에 태스크마다 보안 그룹 규칙을 적용할 수 있습니다.

```text
Internet
  → ALB 보안 그룹: 443 허용
  → ECS Task 보안 그룹: ALB 보안 그룹에서 앱 포트만 허용
  → RDS 보안 그룹: ECS Task 보안 그룹에서 DB 포트만 허용
```

프라이빗 서브넷의 Fargate 태스크가 ECR에서 이미지를 가져오려면, NAT Gateway 또는 관련
VPC Endpoint 등 적절한 네트워크 경로가 필요합니다.

인터넷에 공개할 필요가 없는 태스크에 Public IP를 부여하는 방식으로 해결하지 않습니다.

[ECS 네트워크 보안 권장 사항](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/security-network.html)을
기준으로 확인했습니다.

## 11. CloudWatch Logs

ECS 컨테이너 로그는 보통 CloudWatch Logs로 보냅니다.

`awslogs` 로그 드라이버는 대표적인 선택지입니다.

Task Definition의 `logConfiguration`에서 로그 드라이버와 Log Group, Region, Stream Prefix를 지정합니다.
로그를 보내려면 Task Execution Role에 필요한 CloudWatch Logs 권한도 있어야 합니다.

장애 분석 시에는 다음을 확인합니다.

```text
CloudWatch Logs
ECS Service Events
Stopped Task Reason
```

## 12. IAM Role

ECS에서 자주 헷갈리는 역할은 다음과 같습니다.

| Role                  | 용도                                                                          |
| --------------------- | ----------------------------------------------------------------------------- |
| EC2 인스턴스 프로파일 | EC2 Launch Type의 ECS Agent가 ECS API를 호출할 때 사용합니다.                 |
| Task Execution Role   | ECS/Fargate 실행 계층이 ECR 이미지 pull, 로그 전송, 시크릿 조회에 사용합니다. |
| Task Role             | 컨테이너 안의 애플리케이션 코드가 AWS API를 호출할 때 사용합니다.             |

예:

```text
ECS가 ECR에서 이미지 가져오기
→ Task Execution Role

Spring Boot 앱이 S3에 파일 업로드
→ Task Role
```

Task Execution Role과 Task Role을 바꾸어 부여하면 안 됩니다. 예를 들어
주문 API가 DynamoDB에 쓰기 권한이 필요하면, 해당 테이블에 필요한 작업만 허용한 Task Role을 연결합니다.

## 13. 한눈에 정리

| 구성 요소            | 핵심 역할                            |
| -------------------- | ------------------------------------ |
| Cluster              | ECS 리소스를 묶는 논리적 공간        |
| Task Definition      | 컨테이너 실행 설계도                 |
| Container Definition | Task 안의 개별 컨테이너 설정         |
| Task                 | 실제 실행된 컨테이너 단위            |
| Service              | 원하는 개수의 Task를 유지            |
| Launch Type          | 태스크의 기반 컴퓨팅 호환성을 나타냄 |
| Capacity Provider    | 태스크 실행 용량과 확장 전략 선택    |
| ECR                  | 컨테이너 이미지 저장소               |
| ALB                  | 외부 트래픽을 ECS Task로 분산        |
| CloudWatch Logs      | 컨테이너 로그 수집                   |
| IAM Role             | 실행 계층과 앱의 AWS 권한을 분리     |

## 간단 정리

| 질문               | 답                               |
| ------------------ | -------------------------------- |
| ECS는 무엇인가?    | 컨테이너 실행/관리 서비스        |
| ECR은 무엇인가?    | Docker Image 저장소              |
| Task Definition은? | 컨테이너 실행 설계도             |
| Task는?            | 실제 실행된 컨테이너 단위        |
| Service는?         | Task 개수를 유지하고 배포를 관리 |
| Cluster는?         | ECS 리소스를 묶는 논리적 공간    |
| Fargate는?         | 서버리스 컨테이너 실행 환경      |

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
