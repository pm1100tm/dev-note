# AWS SQS Overview

## 1. SQS란?

Amazon SQS(Simple Queue Service)는 AWS에서 제공하는 완전관리형 메시지 큐 서비스이다.
분산 시스템, 마이크로서비스, 서버리스 애플리케이션에서 컴포넌트 간 직접 호출을 줄이고 비동기 방식으로
작업을 전달하기 위해 사용한다.

SQS의 핵심 목적은 다음과 같다.

- 서비스 간 결합도 감소
- 트래픽 스파이크 흡수
- 백그라운드 작업 처리
- 장애 발생 시 메시지 유실 방지
- 생산자와 소비자의 처리 속도 차이 완충

예를 들어 사용자가 이미지를 업로드하면 API 서버는 즉시 SQS에 메시지를 넣고 응답을 반환한다.
이후 별도의 Worker, Lambda, ECS Task 등이 큐에서 메시지를 읽어 이미지 리사이징, 인코딩,
저장 작업을 비동기로 처리할 수 있다.

## 2. 기본 구성 요소

### Producer

메시지를 SQS 큐에 보내는 주체이다.

예시:

- API 서버
- Lambda 함수
- ECS/EKS 애플리케이션
- 배치 잡
- EventBridge, SNS 같은 AWS 서비스

### Queue

메시지를 임시로 보관하는 버퍼 역할을 한다. SQS는 메시지를 여러 서버에 중복 저장하여 내구성과 가용성을 확보한다.

### Message

큐에 저장되는 작업 단위이다. 메시지는 본문, 속성, 시스템 메타데이터로 구성된다.

주요 구성:

- `MessageBody`: 실제 메시지 내용
- `MessageAttributes`: 애플리케이션에서 사용하는 메타데이터
- `MessageId`: SQS가 부여하는 메시지 ID
- `ReceiptHandle`: 메시지 삭제 또는 Visibility Timeout 변경에 사용하는 핸들

### Consumer

큐에서 메시지를 읽고 처리하는 주체이다.

예시:

- Lambda 함수
- EC2 Worker
- ECS/Fargate 서비스
- EKS Pod
- 온프레미스 애플리케이션

## 3. 메시지 처리 흐름

SQS 메시지의 일반적인 처리 흐름은 다음과 같다.

1. Producer가 `SendMessage` API로 큐에 메시지를 보낸다.
2. SQS가 메시지를 큐에 저장한다.
3. Consumer가 `ReceiveMessage` API로 메시지를 가져온다.
4. 메시지는 큐에서 즉시 삭제되지 않고 일정 시간 동안 다른 Consumer에게 보이지 않는 상태가 된다.
5. Consumer가 처리를 완료하면 `DeleteMessage` API로 메시지를 삭제한다.
6. Consumer가 삭제하지 못하면 Visibility Timeout 이후 메시지가 다시 보이게 되어 재처리될 수 있다.

중요한 점은 SQS에서 메시지를 읽는 행위가 메시지를 삭제하지 않는다는 것이다. 메시지 처리가 성공한 뒤 명시적으로 삭제해야 한다.

## 4. Queue 종류

SQS는 크게 Standard Queue와 FIFO Queue 두 종류를 제공한다.

### Standard Queue

Standard Queue는 높은 처리량과 확장성이 필요한 일반적인 비동기 작업에 적합하다.

특징:

- 거의 무제한에 가까운 처리량
- 최소 1회 전달 보장
- 중복 메시지 전달 가능
- 메시지 순서는 최선 노력 방식
- 대규모 병렬 처리에 적합

주의할 점:

- 같은 메시지가 두 번 이상 처리될 수 있으므로 Consumer는 멱등성(idempotency)을 갖도록 설계해야 한다.
- 메시지 순서가 중요하면 애플리케이션에서 별도 정렬 로직을 구현하거나 FIFO Queue를 사용해야 한다.

사용 예:

- 이미지/동영상 후처리
- 이메일 발송 작업
- 로그 처리
- 대규모 배치 작업 분산
- 비동기 주문 후처리

### FIFO Queue

FIFO(First-In-First-Out) Queue는 메시지 순서와 중복 제거가 중요한 경우에 사용한다.

