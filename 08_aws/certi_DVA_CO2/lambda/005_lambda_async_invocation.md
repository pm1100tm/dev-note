# 🚀 Lambda Async Invocation

| 핵심 정의

- 호출 서비스가 Lambda 실행 결과를 기다리지 않음
- 이벤트는 Event Queue 에 적재
- Lambda가 백그라운드에서 처리
- 실패 시 Lambda가 자동 재시도

## 📌 Async Invocation 핵심 특징

| 항목        | 설명                         |
| ----------- | ---------------------------- |
| 응답        | 즉시 반환 (실행 결과 미대기) |
| 이벤트 저장 | 내부 Event Queue             |
| Retry       | Lambda가 자동 처리           |
| Retry 횟수  | 총 3회                       |
| Retry 간격  | 1분 → 2분                    |
| 로그        | 재시도 시 중복 로그 발생     |
| 실패 처리   | DLQ(SNS / SQS) 가능          |

## 🔄 Retry 메커니즘 (시험 단골)

재시도 규칙

```shell
1차 실행 → 실패
↓ (1분 대기)

2차 실행 → 실패
↓ (2분 대기)

3차 실행 → 실패 → DLQ (옵션)
```

- ✔ 총 3번 시도
- ✔ Retry는 Lambda 책임 (Client 아님)

### ⚠️ 반드시 멱등성(Idempotency)을 보장해야 함

- 같은 이벤트가 여러 번 처리될 수 있다
- 왜?
  - Retry 발생
  - Event Queue 특성
  - Exactly-once ❌ / At-least-once ✅

#### 🧠 멱등성 실무 예시 (Python)

```python
def handler(event, context):
    event_id = event["id"]

    # 이미 처리한 이벤트인지 확인
    if is_already_processed(event_id):
        return "OK"

    process(event)
    mark_as_processed(event_id)

    return "OK"
```

각 코드 역할

- event_id: 이벤트 고유 식별자
- is_already_processed: DB / Redis 체크
- mark_as_processed: 중복 처리 방지

✔ 시험 포인트

- Async Lambda는 중복 실행 가능 → 멱등 처리 필수

### ☠️ DLQ (Dead-Letter Queue)

DLQ 개념

- 모든 Retry 실패 후 이벤트 저장
- 디버깅 / 재처리 목적

지원 대상

- Amazon Simple Queue Service (SQS)
- Amazon Simple Notification Service (SNS)

필수 조건

- Lambda Execution Role 에 SNS/SQS Publish 권한

## Lambda Asynchronous Invocations – Services

### ✅ 대표 Async 서비스

| 서비스                                   | 설명                      |
| ---------------------------------------- | ------------------------- |
| Amazon Simple Storage Service (S3)       | Object 생성 / 삭제        |
| Amazon Simple Notification Service (SNS) | Pub/Sub 이벤트            |
| Amazon EventBridge                       | 이벤트 라우팅             |
| AWS CodeCommit                           | Push / Branch / Tag       |
| AWS CodePipeline                         | 파이프라인 중 Lambda 실행 |

### 🔁 Other Async Services

| 서비스                      | 사용 사례       |
| --------------------------- | --------------- |
| Amazon CloudWatch Logs      | 로그 처리       |
| Amazon Simple Email Service | 메일 이벤트     |
| AWS CloudFormation          | Stack 이벤트    |
| AWS Config                  | 리소스 변경     |
| AWS IoT                     | 디바이스 이벤트 |
| AWS IoT Events              | IoT 상태 처리   |

## 🚀 Async Invocation의 장점 (Speed)

왜 빠른가?

- 호출 서비스는 Lambda 완료 대기 ❌
- 이벤트만 전달하고 즉시 반환
- 대량 이벤트 처리에 적합

📌 대표 사례

- S3 대량 업로드 후 이미지 리사이징
- 로그 비동기 분석
- 이벤트 기반 데이터 파이프라인

