# ECS/ECR 학습 계획

## 학습 범위와 목표

- 참고 범위: `AWS Certified Developer Slides v42.pdf` 323~353쪽
- 학습 목표: ECS의 실행 모델, 권한, 네트워크, 스토리지, 확장, 배포, 이벤트 연동, 태스크 배치와 ECR 사용법을 DVA-C02 문제 해결 관점에서 정리합니다.
- 작성 원칙: 각 챕터는 `개념 이해 -> 구성 흐름 -> 비교 기준 -> 시험 함정 -> 확인 문제` 순서로 작성합니다.
- 범위 경계: 354쪽의 AWS Copilot부터는 이번 계획에 포함하지 않습니다.

## 전체 챕터 구성

| Chapter | 파일명                              | 주제                                    |      참고 쪽 |
| ------: | ----------------------------------- | --------------------------------------- | -----------: |
|      01 | `01_container_services_overview.md` | AWS 컨테이너 서비스와 ECS/ECR 전체 흐름 |          323 |
|      02 | `02_ecs_launch_types.md`            | ECS EC2 시작 유형과 Fargate 비교        |      324~325 |
|      03 | `03_ecs_iam_roles.md`               | ECS 권한 모델과 IAM 역할 구분           | 326, 342~343 |
|      04 | `04_ecs_load_balancing.md`          | ALB/NLB 연동과 포트 매핑                | 327, 340~341 |
|      05 | `05_ecs_storage.md`                 | EFS, 바인드 마운트와 임시 스토리지      |     328, 344 |
|      06 | `06_ecs_auto_scaling.md`            | 서비스 오토 스케일링과 인프라 확장      | 329~331, 337 |
|      07 | `07_ecs_rolling_updates.md`         | 롤링 업데이트와 배포 용량 계산          |      332~334 |
|      08 | `08_ecs_event_driven_patterns.md`   | EventBridge와 SQS 기반 실행 패턴        |      335~338 |
|      09 | `09_ecs_task_definitions.md`        | 태스크 정의, 환경 변수와 보안 값 주입   | 339, 342~344 |
|      10 | `10_ecs_task_placement.md`          | EC2 태스크 배치 전략과 제약 조건        |      345~351 |
|      11 | `11_ecr.md`                         | ECR 저장소, 인증, Push/Pull과 보안      |      352~353 |
|      12 | `12_ecs_ecr_practice_exam.md`       | DVA-C02 형식의 종합 문제                |      323~353 |
|      13 | `13_ecs_ecr_hands_on.md`            | ECR과 ECS Fargate 종합 실습             |      323~353 |

## Chapter 01. AWS 컨테이너 서비스와 ECS/ECR 전체 흐름

### 학습 목표

- ECS, EKS, Fargate, ECR의 책임을 구분합니다.
- 이미지 저장과 컨테이너 실행이 분리되는 이유를 이해합니다.
- `Docker image -> ECR -> Task Definition -> ECS Task` 흐름을 설명할 수 있어야 합니다.

### 핵심 내용

- ECS는 AWS 관리형 컨테이너 오케스트레이션 서비스입니다.
- EKS는 Kubernetes API와 생태계를 사용하는 관리형 Kubernetes 서비스입니다.
- Fargate는 ECS 또는 EKS에서 서버를 직접 관리하지 않고 컨테이너를 실행하는 컴퓨팅 방식입니다.
- ECR은 컨테이너 이미지를 저장하고 관리하는 레지스트리입니다.
- 시험에서는 `저장소`, `오케스트레이션`, `컴퓨팅 방식`을 서로 바꾸어 표현한 선택지를 제거합니다.

## Chapter 02. ECS EC2 시작 유형과 Fargate 비교

### 학습 목표

- EC2 시작 유형과 Fargate의 운영 책임 차이를 비교합니다.
- 요구사항에 따라 적절한 실행 방식을 선택할 수 있어야 합니다.

### 핵심 내용

