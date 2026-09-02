# ECS/ECR 학습 정리 계획

## 학습 범위와 목표

- 참고 자료: `AWS Certified Developer Slides v45.pdf` 슬라이드 312~348
- 학습 범위: Docker 기본 개념, AWS 컨테이너 서비스, ECS 실행 유형, IAM, 로드 밸런싱, 스토리지, 오토 스케일링, 롤링 업데이트, 이벤트 기반 실행, Task Definition, Task Placement, ECR
- 학습 목표: DVA-C02 시험에서 ECS/ECR 문제를 읽었을 때 `무엇을 선택해야 하는지`, `어떤 권한/네트워크/배포 설정을 확인해야 하는지` 빠르게 판단할 수 있도록 정리합니다.
- 작성 방향: 각 세부 문서는 `개념 -> 구성 요소 -> 동작 흐름 -> 시험 포인트 -> 실습/예시` 순서로 작성합니다.

## 전체 챕터 구성

| Chapter | 파일명                                   | 주제                                          | 참고 슬라이드 |
| ------: | ---------------------------------------- | --------------------------------------------- | ------------: |
|      01 | `001_container_and_docker_basic.md`      | 컨테이너와 Docker 기본 개념                   |       312~317 |
|      02 | `002_aws_container_services_overview.md` | AWS 컨테이너 서비스 전체 구조                 |           318 |
|      03 | `003_ecs_launch_types.md`                | ECS EC2 Launch Type과 Fargate                 |       319~320 |
|      04 | `004_ecs_iam_roles.md`                   | ECS IAM 역할과 권한 모델                      |      321, 337 |
|      05 | `005_ecs_load_balancing.md`              | ECS 로드 밸런싱과 포트 매핑                   |  322, 335~336 |
|      06 | `006_ecs_storage.md`                     | ECS 데이터 볼륨: EFS와 Bind Mount             |      323, 339 |
|      07 | `007_ecs_auto_scaling.md`                | ECS Service Auto Scaling과 EC2 용량 확장      |       324~326 |
|      08 | `008_ecs_rolling_update.md`              | ECS Rolling Update 배포 설정                  |       327~329 |
|      09 | `009_ecs_event_driven_tasks.md`          | EventBridge, Schedule, SQS 기반 ECS 실행 패턴 |       330~333 |
|      10 | `010_ecs_task_definition.md`             | Task Definition과 컨테이너 실행 설정          |      334, 338 |
|      11 | `011_ecs_task_placement.md`              | EC2 Launch Type의 Task Placement              |       340~346 |
|      12 | `012_ecr_basic_and_cli.md`               | ECR 저장소, 이미지 관리, CLI 사용             |       347~348 |
|      13 | `013_ecs_ecr_exam_summary.md`            | DVA-C02 ECS/ECR 시험 포인트 종합              |       312~348 |

## Chapter 01. 컨테이너와 Docker 기본 개념

Docker가 애플리케이션을 컨테이너 이미지로 패키징하고, 동일한 실행 환경을 여러 OS와 서버에서 재현하는 방식을 정리합니다. Docker 이미지, 컨테이너, Dockerfile, Repository, Push/Pull 흐름을 먼저 잡아야 ECS와 ECR의 역할이 명확해집니다.

### 목차

- 컨테이너가 해결하는 문제
- Docker 이미지와 컨테이너의 차이
- Docker와 가상 머신의 차이
- Dockerfile, Build, Run, Push, Pull 흐름
- Docker Hub와 ECR의 역할 비교
- 시험에서 헷갈리는 표현 정리

## Chapter 02. AWS 컨테이너 서비스 전체 구조

AWS에서 컨테이너를 실행하고 저장하는 대표 서비스를 구분합니다. ECS는 AWS 자체 컨테이너 오케스트레이션, EKS는 Kubernetes 기반 오케스트레이션, Fargate는 서버리스 실행 환경, ECR은 이미지 레지스트리입니다.

### 목차

- ECS, EKS, Fargate, ECR 한 줄 정의
- 오케스트레이션 서비스와 이미지 저장소의 차이
- ECS + Fargate + ECR 기본 배포 흐름
- ECS와 EKS 선택 기준
- DVA-C02에서 자주 나오는 서비스 매칭 문제

## Chapter 03. ECS EC2 Launch Type과 Fargate

ECS에서 컨테이너를 실행하는 두 가지 대표 방식을 비교합니다. EC2 Launch Type은 EC2 인스턴스와 ECS Agent를 직접 관리하고, Fargate는 인프라를 직접 프로비저닝하지 않고 태스크 단위로 CPU와 메모리를 지정합니다.

