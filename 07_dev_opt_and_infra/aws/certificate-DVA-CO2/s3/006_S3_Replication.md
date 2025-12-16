# 🚀 S3 Replication

## 1️⃣ S3 Replication이란?

한 S3 Bucket의 Object를 다른 Bucket으로 자동 복제하는 기능

- Object 단위 복제
- 비동기(asynchronous)
- Versioning 필수

## 2️⃣ Replication의 필수 조건 (시험에 그대로 나옴)

❗ 반드시 만족해야 한다

| 조건                          | 필수 여부 |
| ----------------------------- | --------- |
| Source Bucket Versioning      | ⭕        |
| Destination Bucket Versioning | ⭕        |
| IAM Role 권한                 | ⭕        |
| Bucket Policy / 권한          | ⭕        |

👉 Versioning 안 켜면 Replication 자체 불가

## 3️⃣ Replication 유형

### 🔹 Cross-Region Replication (CRR)

- 서로 다른 AWS Region 간 복제
- ap-northeast-2 → us-east-1

사용 목적

| 목적          | 이유                 |
| ------------- | -------------------- |
| Compliance    | 지역 분리 요구       |
| DR            | 리전 장애 대비       |
| Latency       | 사용자와 가까운 리전 |
| Cross-Account | 계정 분리            |

### 🔹 Same-Region Replication (SRR)

- 같은 Region 내 Bucket 간 복제
- ap-northeast-2 → ap-northeast-2

사용 목적

| 목적            | 이유                 |
| --------------- | -------------------- |
| Log Aggregation | 중앙 로그 수집       |
| Prod → Test     | 실시간 데이터 복제   |
| 계정 분리       | 동일 리전, 다른 계정 |

## 4️⃣ CRR vs SRR 한 눈에 비교

| 구분           | CRR      | SRR  |
| -------------- | -------- | ---- |
| Region         | 다름     | 같음 |
| DR             | ⭕       | ❌   |
| Latency 최적화 | ⭕       | ❌   |
| 로그 수집      | ❌       | ⭕   |
| 시험 빈도      | ⭐⭐⭐⭐ | ⭐⭐ |

## 5️⃣ 복제 동작 방식 (아주 중요 ⚠️)

🔹 비동기 복제 (Asynchronous)

- 업로드 즉시 복제 ❌
- 지연 발생 가능
- 순서 보장 ❌

```shell
PutObject
 → Source Bucket
 → (시간 경과)
 → Destination Bucket
```

👉 실시간 미러링 아님

## 6️⃣ 무엇이 복제되고, 무엇이 안 되는가?

⭕ 복제됨

- 새로 생성된 Object
- 새 버전
- Delete Marker (옵션 설정 시)

❌ 복제 안 됨

- 기존 Object (과거 데이터)
- 이미 존재하던 Version
- Lifecycle에 의한 전환
- 기존 데이터는 Batch Replication 필요

## 7️⃣ IAM 권한 요구사항 (시험 단골)

S3가 대신 복제한다 ❗

- 👉 그래서 IAM Role을 S3에 위임해야 한다

```shell
Source Bucket → Destination Bucket
```

필요 권한:

- s3:GetObjectVersion
- s3:ReplicateObject
- s3:ReplicateDelete

## 8️⃣ Cross-Account Replication 가능

- Source / Destination Bucket
- 서로 다른 AWS Account 가능
- Destination Bucket Policy에서 Source Account 허용 필수

👉 계정 분리 백업/감사 구조