특징:

- 메시지 그룹 내 순서 보장
- 중복 제거 지원
- `MessageGroupId` 필수
- `MessageDeduplicationId` 또는 Content-Based Deduplication 사용 가능
- Standard Queue보다 처리량 제약이 더 명확함

주의할 점:

- 같은 `MessageGroupId`를 가진 메시지는 순차적으로 처리된다.
- 하나의 메시지 그룹에 부하가 몰리면 병렬성이 제한된다.
- 높은 처리량이 필요하면 여러 `MessageGroupId`로 메시지를 분산해야 한다.

사용 예:

- 계좌 입출금 이벤트
- 주문 상태 변경 이벤트
- 재고 수량 변경
- 사용자 명령 순서 보장

## 5. Delivery Semantics

### At-Least-Once Delivery

SQS Standard Queue는 메시지를 최소 1회 전달한다. 네트워크 지연, Consumer 장애, Visibility Timeout 만료 등의 이유로 같은 메시지가 여러 번 전달될 수 있다.

따라서 Consumer는 다음을 고려해야 한다.

- 중복 메시지를 처리해도 결과가 변하지 않도록 멱등성 보장
- 처리 완료 여부를 DB 등에 기록
- 고유 작업 ID 또는 메시지 ID 기반 중복 방지
- 외부 API 호출 시 중복 호출 방어

### Exactly-Once Processing

FIFO Queue는 중복 제거 기능과 순서 보장을 제공한다. 다만 실무적으로는 외부 시스템 연동, Consumer 재시도, 트랜잭션 경계 때문에 애플리케이션 레벨의 멱등성 설계도 함께 고려하는 것이 안전하다.

## 6. Visibility Timeout

Visibility Timeout은 Consumer가 메시지를 받은 뒤 해당 메시지가 다른 Consumer에게 보이지 않는 시간이다.

예를 들어 Visibility Timeout이 30초라면 Consumer가 메시지를 받은 후 30초 동안 다른 Consumer는 그 메시지를 받을 수 없다. 이 시간 안에 Consumer가 `DeleteMessage`를 호출하면 메시지는 큐에서 제거된다. 삭제하지 못하면 메시지는 다시 큐에 나타나 재처리 대상이 된다.

기본값과 범위:

- 기본값: 30초
- 최소값: 0초
- 최대값: 12시간

설계 기준:

- 일반적으로 평균 처리 시간보다 길게 설정한다.
- 너무 짧으면 정상 처리 중인 메시지가 중복 처리될 수 있다.
- 너무 길면 실패한 메시지의 재시도가 늦어진다.
- 처리 시간이 가변적이면 `ChangeMessageVisibility`로 Timeout을 연장하는 방식도 사용할 수 있다.

## 7. Dead-Letter Queue

Dead-Letter Queue(DLQ)는 여러 번 처리에 실패한 메시지를 별도 큐로 격리하는 기능이다.

DLQ를 사용하는 이유:

- 반복 실패 메시지가 메인 큐를 계속 순환하는 문제 방지
- 실패 메시지 분석
- Consumer 버그 또는 데이터 문제 확인
- 장애 복구 후 메시지 재처리

주요 설정:

- `RedrivePolicy`: 실패 메시지를 어떤 DLQ로 보낼지 정의
- `maxReceiveCount`: 메시지를 몇 번까지 재시도한 뒤 DLQ로 보낼지 정의

예를 들어 `maxReceiveCount = 5`이면 Consumer가 같은 메시지를 5번 받아도 삭제하지 못한 경우 해당 메시지가 DLQ로 이동한다.

운영 팁:

- DLQ의 메시지 보존 기간은 원본 큐보다 길게 설정하는 것이 좋다.
- DLQ에 메시지가 쌓이면 CloudWatch Alarm으로 알림을 구성한다.
- DLQ 메시지는 원인을 분석한 뒤 Redrive 기능으로 원본 큐 또는 별도 큐에 재전송할 수 있다.
- FIFO Queue에서 DLQ를 사용할 경우 엄격한 순서 보장이 깨질 수 있으므로 업무 요구사항을 검토해야 한다.

## 8. Short Polling과 Long Polling

### Short Polling