## 🧠 시험 대비 한 방 정리

| 질문                       | 정답            |
| -------------------------- | --------------- |
| Async Lambda Retry는 누가? | Lambda          |
| Retry 횟수                 | 3회             |
| Retry 간격                 | 1분 → 2분       |
| 중복 실행 가능?            | 가능            |
| 대응 방법                  | 멱등성          |
| 실패 이벤트 저장           | DLQ (SNS / SQS) |

---

## 🔥 한 줄 요약 (시험용)

- Asynchronous Lambda 는 Event Queue 기반으로 실행되며,
  자동 재시도와 DLQ를 지원하므로 멱등 처리가 필수다.

---

## 멱등성이란?

☕️ 한 문장 정의

- 같은 요청을 여러 번 실행해도 “결과가 한 번 실행한 것과 동일”하도록 보장하는 성질
- f(x) = f(f(x)) = f(f(f(x)))

👉 시스템 관점 정의

| 항목      | 의미             |
| --------- | ---------------- |
| 입력      | 동일한 이벤트    |
| 실행 횟수 | 1번이든 3번이든  |
| 최종 결과 | 항상 동일해야 함 |
| 부작용    | 중복 발생 ❌     |

## 왜 Lambda Asynchronous 에서 멱등성이 필수인가?

Lambda 비동기 호출의 현실

- At-least-once 실행 보장
- 실패 시 자동 재시도 (최대 3회)
- 네트워크 오류, 타임아웃, 내부 에러 발생 가능

👉 즉,

- 같은 이벤트가 여러 번 실행될 수 있다

## 멱등성이 없으면 생기는 문제

| 상황                    | 결과            |
| ----------------------- | --------------- |
| S3 업로드 이벤트 재시도 | 이미지 3번 처리 |
| 결제 이벤트 중복        | 돈 3번 빠짐 ❌  |
| 이메일 발송             | 동일 메일 3통   |
| DB INSERT               | 중복 데이터     |

### 🧑‍💻 멱등성이 보장되지 않는 코드

```python
def handler(event, context):
    user_id = event["user_id"]

    # ❌ 실행될 때마다 포인트 증가
    increase_point(user_id, 100)

    return "OK"
```

왜 문제인가?

- 같은 이벤트가 3번 실행되면
- 포인트가 300 증가

### 🧑‍💻 멱등성이 보장되는 코드

```python
def handler(event, context):
    event_id = event["id"]  # 이벤트 고유 ID

    # 1. 이미 처리했는지 확인
    if is_already_processed(event_id):
        return "OK"

    # 2. 실제 처리
    process_business_logic(event)

    # 3. 처리 완료 기록
    mark_as_processed(event_id)

    return "OK"
```

### ⭐️ 멱등성 구현 방법 패턴 정리

| 방법                       | 설명                             |
| -------------------------- | -------------------------------- |
| DB Unique Key              | `event_id`를 PK / UNIQUE로 설정  |
| Redis SETNX                | 최초 1회만 처리                  |
| DynamoDB Conditional Write | `attribute_not_exists` 조건 사용 |
| S3 Object 존재 체크        | 이미 있으면 skip                 |

DynamoDB 예시 (시험에도 자주 나옴)

```python
table.put_item(
    Item={"event_id": event_id},
    ConditionExpression="attribute_not_exists(event_id)"
)
```

- 최초 1회만 성공
- 재시도 시 조건 실패 → 중복 방지

### 멱등성과 정확히 구분해야 할 개념

| 개념          | 의미                                   |
| ------------- | -------------------------------------- |
| 멱등성        | 여러 번 실행해도 결과 동일             |
| Exactly-once  | 정확히 1번 실행 ❌ (Lambda Async 불가) |
| At-least-once | 1번 이상 실행 ✅                       |
| Retry         | 실패 시 재실행                         |

👉 Lambda Async = At-least-once → 멱등성 필요