### 목차

- ECS Cluster, ECS Task, ECS Agent 기본 구조
- EC2 Launch Type 동작 방식
- Fargate Launch Type 동작 방식
- EC2와 Fargate 운영 책임 비교
- 확장 방식 차이: EC2 용량 vs Task 수
- 시험 선택 기준: 서버 관리, 호스트 제어, 운영 부담

## Chapter 04. ECS IAM 역할과 권한 모델

ECS 문제에서 가장 자주 틀리는 부분은 권한 주체를 잘못 고르는 것입니다. EC2 Instance Profile, Task Execution Role, Task Role을 구분하고, ECR Pull, CloudWatch Logs 전송, 애플리케이션의 AWS API 호출 권한을 각각 어디에 부여해야 하는지 정리합니다.

### 목차

- EC2 Instance Profile의 역할
- Task Execution Role의 역할
- Task Role의 역할
- 서비스별 Task Role 분리 원칙
- ECR Pull 권한 문제 진단
- S3, DynamoDB, SQS 접근 권한 문제 진단
- Secrets Manager와 SSM Parameter Store 접근 권한

## Chapter 05. ECS 로드 밸런싱과 포트 매핑

ECS 서비스 앞단에 ALB, NLB, CLB를 연결하는 기준과 EC2/Fargate의 포트 매핑 차이를 정리합니다. 특히 EC2 Launch Type의 Dynamic Host Port Mapping과 Fargate의 Task별 ENI/Private IP 구조는 시험에 자주 나옵니다.

### 목차

- ECS에서 지원하는 Load Balancer 종류
- ALB를 사용하는 일반적인 HTTP/HTTPS 패턴
- NLB를 선택하는 고성능/PrivateLink 패턴
- CLB가 권장되지 않는 이유
- EC2 Launch Type의 Dynamic Host Port Mapping
- Fargate의 Task ENI와 고유 Private IP
- ALB Security Group과 Task Security Group 설계
- Health Check 실패 시 배포 영향

## Chapter 06. ECS 데이터 볼륨: EFS와 Bind Mount

컨테이너가 데이터를 저장하거나 공유해야 할 때 사용하는 ECS 스토리지 옵션을 정리합니다. EFS는 여러 AZ의 태스크가 공유할 수 있는 영속 파일 시스템이고, Bind Mount는 같은 Task Definition 안의 컨테이너들이 임시 데이터를 공유할 때 적합합니다.

### 목차

- ECS에서 데이터 볼륨이 필요한 상황
- EFS를 ECS Task에 마운트하는 구조
- EC2와 Fargate 모두에서 EFS를 사용할 수 있는 이유
- Fargate + EFS 서버리스 스토리지 패턴
- S3를 파일 시스템처럼 마운트할 수 없다는 시험 포인트
- Bind Mount의 목적
- EC2 Task와 Fargate Task의 데이터 수명 주기
- Sidecar 컨테이너와 로그/메트릭 공유 패턴

## Chapter 07. ECS Service Auto Scaling과 EC2 용량 확장

ECS Service Auto Scaling은 서비스의 desired task count를 늘리거나 줄입니다. EC2 Launch Type에서는 태스크 수를 늘리는 것과 별개로 underlying EC2 인스턴스 용량도 확보해야 하므로, EC2 Auto Scaling Group과 Capacity Provider 개념을 함께 정리합니다.

### 목차

- ECS Service Auto Scaling의 대상: desired count
- Application Auto Scaling과 ECS의 관계
- CPU, Memory, ALB Request Count Per Target 지표
- Target Tracking, Step Scaling, Scheduled Scaling 비교
- Fargate Auto Scaling이 단순한 이유
- EC2 Launch Type에서 ASG 확장이 필요한 이유
- ECS Cluster Capacity Provider
- CPU 사용률 기반 확장 예시

## Chapter 08. ECS Rolling Update 배포 설정

ECS 서비스 업데이트 시 기존 태스크를 몇 개까지 중지하고 새 태스크를 몇 개까지 추가할 수 있는지 `minimumHealthyPercent`와 `maximumPercent`로 제어합니다. 계산형 문제에 대비해 desired count 기준으로 실제 태스크 수를 산정하는 연습이 필요합니다.

### 목차

