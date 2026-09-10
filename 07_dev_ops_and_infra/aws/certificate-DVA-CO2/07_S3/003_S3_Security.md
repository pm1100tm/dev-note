# 🚀 S3 Security

## 1️⃣ S3 보안의 두 축

S3 보안은 User-Based + Resource-Based 두 가지로 나뉜다.

## 2️⃣ User-Based Security (IAM Policies)

IAM 사용자 / 역할이 어떤 S3 API를 호출할 수 있는지 정의

```shell
s3:GetObject
s3:PutObject
s3:DeleteObject
s3:ListBucket
```

특징

- 주체(Principal) 기준
- “이 사용자가 뭘 할 수 있나?”
- 계정 내부 중심

## 3️⃣ Resource-Based Security

### 3-1. Bucket Policy (가장 중요 ⭐)

정의

- S3 Bucket에 직접 붙는 정책
- Bucket 전체에 적용
- Cross-Account 접근 가능
- JSON Policy

```shell
{
  "Effect": "Allow",
  "Principal": {"AWS": "arn:aws:iam::12345678****:role/ExternalRole"},
  "Action": "s3:GetObject",
  "Resource": "arn:aws:s3:::my-bucket/*"
}
```

특징 요약

| 항목          | Bucket Policy   |
| ------------- | --------------- |
| 범위          | Bucket 전체     |
| Cross-Account | ⭕              |
| 조건          | IP, VPC, TLS 등 |

### 3-2. Object ACL (Access Control List)

개별 Object 단위 접근 제어

- 아주 세밀
- Legacy
- 비활성화 가능 (권장)

| 특징        | 설명        |
| ----------- | ----------- |
| Granularity | Object 단위 |
| 사용성      | 낮음        |
| 관리        | 복잡        |

### 3-3. Bucket ACL

- Bucket 레벨 ACL
- 거의 안 씀
- 역시 비활성화 가능

👉 시험/실무 모두 “ACL은 쓰지 마라”가 정답

## 🔥 S3 권한 평가 로직 (시험 핵심)

- IAM Principal은 아래 조건을 만족하면 S3 Object 접근 가능

```shell
# ✅ 허용 조건
(IAM Policy에서 ALLOW)
OR
(Bucket Policy / Object ACL에서 ALLOW)

# ❌ 단, 조건 하나
AND Explicit DENY가 없어야 한다
```

## 6️⃣ 시험에 자주 나오는 함정 문장

- “User has no IAM permission but can access S3”
  - 👉 Bucket Policy에서 ALLOW

- “IAM allows access but still denied”
  - 👉 Bucket Policy / SCP / Explicit DENY

- “Grant access to another AWS account”
  - 👉 Bucket Policy (ACL ❌)

## 7️⃣ S3 Encryption (암호화)

- Object 데이터를 저장 시 암호화

## 8️⃣ S3 Encryption 종류 (시험 단골)

### 🔹 Server-Side Encryption (SSE)

| 방식    | 키 관리      |
| ------- | ------------ |
| SSE-S3  | AWS 관리     |
| SSE-KMS | **KMS 키**   |
| SSE-C   | 고객 제공 키 |

### 🔹 Client-Side Encryption

- 업로드 전에 암호화
- S3는 암호화된 데이터만 저장

### ⭐ 시험 포인트

- “Who manages the encryption keys?”

## 9️⃣ 보안 Best Practice (실무 기준)

```shell
- ACL 비활성화
- IAM + Bucket Policy만 사용
- 최소 권한 원칙
- SSE-KMS 사용
- Public Access Block 활성화
```
