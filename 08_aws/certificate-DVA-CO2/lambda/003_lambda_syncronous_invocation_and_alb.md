# 🚀 Lambda Syncronous Invocation and ALB

| 핵심 정의

- 동기 호출 -> Lambda 실행 -> 결과 반환 즉시
- 호출자는 응답 결과(success/error) 를 직접 받음
- 실패 시 재시도 / 백오프는 호출자 책임

## 📌 Synchronous Invocation 특징 요약

| 항목         | 설명                                 |
| ------------ | ------------------------------------ |
| 응답 반환    | Lambda 실행 결과를 즉시 반환         |
| 호출 흐름    | Client → Service → Lambda → Response |
| 에러 처리    | Client Side에서 직접 처리            |
| Retry        | 자동 ❌ (SDK / Client가 직접 구현)   |
| Timeout 영향 | Lambda timeout = 호출자 대기 시간    |

### 👤 User-Invoked Synchronous Services

| 서비스                          | 설명                        |
| ------------------------------- | --------------------------- |
| Elastic Load Balancing (ALB)    | HTTP 요청 → Lambda          |
| Amazon API Gateway              | REST / HTTP API → Lambda    |
| Amazon CloudFront (Lambda@Edge) | CDN 요청 시 Lambda 실행     |
| Amazon S3 Batch Operations      | 대량 S3 작업 중 Lambda 호출 |

- ✔ 공통점
  - → 사용자 요청에 대한 즉각적인 응답 필요

### AWS 서비스가 직접 호출

✔ 특징

- 워크플로우 / 인증 흐름 중 즉시 결과 필요
- 실패 시 전체 프로세스에 영향

| 서비스             | 설명                      |
| ------------------ | ------------------------- |
| AWS Step Functions | State 실행 중 Lambda 호출 |
| Amazon Cognito     | 인증 / 가입 트리거        |

### 🧠 Other Synchronous Services

| 서비스                       | 설명               |
| ---------------------------- | ------------------ |
| Amazon Lex                   | 챗봇 응답 생성     |
| Amazon Alexa                 | 음성 명령 처리     |
| Amazon Kinesis Data Firehose | 실시간 데이터 변환 |

<br>

### ⚠️ 에러 처리 책임 (매우 중요)

- Synchronous Invocation에서는 Lambda 가 재시도 안 해준다

호출자(Client)가 해야 할 것

- Retry 로직
- Exponential Backoff
- Timeout 제어
- Circuit Breaker (실무)

## 📌 시험 포인트

- ❌ Lambda가 자동 재시도한다 → 틀림
- ✅ 호출자가 직접 처리 → 정답

---

## 🔐 ALB + Lambda – Permissions (매우 중요)

핵심 원칙

- ALB는 Lambda를 호출하려면 반드시 권한이 필요
- Lambda 에 Resource-based Policy 를 추가해야 한다.

### ✅ 누가 누구를 호출하나?

- 호출자: Elastic Load Balancing
- 대상: AWS Lambda

### 📌 Lambda Permission 예시 (CLI)

```shell
aws lambda add-permission \
  --function-name my-lambda \
  --statement-id alb-invoke \
  --action "lambda:InvokeFunction" \
  --principal elasticloadbalancing.amazonaws.com \
  --source-arn arn:aws:elasticloadbalancing:ap-northeast-2:12345678****:targetgroup/my-tg/abc123
```

#### 각 옵션 역할

| 옵션       | 역할                     |
| ---------- | ------------------------ |
| principal  | 호출 주체 (ALB)          |
| action     | Lambda 실행 권한         |
| source-arn | 특정 Target Group만 허용 |

---

## 🔥 한 줄 요약

| 항목         | 핵심                        |
| ------------ | --------------------------- |
| ALB → Lambda | HTTP → JSON Event           |
| Lambda → ALB | JSON → HTTP Response        |
| Body         | 항상 문자열                 |
| Binary       | Base64 처리                 |
| Multi-Value  | 배열로 전달                 |
| Permission   | Lambda Resource Policy 필수 |

- ALB는 HTTP 요청을 JSON 이벤트로 변환해 Lambda를 동기 호출하며,
  - 호출 권한은 Lambda Resource Policy로 허용해야 한다