| 비교 항목   | EC2 시작 유형                                          | Fargate                                              |
| ----------- | ------------------------------------------------------ | ---------------------------------------------------- |
| 인프라 관리 | EC2 프로비저닝, 패치, 용량을 직접 관리합니다.          | EC2 인스턴스를 직접 관리하지 않습니다.               |
| ECS Agent   | 각 컨테이너 인스턴스에서 실행합니다.                   | AWS가 실행 환경을 관리합니다.                        |
| 확장 단위   | 태스크와 EC2 용량을 함께 고려합니다.                   | 필요한 태스크 수를 조절합니다.                       |
| 자원 지정   | 인스턴스 전체 자원과 태스크 배치를 고려합니다.         | 태스크별 CPU와 메모리를 지정합니다.                  |
| 적합한 상황 | 호스트 제어와 높은 자원 활용률이 중요할 때 적합합니다. | 운영 부담 최소화와 빠른 배포가 중요할 때 적합합니다. |

### 시험 포인트

- Fargate의 `서버리스`는 서버 관리 책임이 AWS에 있다는 의미입니다.
- EC2 시작 유형에서 태스크만 늘리면 클러스터의 CPU 또는 메모리가 부족할 수 있습니다.

## Chapter 03. ECS 권한 모델과 IAM 역할 구분

### 학습 목표

- EC2 인스턴스 프로파일, Task Execution Role, Task Role을 구분합니다.
- ECR Pull, CloudWatch Logs 전송, 애플리케이션의 AWS API 호출에 필요한 권한 주체를 판단합니다.

### 핵심 내용

| 권한 주체             | 사용 시점                                                    | 대표 권한                                               |
| --------------------- | ------------------------------------------------------------ | ------------------------------------------------------- |
| EC2 인스턴스 프로파일 | EC2 시작 유형의 ECS Agent가 AWS API를 호출할 때 사용합니다.  | ECS 등록, ECR 이미지 가져오기, 로그 전송                |
| Task Execution Role   | ECS/Fargate 실행 계층이 태스크를 시작할 때 사용합니다.       | ECR 이미지 가져오기, CloudWatch Logs 전송, 보안 값 조회 |
| Task Role             | 컨테이너 내부 애플리케이션이 AWS API를 호출할 때 사용합니다. | S3, DynamoDB, SQS 등 업무 권한                          |

### 시험 포인트

- 애플리케이션이 S3에 접근하지 못하면 Task Role을 먼저 확인합니다.
- Fargate가 ECR 이미지를 가져오지 못하거나 로그를 전송하지 못하면 Task Execution Role을 먼저 확인합니다.
- 서로 다른 ECS 서비스에는 최소 권한 원칙에 따라 서로 다른 Task Role을 부여합니다.
- 평문 환경 변수에 비밀 값을 저장하지 않고 Secrets Manager 또는 SSM Parameter Store를 사용합니다.

## Chapter 04. 로드 밸런싱과 포트 매핑

### 학습 목표

- ECS에서 ALB, NLB, CLB를 선택하는 기준을 이해합니다.
- EC2의 동적 호스트 포트 매핑과 Fargate의 태스크별 ENI를 비교합니다.
- 로드 밸런서와 태스크 보안 그룹의 규칙을 설계할 수 있어야 합니다.

### 핵심 내용

- 대부분의 HTTP/HTTPS 워크로드에는 ALB를 사용합니다.
- 높은 처리량, 낮은 지연 시간 또는 AWS PrivateLink 연동에는 NLB를 고려합니다.
- CLB는 고급 기능이 부족하고 Fargate를 지원하지 않으므로 신규 구성에는 권장하지 않습니다.
- EC2 시작 유형에서는 컨테이너 포트만 지정하면 동적 호스트 포트가 할당될 수 있습니다.
- Fargate의 `awsvpc` 모드에서는 각 태스크에 고유한 사설 IP가 부여됩니다.
- 태스크 보안 그룹은 인터넷 전체가 아니라 ALB 보안 그룹에서 오는 애플리케이션 포트만 허용합니다.

## Chapter 05. EFS, 바인드 마운트와 임시 스토리지

### 학습 목표

- 영속 공유 스토리지와 태스크 내부 임시 공유 공간을 구분합니다.
- EC2와 Fargate에서 데이터 수명 주기가 어떻게 달라지는지 이해합니다.

### 핵심 내용

| 요구사항                                              | 적합한 선택                                             |
| ----------------------------------------------------- | ------------------------------------------------------- |
| 여러 AZ의 태스크가 동일한 파일을 공유합니다.          | Amazon EFS를 마운트합니다.                              |
| 같은 태스크의 앱과 사이드카가 임시 파일을 공유합니다. | 바인드 마운트를 사용합니다.                             |
| Fargate에서 임시 저장 공간이 필요합니다.              | 20~200 GiB의 임시 스토리지를 구성합니다.                |
| S3를 일반 파일 시스템처럼 직접 마운트하려고 합니다.   | 해당 범위의 ECS 데이터 볼륨 해법으로 적합하지 않습니다. |