- Rolling Update의 기본 동작
- minimumHealthyPercent 의미
- maximumPercent 의미
- Min 50%, Max 100% 배포 흐름
- Min 100%, Max 150% 배포 흐름
- 무중단 배포와 추가 용량의 관계
- 시험 계산 문제 풀이 방식

## Chapter 09. EventBridge, Schedule, SQS 기반 ECS 실행 패턴

ECS는 항상 장기 실행 서비스만 운영하는 것이 아니라 이벤트나 스케줄에 의해 단발성 Fargate Task를 실행할 수도 있습니다. S3 이벤트, 정기 배치, SQS 워커, stopped task 알림 패턴을 구분합니다.

### 목차

- EventBridge Rule로 ECS Task 실행
- S3 Object 생성 이벤트 처리 패턴
- EventBridge Schedule로 정기 배치 실행
- SQS Queue를 polling하는 ECS Service 패턴
- Queue 적체량 기반 ECS Service Auto Scaling
- Stopped Task 이벤트 감지
- EventBridge + SNS 운영 알림 구성
- 이벤트 규칙 권한과 Task Role 권한 구분

## Chapter 10. Task Definition과 컨테이너 실행 설정

Task Definition은 ECS가 컨테이너를 어떻게 실행할지 정의하는 JSON 메타데이터입니다. 이미지, CPU, 메모리, 포트, 네트워크, IAM, 로그, 환경 변수와 민감 정보 주입 방식을 한 번에 정리합니다.

### 목차

- Task Definition의 역할
- Container Definition과 최대 컨테이너 수
- Image Name과 ECR 이미지 참조
- CPU와 Memory 설정
- Container Port와 Host Port
- Network Mode와 Fargate 관련 고려사항
- IAM Role 지정 위치
- CloudWatch Logs 설정
- Environment Variable, SSM Parameter Store, Secrets Manager
- S3 Environment File을 이용한 bulk 환경 변수 관리

## Chapter 11. EC2 Launch Type의 Task Placement

Task Placement는 EC2 Launch Type에서 ECS가 어떤 EC2 컨테이너 인스턴스에 태스크를 배치할지 결정하는 과정입니다. Fargate에는 적용되지 않는다는 점과 Strategy/Constraint의 차이를 명확히 정리합니다.

### 목차

- Task Placement가 필요한 상황
- Scale-out 시 배치와 Scale-in 시 종료 대상 결정
- Task Placement Process 4단계
- Strategy와 Constraint 차이
- Binpack: 비용 절감 목적
- Random: 무작위 배치
- Spread: AZ 또는 Instance 기준 분산
- Strategy 조합 방식
- distinctInstance Constraint
- memberOf Constraint와 Cluster Query Language
- Fargate에는 Task Placement가 적용되지 않는다는 시험 포인트

## Chapter 12. ECR 저장소, 이미지 관리, CLI 사용

ECR은 Docker 이미지를 저장하고 관리하는 AWS 관리형 컨테이너 레지스트리입니다. ECS와 통합되며 IAM으로 접근을 제어하고, 이미지 스캔, 태그, 버전 관리, lifecycle policy를 지원합니다.

### 목차

- ECR Private Repository와 Public Repository
- ECR과 Amazon S3 백엔드
- ECS에서 ECR 이미지를 Pull하는 흐름
- IAM 기반 접근 제어
- 이미지 태그와 버전 관리
- 이미지 취약점 스캔
- Lifecycle Policy 개념
- AWS CLI v2 로그인 명령
- Docker Push/Pull 명령
- Image Pull 실패 시 IAM 권한 확인

## Chapter 13. DVA-C02 ECS/ECR 시험 포인트 종합

앞선 챕터에서 나온 개념을 시험 문제 풀이 관점으로 다시 압축합니다. 서비스 선택, 권한 주체, 네트워크 연결, 배포 설정, 스케일링 대상, 이벤트 기반 실행, 이미지 Pull 실패 원인을 빠르게 구분하는 것이 목표입니다.

### 목차

- ECS/ECR 핵심 용어 빠른 비교
- EC2 Launch Type vs Fargate 선택 문제
- Task Role vs Task Execution Role vs Instance Profile
- ALB/NLB/CLB 선택 기준
- EFS vs Bind Mount vs S3 관련 함정
- Service Auto Scaling vs EC2 Auto Scaling
- Rolling Update 계산 문제
- EventBridge RunTask vs ECS Service
- Task Placement Strategy vs Constraint
- ECR 인증과 Image Pull 권한 문제
- 자주 나오는 장애 시나리오 정리
- 실습 체크리스트