Short Polling은 `ReceiveMessage` 요청 시 SQS 서버 일부만 조회하고 즉시 응답한다.

특징:

- 기본 방식
- 메시지가 없어도 즉시 빈 응답 반환
- 불필요한 API 호출이 늘어날 수 있음
- 비용과 Consumer 부하가 증가할 수 있음

### Long Polling

Long Polling은 메시지가 도착할 때까지 일정 시간 대기한 후 응답한다.

특징:

- `WaitTimeSeconds`를 1초 이상으로 설정하면 활성화
- 최대 대기 시간은 20초
- 빈 응답 수 감소
- API 호출 비용 절감
- 대부분의 일반 Consumer에서 권장

권장 설정:

```text
ReceiveMessageWaitTimeSeconds = 20
```

실무에서는 특별히 낮은 지연 시간이 필요한 경우가 아니라면 Long Polling을 사용하는 것이 일반적이다.

## 9. Delay Queue와 Message Timer

### Delay Queue

Delay Queue는 큐에 들어온 모든 메시지를 일정 시간 동안 Consumer에게 보이지 않게 하는 기능이다.

사용 예:

- 일정 시간 후 작업 실행
- 일시적인 후속 처리 지연
- 외부 시스템 반영 대기

### Message Timer

Message Timer는 개별 메시지마다 지연 시간을 설정하는 기능이다.

특징:

- 메시지 단위로 지연 시간 설정
- 최대 15분까지 지연 가능
- FIFO Queue에서는 개별 Message Timer를 지원하지 않으므로 Queue 수준 Delay를 검토해야 한다.

## 10. Message Retention

Message Retention은 메시지가 큐에 보관될 수 있는 최대 시간이다.

기본값과 범위:

- 기본값: 4일
- 최소값: 60초
- 최대값: 14일

주의할 점:

- Retention 기간을 초과한 메시지는 자동 삭제된다.
- Consumer 장애가 길어질 수 있는 시스템은 Retention 기간을 충분히 길게 잡아야 한다.
- DLQ의 Retention은 원본 큐보다 길게 설정하는 것이 일반적으로 안전하다.

## 11. Message Size와 Batch

주요 제한:

- 메시지 최소 크기: 1바이트
- 메시지 최대 크기: 1MiB
- 메시지 속성: 최대 10개
- Batch 요청: 최대 10개 메시지

큰 메시지를 처리해야 하는 경우:

- 메시지 본문에 큰 데이터를 직접 넣지 않는다.
- S3에 원본 데이터를 저장하고 SQS에는 S3 Object Key 또는 URL만 넣는다.
- Java/Python용 SQS Extended Client Library를 사용할 수 있다.

Batch API:

- `SendMessageBatch`
- `DeleteMessageBatch`
- `ChangeMessageVisibilityBatch`

Batch를 사용하면 API 호출 수를 줄이고 처리량과 비용 효율을 개선할 수 있다.

## 12. Lambda와 SQS 연동

SQS는 Lambda Event Source Mapping과 자주 함께 사용된다.

동작 방식:

1. Lambda 서비스가 SQS를 Polling한다.
2. 메시지가 있으면 Lambda 함수를 호출한다.
3. 함수가 성공하면 메시지를 삭제한다.
4. 함수가 실패하면 메시지는 Visibility Timeout 이후 다시 처리된다.
5. 반복 실패하면 설정에 따라 DLQ로 이동할 수 있다.

주요 설정:

- Batch Size
- Maximum Batching Window
- Visibility Timeout
- Partial Batch Response
- Maximum Concurrency

주의할 점:

- Lambda Timeout은 SQS Visibility Timeout보다 짧아야 한다.
- Batch 처리 중 일부 메시지만 실패할 수 있으면 Partial Batch Response를 사용한다.
- Consumer 처리량이 부족하면 `ApproximateNumberOfMessagesVisible`이 증가한다.

## 13. 보안

### IAM

SQS 접근은 IAM 정책으로 제어한다.

주요 권한:

- `sqs:SendMessage`
- `sqs:ReceiveMessage`
- `sqs:DeleteMessage`
- `sqs:GetQueueAttributes`
- `sqs:ChangeMessageVisibility`
- `sqs:PurgeQueue`