### 시험 포인트

- EFS는 EC2와 Fargate에서 모두 사용할 수 있으며 다중 AZ 공유 데이터에 적합합니다.
- 바인드 마운트는 실행 환경의 수명 주기에 종속되므로 영구 데이터 저장소로 선택하면 안 됩니다.

## Chapter 06. 서비스 오토 스케일링과 인프라 확장

### 학습 목표

- ECS Service Auto Scaling과 EC2 Auto Scaling을 구분합니다.
- Target Tracking, Step Scaling, Scheduled Scaling의 선택 기준을 이해합니다.
- SQS 기반 워커를 메시지 적체량에 따라 확장하는 흐름을 설명할 수 있어야 합니다.

### 핵심 내용

- ECS Service Auto Scaling은 서비스의 `desired count`를 변경합니다.
- 대표 지표는 평균 CPU, 평균 메모리, ALB 대상별 요청 수입니다.
- Target Tracking은 특정 지표를 목표값 근처로 유지합니다.
- Step Scaling은 CloudWatch 경보의 구간에 따라 확장 폭을 다르게 적용합니다.
- Scheduled Scaling은 예측 가능한 시간대의 부하에 적합합니다.
- EC2 시작 유형에서는 태스크 수와 별도로 기반 EC2 용량도 확장해야 합니다.
- Capacity Provider를 Auto Scaling Group과 연결하면 부족한 CPU와 메모리에 맞춰 인스턴스를 확장할 수 있습니다.
- SQS 워커는 큐 적체량 또는 태스크당 메시지 수를 기준으로 확장하는 패턴을 함께 정리합니다.

## Chapter 07. 롤링 업데이트와 배포 용량 계산

### 학습 목표

- `minimumHealthyPercent`와 `maximumPercent`가 배포 순서에 미치는 영향을 계산합니다.
- 무중단 우선 배포와 추가 용량이 제한된 배포의 차이를 설명할 수 있어야 합니다.

### 핵심 내용

- `minimumHealthyPercent`는 배포 중 유지해야 하는 정상 태스크의 하한입니다.
- `maximumPercent`는 배포 중 실행할 수 있는 태스크의 상한입니다.
- Desired Count가 4이고 최소 50%, 최대 100%라면 정상 태스크를 2개까지 줄일 수 있으며 총 4개를 초과할 수 없습니다.
- Desired Count가 4이고 최소 100%, 최대 150%라면 기존 태스크 4개를 유지한 채 새 태스크를 최대 2개 먼저 시작할 수 있습니다.
- 계산 문제에서는 백분율을 Desired Count에 적용한 실제 태스크 수로 변환합니다.

## Chapter 08. EventBridge와 SQS 기반 실행 패턴

### 학습 목표

- 이벤트 기반 단발성 태스크와 장시간 실행되는 ECS 서비스를 구분합니다.
- EventBridge 규칙, 스케줄, SQS, ECS 상태 변경 이벤트의 역할을 이해합니다.

### 핵심 내용

- S3 객체 생성 이벤트를 EventBridge로 전달하여 Fargate 태스크를 실행할 수 있습니다.
- EventBridge 예약 규칙으로 정기 배치 태스크를 실행할 수 있습니다.
- 지속적으로 메시지를 처리하는 워커는 ECS Service가 SQS를 폴링하고 적체량에 따라 확장하도록 구성합니다.
- ECS 태스크 중지 이벤트를 EventBridge에서 감지하고 SNS로 운영자에게 알릴 수 있습니다.
- S3 또는 DynamoDB 접근 권한은 이벤트 규칙이 아니라 실행되는 태스크의 Task Role에 부여합니다.

## Chapter 09. 태스크 정의와 애플리케이션 구성

### 학습 목표

- Task Definition의 주요 필드를 읽고 실행 결과를 예상할 수 있어야 합니다.
- 일반 설정, 민감 정보, 대량 환경 설정의 저장 위치를 구분합니다.

### 핵심 내용

