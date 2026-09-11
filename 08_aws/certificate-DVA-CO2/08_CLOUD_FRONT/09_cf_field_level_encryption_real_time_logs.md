# CloudFront Field-Level Encryption and Real-Time Logs

Field-Level Encryption과 Real-Time Logs는 분량은 짧지만 시험 선택지에 섞이기 좋은 기능입니다.

## Field-Level Encryption

Field-Level Encryption은 사용자의 민감한 POST field를 CloudFront Edge Location에서 암호화하는 기능입니다.

HTTPS와 함께 추가적인 보안 계층을 제공합니다.

```text
Client
  |
  | HTTPS
  v
CloudFront Edge Location
  |
  | 특정 POST field 암호화
  v
Origin
  |
  | private key로 복호화
  v
Application
```

## 동작 방식

Field-Level Encryption은 비대칭 암호화를 사용합니다.

| 키 | 위치 | 역할 |
| --- | --- | --- |
| Public Key | CloudFront에 등록 | Edge에서 민감 필드 암호화 |
| Private Key | 애플리케이션 또는 안전한 서버 측 보관 | Origin/Application에서 복호화 |

CloudFront는 지정된 POST field를 public key로 암호화합니다.
Origin에 도착한 애플리케이션은 private key로 해당 field를 복호화합니다.

## 사용 조건

| 항목 | 설명 |
| --- | --- |
| 대상 요청 | POST request |
| 암호화 대상 | 지정한 field |
| 필드 수 | 최대 10개 field |
| 암호화 방식 | Asymmetric encryption |
| 목적 | 애플리케이션 스택 내부에서도 민감 정보 노출 최소화 |

대표 예시는 결제 정보, 주민등록번호에 준하는 민감 식별자, 개인정보 입력 field입니다.

## Real-Time Logs

Real-Time Logs는 CloudFront가 받은 요청 로그를 거의 실시간으로 Kinesis Data Streams에 보내는 기능입니다.

```text
Users
  |
  v
CloudFront
  |
  v
Kinesis Data Streams
  |
  v
Lambda / Kinesis Data Firehose / Analytics
```

## Real-Time Logs 설정 포인트

Real-Time Logs에서는 다음을 선택할 수 있습니다.

| 설정 | 설명 |
| --- | --- |
| Sampling Rate | 로그를 받을 요청 비율 |
| Fields | 로그에 포함할 field |
| Cache Behaviors | 어떤 path pattern의 요청 로그를 받을지 선택 |

## Real-Time Processing vs Near Real-Time Processing

| 처리 방식 | 구조 |
| --- | --- |
| Real-time processing | CloudFront -> Kinesis Data Streams -> Lambda |
| Near real-time processing | CloudFront -> Kinesis Data Streams -> Kinesis Data Firehose |

## 시험 함정

| 문제 표현 | 정답 방향 |
| --- | --- |
| "Encrypt sensitive POST fields at the edge" | Field-Level Encryption |
| "Use public key at CloudFront and private key at application" | Field-Level Encryption |
| "Up to 10 fields" | Field-Level Encryption |
| "Send CloudFront request logs to Kinesis Data Streams" | Real-Time Logs |
| "Choose sampling rate and specific fields" | Real-Time Logs |
| "Choose specific cache behaviors for logs" | Real-Time Logs |

## 한 줄 요약

Field-Level Encryption은 민감한 POST field를 Edge에서 암호화하는 기능이고, Real-Time Logs는 CloudFront
요청 로그를 Kinesis Data Streams로 거의 실시간 전송하는 기능입니다.
