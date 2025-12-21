# 🚀 Lambda Overview

## 1️⃣ Serverless란 무엇인가? (정확한 정의)

| Serverless는 “서버가 없는 것”이 아니라 “서버를 직접 관리하지 않는 것”이다.

- 서버는 존재한다
- 하지만:

  - 서버 생성 ❌
  - 서버 패치 ❌
  - 서버 스케일링 ❌
  - 서버 장애 대응 ❌
  - 서버 프로비저닝 ❌
  - 인프라 관리 ❌
  - 자동 확장 ⭕
  - 사용한 만큼만 비용 지불 ⭕

- 👉 개발자는 코드만 배포
- 👉 Infra 관심 0, 비즈니스 로직에만 집중

| 초기 개념

```shell
Serverless ≈ FaaS (Function as a Service)
```

- 이벤트 발생
- 함수 실행
- 종료

👉 이걸 처음 대중화한 게 AWS Lambda

## 2️⃣ Serverless 현재 개념 (중요)

Serverless = “완전 관리형(Managed) 서비스 전반”

즉,

- 서버가 있어도
- 내가 안 만지면 Serverless
- 서버는 AWS(GCP)가 운영하고, 나는 보지도 만지지도 않음

## 3️⃣ AWS에서의 Serverless 서비스 분류

- 🧠 Compute (실행)

  - AWS Lambda
    - 코드 실행(함수)
    - 이벤트 기반
    - 자동 확장
  - AWS Fargate
    - 컨테이너 실행
    - EC2 관리 ❌
    - 컨테이너 기반 Serverless

- 🗄️ Database (저장)

  - DynamoDB
    - 완전 관리형 NoSQL
    - 스케일 무한
    - 서버 개념 ❌
  - Aurora Serverless
    - RDS 처럼 보이지만
    - 용량/서버 자동 조절

- 🌐 API / 인증

  - API Gateway
    - HTTP 엔드포인트
    - Lambda 트리거
  - AWS Cognito
    - 사용자 인증/인가
    - 로그인 서버 ❌

- 📦 Storage

  - Amazon S3
    - 객체 스토리지
    - 서버 관리 ❌
    - 정적 웹 호스팅 가능

- 📣 Messaging / Streaming

  - SNS / SQS

    - Pub/Sub
    - 큐
    - 서버 관리 ❌

  - Kinesis Data Firehose
    - 실시간 데이터 수집
    - 대상: S3, Redshift, OpenSearch

- 🔁 Orchestration
  - Step Functions
    - Lambda 워크플로우
    - 상태 관리
    - 재시도/분기/병렬 처리

## 4️⃣ Serverless의 장단점

✅ 장점

| 항목      | 설명        |
| --------- | ----------- |
| 운영 부담 | 거의 0      |
| 확장성    | 자동        |
| 비용      | 사용량 기반 |
| 속도      | 빠른 개발   |

❌ 단점

| 항목       | 설명               |
| ---------- | ------------------ |
| Cold Start | Lambda 초기 지연   |
| 제한       | 실행 시간 / 메모리 |
| 벤더 종속  | AWS 의존           |
| 디버깅     | 로컬보다 복잡      |

## 5️⃣ 기존 방식: Virtual Servers (EC2 기준)

- 가상 서버를 직접 생성
- RAM / CPU 스펙 고정
- 항상 실행 중
- 트래픽 증가 → 수동/반자동 스케일링 필요

| 문제      | 설명                                |
| --------- | ----------------------------------- |
| 운영 부담 | 패치, 장애 대응, 스케일링 직접 관리 |
| 비용      | 트래픽이 없어도 인프라 비용 발생    |
| 확장성    | Scale-out 시 수동 개입 필요         |
| 민첩성    | 배포 및 변경 속도가 느림            |

👉 “서버를 운영하는 순간 DevOps 비용 발생”

## 6️⃣ Lambda 방식: Virtual Functions

### 핵심 개념

```text
- 서버 관리 ❌
- 함수 단위 실행 ⭕
- 필요할 때만 실행
- 자동 확장
```

서버가 아니라 “코드 실행”에 집중

### Lambda의 실행 모델

```text
Event 발생
 → Lambda 실행
 → 처리
 → 종료
```

- 지속 실행 ❌
- 요청 기반 ⭕
- 초 단위 과금

## 7️⃣ Lambda의 핵심 제한 (시험에 자주 나옴)

| 항목      | 제한                     |
| --------- | ------------------------ |
| 실행 시간 | 최대 15분                |
| 메모리    | 128MB ~ 10GB             |
| 디스크    | `/tmp` 512MB (확장 가능) |
| 상태      | Stateless                |

👉 장시간 서버 / 배치 작업 ❌

## 8️⃣ 왜 Lambda를 쓰는가? (Benefits)

### 💰 1. 쉬운 과금 모델

Pay per use

- 요청 수
- 실행 시간(ms)

Free Tier

- 1,000,000 requests / 월
- 400,000 GB-seconds / 월

👉 트래픽 없으면 비용 0

### 🔌 2. AWS 서비스와의 완벽한 통합

Lambda는 이벤트 허브다.

| 서비스      | 연계          |
| ----------- | ------------- |
| S3          | 업로드 트리거 |
| API Gateway | HTTP API      |
| DynamoDB    | Stream        |
| SQS         | 메시지 소비   |
| EventBridge | 이벤트 처리   |
| SNS         | 알림 처리     |

### 🧠 3. 리소스 조절이 쉬움 (중요 ⚠️)

Lambda 리소스 특징

- RAM 증가 = CPU + Network 증가
- CPU를 직접 설정 ❌
- RAM으로 간접 제어 ⭕

- RAM 1GB → CPU 1
- RAM 4GB → CPU ↑↑
- RAM 10GB → CPU 최고

👉 성능 튜닝 = 메모리 조절

### 📊 4. 모니터링이 쉬움

- CloudWatch Logs
- CloudWatch Metrics
- X-Ray 연계

👉 로그 서버, 에이전트 설치 ❌

### 5️⃣ Lambda 언어 지원 (시험 단골)

공식 지원 런타임

- Node.js
- Python
- Java
- C# (.NET)
- PowerShell
- Ruby

### 6️⃣ Custom Runtime & Container Image

Custom Runtime API

- Rust, Go 등 가능
- Lambda Runtime API 구현 필수

Lambda Container Image

| 항목        | 내용             |
| ----------- | ---------------- |
| 이미지 크기 | 최대 10GB        |
| 실행 방식   | Lambda           |
| 제약        | Runtime API 필수 |

👉 “아무 Docker 이미지나”는 ❌

### 7️⃣ ❗ ECS / Fargate와의 구분 (시험 핵심)

| 상황               | 선택          |
| ------------------ | ------------- |
| 이벤트 기반 함수   | Lambda        |
| 짧은 실행          | Lambda        |
| 서버 관리 ❌       | Lambda        |
| 임의 Docker 이미지 | ECS / Fargate |
| 장시간 실행        | ECS / EC2     |

---

## 한 줄 요약 (암기)

- Lambda = 서버 없는 함수 실행
- 요청 기반 / 자동 확장
- 비용 = 실행 시간
- RAM ↑ = CPU ↑
- 15분 이하 작업에 최적