- Task Definition은 이미지, CPU, 메모리, 포트, 네트워크, IAM 역할, 로그 구성을 담는 JSON 메타데이터입니다.
- 하나의 Task Definition에는 최대 10개의 컨테이너 정의를 포함할 수 있습니다.
- 일반 URL과 같은 비민감 값은 환경 변수로 구성할 수 있습니다.
- API 키와 공유 설정은 SSM Parameter Store, 데이터베이스 비밀번호는 Secrets Manager를 고려합니다.
- 다수의 환경 변수를 파일로 관리할 때는 S3의 Environment File을 사용할 수 있습니다.
- 앱 컨테이너와 로그 또는 메트릭 사이드카가 바인드 마운트를 공유하는 패턴을 함께 정리합니다.

## Chapter 10. EC2 태스크 배치 전략과 제약 조건

### 학습 목표

- EC2 시작 유형에서 태스크 배치가 수행되는 순서를 이해합니다.
- Binpack, Random, Spread 전략과 `distinctInstance`, `memberOf` 제약 조건을 구분합니다.

### 배치 순서

1. CPU, 메모리, 포트 요구사항을 만족하는 인스턴스를 찾습니다.
2. Task Placement Constraint를 적용합니다.
3. Task Placement Strategy를 적용합니다.
4. 최종 컨테이너 인스턴스를 선택합니다.

| 구분               | 목적                                                                         |
| ------------------ | ---------------------------------------------------------------------------- |
| Binpack            | 남은 CPU 또는 메모리가 가장 적은 인스턴스부터 채워 EC2 사용 대수를 줄입니다. |
| Random             | 조건을 만족하는 인스턴스에 무작위로 배치합니다.                              |
| Spread             | 가용 영역, 인스턴스 ID 등의 값을 기준으로 고르게 분산합니다.                 |
| `distinctInstance` | 각 태스크를 서로 다른 EC2 인스턴스에 배치합니다.                             |
| `memberOf`         | Cluster Query Language 표현식을 만족하는 인스턴스에만 배치합니다.            |

### 시험 포인트

- 배치 전략과 제약 조건은 EC2 시작 유형에서 사용하며 Fargate에는 적용하지 않습니다.
- 비용 절감은 Binpack, 가용 영역 분산은 Spread, 호스트 중복 방지는 `distinctInstance`로 연결합니다.
- 여러 배치 전략을 조합할 수 있습니다.

## Chapter 11. ECR 저장소, 인증, Push/Pull과 보안

### 학습 목표

- ECR Private/Public Repository의 용도를 이해합니다.
- AWS CLI v2와 Docker CLI를 사용한 인증, Push, Pull 순서를 설명할 수 있어야 합니다.
- 이미지 가져오기 실패 시 IAM 권한 문제를 진단할 수 있어야 합니다.

### 핵심 내용

- ECR은 AWS의 관리형 컨테이너 이미지 레지스트리이며 ECS와 통합됩니다.
- 접근 제어에는 IAM 정책과 Repository Policy를 사용합니다.
- 이미지 취약점 스캔, 태그, 버전 관리와 Lifecycle Policy를 정리합니다.
- 인증 토큰을 `docker login`의 표준 입력으로 전달하고 계정 ID와 리전이 포함된 Registry URI에 로그인합니다.

```bash
aws ecr get-login-password --region <region> \
  | docker login --username AWS --password-stdin \
    <account-id>.dkr.ecr.<region>.amazonaws.com

docker push <account-id>.dkr.ecr.<region>.amazonaws.com/demo:latest
docker pull <account-id>.dkr.ecr.<region>.amazonaws.com/demo:latest
```

### 시험 포인트

- 로그인 대상 리전, Registry URI, 이미지 태그가 일치하는지 확인합니다.
- 이미지 가져오기 실패는 네트워크 경로와 함께 Task Execution Role 또는 EC2 인스턴스 프로파일의 ECR 권한을 확인합니다.
- 오래된 이미지를 자동 정리해야 한다면 Lifecycle Policy를 선택합니다.

## Chapter 12. DVA-C02 형식의 종합 문제

### 문제 1

한 회사가 Amazon ECS의 EC2 시작 유형으로 웹 애플리케이션을 실행하고 있습니다. ECS Service Auto Scaling이 Desired Count를 늘렸지만 새 태스크가 `PENDING` 상태에 머물러 있습니다. 기존 EC2 인스턴스에는 CPU와 메모리가 충분하지 않습니다. 운영 부담을 최소화하면서 기반 인프라를 자동으로 확장하려면 무엇을 구성해야 합니까?