운영 환경에서는 최소 권한 원칙을 적용해야 한다.

예:

- Producer는 `SendMessage`만 허용
- Consumer는 `ReceiveMessage`, `DeleteMessage`, `ChangeMessageVisibility`만 허용
- 관리자 작업인 `PurgeQueue`는 제한적으로 허용

### Encryption

SQS는 서버 측 암호화를 지원한다.

옵션:

- SQS 관리형 SSE
- AWS KMS 고객 관리형 키

KMS를 사용할 경우 고려할 점:

- Producer와 Consumer에 KMS 권한 필요
- KMS API 호출 비용과 제한
- Cross-account 사용 시 Key Policy와 IAM Policy 모두 확인

### Queue Policy

Queue Policy를 사용하면 특정 AWS 계정, 서비스, 리소스에서 큐에 접근하도록 허용할 수 있다.

대표 사례:

- SNS Topic이 SQS Queue로 메시지 전송
- EventBridge Rule이 SQS Queue로 이벤트 전송
- 다른 AWS 계정의 Producer가 메시지 전송

## 14. 모니터링

SQS는 CloudWatch Metrics를 제공한다.

자주 보는 지표:

- `ApproximateNumberOfMessagesVisible`: 처리 대기 중인 메시지 수
- `ApproximateNumberOfMessagesNotVisible`: Consumer가 가져갔지만 아직 삭제되지 않은 메시지 수
- `ApproximateAgeOfOldestMessage`: 가장 오래된 메시지의 대기 시간
- `NumberOfMessagesSent`: 전송된 메시지 수
- `NumberOfMessagesReceived`: 수신된 메시지 수
- `NumberOfMessagesDeleted`: 삭제된 메시지 수
- `NumberOfEmptyReceives`: 빈 응답 수

알람 예:

- 대기 메시지 수가 지속적으로 증가
- 가장 오래된 메시지의 나이가 SLA 초과
- DLQ 메시지 수가 1개 이상
- Empty Receive가 과도하게 증가

## 15. 비용 구조

SQS 비용은 주로 API 요청 수 기반으로 발생한다.

비용 최적화 방법:

- Long Polling 사용
- Batch API 사용
- 불필요한 Polling 감소
- 메시지 크기 최적화
- Consumer Auto Scaling으로 처리 지연 방지
- DLQ에 쌓이는 실패 메시지 원인 제거

## 16. SQS와 다른 메시징 서비스 비교

### SQS vs SNS

SQS는 Queue 기반 메시징이다. Consumer가 메시지를 Pull 방식으로 가져가 처리한다.

SNS는 Pub/Sub 기반 메시징이다. Topic에 메시지를 Publish하면 여러 Subscriber에게 Push 방식으로 전달한다.

비교:

| 구분        | SQS                         | SNS                      |
| ----------- | --------------------------- | ------------------------ |
| 모델        | Queue                       | Pub/Sub                  |
| 소비 방식   | Pull                        | Push                     |
| 메시지 보관 | 가능                        | 기본적으로 보관하지 않음 |
| 주요 목적   | 비동기 작업 큐              | 이벤트 팬아웃, 알림      |
| 수신자 수   | 일반적으로 하나의 처리 흐름 | 여러 Subscriber          |

SNS와 SQS는 함께 많이 사용된다. SNS Topic에 이벤트를 발행하고 여러 SQS Queue가 구독하면 Fan-out 구조를 만들 수 있다.

### SQS vs EventBridge

EventBridge는 이벤트 라우팅과 SaaS/AWS 서비스 이벤트 통합에 강점이 있다. SQS는 작업을 안정적으로 쌓아두고 Consumer가 처리하는 큐에 강점이 있다.

### SQS vs Amazon MQ

Amazon MQ는 ActiveMQ, RabbitMQ 같은 기존 메시지 브로커 호환성이 필요한 경우에 적합하다. SQS는 AWS Native 환경에서 단순하고 확장성 높은 큐가 필요할 때 적합하다.

## 17. 대표 아키텍처 패턴

### 비동기 작업 처리

```text
Client -> API Gateway -> Lambda/API Server -> SQS -> Worker -> DB/S3
```

사용 예:

- 이미지 변환
- 리포트 생성
- 이메일 발송
- 결제 후속 처리

### 트래픽 버퍼링

```text
Producer -> SQS -> Auto Scaling Worker Fleet -> Downstream System
```

Consumer 처리량보다 Producer 유입량이 순간적으로 많아도 SQS가 버퍼 역할을 한다.

### SNS Fan-out

```text
Producer -> SNS Topic -> SQS Queue A -> Service A
                      -> SQS Queue B -> Service B
                      -> SQS Queue C -> Service C
```

하나의 이벤트를 여러 서비스가 독립적으로 처리할 수 있다.

### Lambda Event Source

```text
SQS -> Lambda -> DynamoDB/RDS/S3/OpenSearch
```

서버리스 기반 비동기 처리에 적합하다.

## 18. 운영 Best Practices

- Consumer는 반드시 멱등하게 설계한다.
- Visibility Timeout은 실제 처리 시간보다 충분히 길게 설정한다.
- Long Polling을 사용해 빈 응답과 비용을 줄인다.
- 실패 메시지는 DLQ로 격리한다.
- DLQ 메시지 수에 CloudWatch Alarm을 설정한다.
- Batch API를 사용해 비용과 처리량을 최적화한다.
- 큰 Payload는 S3에 저장하고 SQS에는 참조 정보만 넣는다.
- 메시지 처리 성공 후 반드시 삭제한다.
- `PurgeQueue`는 운영 환경에서 매우 신중하게 사용한다.
- FIFO Queue는 `MessageGroupId` 설계가 처리량을 좌우한다.
- Lambda 연동 시 Lambda Timeout, Visibility Timeout, Batch Size를 함께 조정한다.
- Consumer 장애 시 메시지가 Retention 기간 안에 처리될 수 있도록 보존 기간을 설정한다.

## 19. 시험 관점 핵심 정리

- SQS는 완전관리형 메시지 큐 서비스이다.
- Producer와 Consumer를 분리해 비동기 아키텍처를 구성한다.
- Standard Queue는 높은 처리량, 최소 1회 전달, 최선 노력 순서를 제공한다.
- FIFO Queue는 순서 보장과 중복 제거가 필요할 때 사용한다.
- 메시지를 Receive해도 삭제되지 않으며, 성공 처리 후 Delete해야 한다.
- Visibility Timeout 동안 메시지는 다른 Consumer에게 보이지 않는다.
- Visibility Timeout 안에 삭제하지 않으면 메시지는 다시 처리될 수 있다.
- DLQ는 반복 실패 메시지를 격리하는 용도이다.
- Long Polling은 빈 응답을 줄이고 비용을 절감한다.
- 메시지 보존 기간 기본값은 4일, 최대 14일이다.
- 메시지 최대 크기는 1MiB이다.
- Batch 요청은 최대 10개 메시지를 처리할 수 있다.
- SNS + SQS 조합은 Fan-out 패턴에 자주 사용된다.
- Lambda와 SQS를 연동할 때 Visibility Timeout은 Lambda Timeout보다 길게 설정해야 한다.

## 20. 참고 문서

- [Amazon SQS Developer Guide - What is Amazon SQS?](https://docs.aws.amazon.com/AWSSimpleQueueService/latest/SQSDeveloperGuide/welcome.html)
- [Amazon SQS queue types](https://docs.aws.amazon.com/AWSSimpleQueueService/latest/SQSDeveloperGuide/sqs-queue-types.html)
- [Amazon SQS visibility timeout](https://docs.aws.amazon.com/AWSSimpleQueueService/latest/SQSDeveloperGuide/sqs-visibility-timeout.html)
- [Using dead-letter queues in Amazon SQS](https://docs.aws.amazon.com/AWSSimpleQueueService/latest/SQSDeveloperGuide/sqs-dead-letter-queues.html)
- [Amazon SQS short and long polling](https://docs.aws.amazon.com/AWSSimpleQueueService/latest/SQSDeveloperGuide/sqs-short-and-long-polling.html)
- [Amazon SQS message quotas](https://docs.aws.amazon.com/AWSSimpleQueueService/latest/SQSDeveloperGuide/quotas-messages.html)