- A. ECS 서비스에 Scheduled Scaling을 구성합니다.
- B. Auto Scaling Group과 연결된 ECS Capacity Provider를 구성합니다.
- C. Task Definition의 `maximumPercent`를 200으로 변경합니다.
- D. ECR Repository에 Lifecycle Policy를 구성합니다.

**정답: B**

**해설:** ECS Service Auto Scaling은 태스크 수를 조절하지만 EC2 용량을 직접 추가하지 않습니다. Capacity Provider를 Auto Scaling Group과 연결하면 부족한 클러스터 용량에 맞춰 EC2 인스턴스를 확장할 수 있습니다.

### 문제 2

Amazon ECS Fargate에서 실행되는 주문 처리 애플리케이션이 DynamoDB 테이블에 항목을 기록해야 합니다. 컨테이너는 정상적으로 시작되지만 DynamoDB 호출에서 `AccessDeniedException`이 발생합니다. 최소 권한 원칙을 따르는 해결 방법은 무엇입니까?

- A. Task Execution Role에 DynamoDB 쓰기 권한을 추가합니다.
- B. Task Role에 대상 테이블의 DynamoDB 쓰기 권한을 추가합니다.
- C. Fargate 태스크의 보안 그룹에 DynamoDB 포트를 허용합니다.
- D. ECR Repository Policy에 DynamoDB 쓰기 권한을 추가합니다.

**정답: B**

**해설:** 컨테이너 내부 애플리케이션의 AWS API 호출에는 Task Role을 사용합니다. Task Execution Role은 이미지 가져오기와 로그 전송처럼 태스크 시작에 필요한 작업에 사용합니다.

### 문제 3

개발자가 인터넷에 공개되는 HTTP 서비스를 ECS Fargate와 ALB로 배포하려고 합니다. 태스크는 ALB를 통해서만 요청을 받아야 합니다. 요구사항을 충족하는 구성을 두 개 선택하십시오.

- A. ALB 보안 그룹의 인바운드에 인터넷의 80 또는 443 포트를 허용합니다.
- B. 태스크 보안 그룹의 애플리케이션 포트를 ALB 보안 그룹 소스로 허용합니다.
- C. 태스크 보안 그룹의 모든 포트를 `0.0.0.0/0`에 허용합니다.
- D. 각 Fargate 태스크에 동일한 사설 IP를 할당합니다.
- E. Classic Load Balancer를 사용하고 동적 호스트 포트를 등록합니다.

**정답: A, B**

**해설:** 외부 요청은 ALB가 받고, 태스크는 ALB 보안 그룹에서 들어오는 애플리케이션 트래픽만 허용합니다. Fargate 태스크에는 각각 고유한 사설 IP가 부여됩니다.

### 문제 4

ECS 서비스의 Desired Count는 4입니다. 배포 중 기존 정상 태스크 4개를 모두 유지하면서 새 버전 태스크를 가능한 한 많이 먼저 시작하려고 합니다. `minimumHealthyPercent`는 100%, `maximumPercent`는 150%입니다. 첫 배포 단계에서 동시에 시작할 수 있는 새 태스크의 최대 개수는 몇 개입니까?

- A. 1개
- B. 2개
- C. 4개
- D. 6개

**정답: B**

**해설:** 최대 실행 가능 태스크는 `4 x 150% = 6`개입니다. 기존 태스크 4개를 유지해야 하므로 새 태스크는 최대 2개를 먼저 시작할 수 있습니다.

### 문제 5

개발자가 프라이빗 ECR Repository의 이미지를 사용하는 Fargate 태스크를 실행했습니다. 태스크가 애플리케이션 시작 전에 이미지 가져오기 권한 오류로 중지됩니다. 네트워크 연결은 정상입니다. 어떤 변경이 문제를 가장 직접적으로 해결합니까?

- A. Task Role에 S3 읽기 권한을 추가합니다.
- B. Task Execution Role에 필요한 ECR 이미지 가져오기 권한을 추가합니다.
- C. ECS Service Auto Scaling의 최소 태스크 수를 늘립니다.
- D. Task Placement Strategy를 Binpack으로 변경합니다.

**정답: B**

**해설:** Fargate 실행 계층이 ECR에서 이미지를 가져올 때 Task Execution Role을 사용합니다. 애플리케이션 업무 권한을 담당하는 Task Role과 혼동하지 않아야 합니다.

## Chapter 13. ECR과 ECS Fargate 종합 실습

### 실습 목표

- 로컬 애플리케이션을 컨테이너 이미지로 만들고 ECR에 저장합니다.
- ECR 이미지를 사용하는 ECS Fargate 서비스를 배포합니다.
- ALB, CloudWatch Logs, IAM 역할과 Service Auto Scaling을 연결합니다.
- 장애를 의도적으로 발생시킨 후 ECS 이벤트와 중지 사유로 원인을 찾습니다.
- 실습 종료 후 비용이 발생하는 리소스를 빠짐없이 제거합니다.

### 사전 준비

- Docker, AWS CLI v2와 적절한 AWS 자격 증명이 필요합니다.
- 하나의 AWS 리전과 고유한 리소스 접두사를 정합니다.
- ALB, NAT Gateway, Fargate, ECR 이미지 저장 비용이 발생할 수 있습니다.
- 작은 이미지와 최소 태스크 수를 사용하고 실습 당일 리소스를 정리합니다.

### 실습 아키텍처

```text
Developer
  -> Docker build
  -> Amazon ECR
  -> ECS Task Definition
  -> ECS Fargate Service
  -> Application Load Balancer
  -> CloudWatch Logs
```

### 실습 단계

1. `/health` 엔드포인트가 있는 간단한 HTTP 애플리케이션을 준비합니다.
2. `Dockerfile`을 작성하고 로컬에서 이미지와 컨테이너 응답을 검증합니다.
3. ECR Private Repository를 생성하고 이미지 스캔과 태그 정책을 확인합니다.
4. ECR에 인증하고 이미지에 Registry URI 태그를 추가한 후 Push합니다.
5. CloudWatch Log Group, Task Execution Role과 최소 권한 Task Role을 준비합니다.
6. CPU, 메모리, 포트, `awsvpc`, `awslogs`를 포함한 Fargate Task Definition을 등록합니다.
7. VPC, 서브넷, ALB, Target Group과 보안 그룹을 구성합니다.
8. ECS Cluster와 Fargate Service를 생성하고 Desired Count를 2로 설정합니다.
9. ALB DNS 이름으로 애플리케이션과 `/health` 응답을 확인합니다.
10. CloudWatch Logs에서 각 태스크의 표준 출력 로그를 확인합니다.
11. 평균 CPU 또는 ALB 대상별 요청 수를 기준으로 Target Tracking을 구성합니다.
12. 새 이미지 태그와 Task Definition Revision을 배포하고 롤링 업데이트를 관찰합니다.
13. 잘못된 이미지 태그 또는 부족한 ECR 권한으로 배포 실패를 재현합니다.
14. ECS Service Event, 태스크 중지 사유와 로그를 사용해 원인을 진단하고 복구합니다.
15. ECS Service, ALB, Target Group, Log Group, ECR 이미지와 Repository 등 생성한 리소스를 역순으로 삭제합니다.

### 필수 검증 항목

- ECR에서 Push한 이미지 태그와 스캔 결과를 확인합니다.
- Fargate 태스크 2개가 서로 다른 사설 IP로 정상 등록되었는지 확인합니다.
- 태스크 보안 그룹이 ALB 보안 그룹의 트래픽만 허용하는지 확인합니다.
- ALB Target Group의 두 대상이 `healthy`인지 확인합니다.
- CloudWatch Logs에서 각 태스크의 로그가 수집되는지 확인합니다.
- 롤링 업데이트 중 설정한 최소/최대 비율이 지켜지는지 확인합니다.
- 실패한 태스크의 중지 사유에서 이미지 또는 IAM 문제를 식별합니다.

### 실습 완료 기준

- ECR Push부터 Fargate 서비스 응답까지 전체 경로를 설명할 수 있어야 합니다.
- Task Role과 Task Execution Role을 서로 바꾸지 않고 구성할 수 있어야 합니다.
- 서비스 태스크 확장과 EC2 기반 용량 확장의 차이를 설명할 수 있어야 합니다.
- 배포 실패를 ECS 이벤트, 중지 사유, 로그의 순서로 진단할 수 있어야 합니다.
- 비용 발생 리소스가 남지 않도록 최종 정리 결과를 확인해야 합니다.
