# 🚀 Elastic Beanstalk Tier

Elastic Beanstalk의 Tier는 Environment가 어떤 종류의 작업을 처리하는지 구분하는 개념입니다.

여기서는 두 가지 Tier를 강조합니다.

```shell
Web Server Environment Tier
Worker Environment Tier
```

초보자 관점에서는 다음처럼 이해하면 됩니다.

```shell
Web Server Tier = 사용자의 HTTP 요청을 직접 받는 웹 서버 환경
Worker Tier     = SQS 메시지를 기반으로 백그라운드 작업을 처리하는 환경
```

<br>

## 1. Tier가 필요한 이유

웹 애플리케이션은 보통 두 종류의 일을 처리합니다.

첫 번째는 사용자가 바로 응답을 기다리는 작업입니다.

예를 들어:

- 로그인 요청
- 상품 목록 조회
- 주문 생성 API 호출
- 게시글 작성 요청
- 파일 업로드 요청

두 번째는 사용자가 즉시 기다릴 필요가 없는 백그라운드 작업입니다.

예를 들어:

- 이메일 발송
- 이미지 리사이징
- 주문 완료 후 정산 처리
- 동영상 변환
- 대량 데이터 처리
- 외부 시스템 연동 재시도

Elastic Beanstalk는 이런 작업 성격에 맞춰 Web Server Tier와 Worker Tier를 제공합니다.

<br>

## 2. Web Server Environment Tier

Web Server Environment Tier는 사용자의 HTTP 요청을 직접 처리하는 환경입니다.

일반적인 웹 애플리케이션이나 API 서버는 Web Server Tier에 배포합니다.

구조는 다음과 같습니다.

```shell
사용자 / 클라이언트
  ↓
Route 53
  ↓
Elastic Load Balancer
  ↓
EC2 Instance - Web Server
```

슬라이드에서도 Web Environment는 Load Balancer와 Auto Scaling Group 뒤에
EC2 Web Server가 배치된 구조로 설명됩니다.

<br>

## 3. Web Server Tier의 특징

Web Server Tier는 사용자 요청을 직접 받기 때문에 응답 시간과 가용성이 중요합니다.

주요 특징은 다음과 같습니다.

- HTTP 또는 HTTPS 요청을 처리합니다.
- Load Balancer를 통해 여러 EC2 인스턴스로 트래픽을 분산할 수 있습니다.
- Auto Scaling Group을 통해 트래픽 증가에 따라 EC2 수를 조절할 수 있습니다.
- 퍼블릭 엔드포인트를 가질 수 있습니다.
- 일반적인 API 서버, 웹 서버, 백엔드 서버에 적합합니다.

예를 들어 Spring Boot API 서버를 Elastic Beanstalk에 배포할 경우 대부분
Web Server Tier를 사용합니다.

```shell
브라우저 / 모바일 앱
  ↓
API 요청
  ↓
Web Server Tier
  ↓
Spring Boot Application
```

<br>

## 4. Worker Environment Tier

Worker Environment Tier는 백그라운드 작업을 처리하는 환경입니다.

Worker Tier는 사용자의 HTTP 요청을 직접 받는 구조가 아니라, SQS Queue에 쌓인 메시지를 가져와
처리하는 구조입니다.

슬라이드의 핵심은 다음입니다.

```shell
SQS Queue
  ↓
EC2 Worker
  ↓
백그라운드 작업 처리
```

Worker Tier는 메시지를 Pull 방식으로 가져와 처리합니다.

즉, 작업 요청이 SQS Queue에 쌓이면 Worker EC2 인스턴스가 메시지를 가져와 작업을 수행합니다.

<br>

## 5. Worker Tier의 특징

Worker Tier는 비동기 처리에 적합합니다.

주요 특징은 다음과 같습니다.

- SQS Queue 기반으로 작업을 처리합니다.
- EC2 Worker 인스턴스가 SQS 메시지를 가져와 처리합니다.
- SQS 메시지 수를 기준으로 스케일링할 수 있습니다.
- 사용자가 즉시 응답을 기다리지 않아도 되는 작업에 적합합니다.
- Web Server Tier에서 SQS Queue로 메시지를 넣고 Worker Tier가 처리하는 구조가 가능합니다.

예를 들어 주문 API가 호출된 뒤 이메일 발송 작업을 분리할 수 있습니다.

```shell
사용자 주문 요청
  ↓
Web Server Tier에서 주문 저장
  ↓
SQS Queue에 "주문 완료 이메일 발송" 메시지 전송
  ↓
Worker Tier가 메시지 처리
  ↓
이메일 발송
```

이 구조를 사용하면 사용자는 주문 완료 응답을 빠르게 받을 수 있고, 이메일 발송은 뒤에서 안정적으로
처리할 수 있습니다.

<br>

## 6. Web Server Tier와 Worker Tier 비교

| 구분             | Web Server Tier                  | Worker Tier                           |
| ---------------- | -------------------------------- | ------------------------------------- |
| 주요 역할        | HTTP 요청 처리                   | 백그라운드 작업 처리                  |
| 트래픽 진입점    | Load Balancer 또는 웹 엔드포인트 | SQS Queue                             |
| 처리 방식        | 사용자의 요청에 즉시 응답        | Queue 메시지를 비동기로 처리          |
| 대표 예시        | API 서버, 웹 서버                | 이메일 발송, 이미지 처리, 주문 후처리 |
| 스케일링 기준    | CPU, Request Count 등            | SQS 메시지 수                         |
| 사용자 대기 여부 | 사용자가 응답을 기다림           | 사용자가 직접 기다리지 않음           |

<br>

## 7. Web Server Tier와 Worker Tier를 함께 쓰는 구조

실무에서는 Web Server Tier와 Worker Tier를 함께 사용하는 경우가 많습니다.

예를 들어 쇼핑몰 서비스에서는 다음 구조를 사용할 수 있습니다.

```shell
사용자
  ↓
Web Server Tier
  ↓
주문 데이터 저장
  ↓
SQS Queue
  ↓
Worker Tier
  ↓
이메일 발송 / 재고 동기화 / 정산 처리
```

이렇게 나누면 사용자 응답이 빨라지고, 시간이 오래 걸리는 작업을 별도 Worker 환경에서 안정적으로
처리할 수 있습니다.

또한 Worker Tier는 SQS 메시지 수에 따라 스케일링할 수 있으므로, 작업량이 급증해도 처리량을
늘리기 쉽습니다.

<br>

## 8. 초보자가 헷갈리기 쉬운 부분

### 8.1 Worker Tier는 사용자의 웹 요청을 직접 받는 환경이 아닙니다

Worker Tier는 웹 서버가 아니라 백그라운드 작업자 환경입니다.

사용자 요청은 보통 Web Server Tier가 받고, Worker Tier는 SQS Queue의 메시지를 처리합니다.

<br>

### 8.2 Worker Tier는 SQS와 밀접하게 연결됩니다

슬라이드에서는 Worker Tier가 SQS Queue 메시지를 Pull해서 처리하는 구조를 보여줍니다.

따라서 시험에서 `SQS`, `background job`, `asynchronous processing`, `queue messages`
같은 표현이 나오면 Worker Tier를 의심해야 합니다.

<br>

### 8.3 Web Server Tier와 Worker Tier는 서로 다른 Environment입니다

Web Server Tier와 Worker Tier는 하나의 Environment 안에서 역할만 바뀌는 개념이 아니라,
서로 다른 유형의 Elastic Beanstalk Environment로 이해하는 편이 좋습니다.

예를 들어 운영 환경에서 아래처럼 나눌 수 있습니다.

```shell
shopping-prod-web    = Web Server Tier
shopping-prod-worker = Worker Tier
```

<br>

## 9. DVA-C02 시험 포인트

- Elastic Beanstalk에는 Web Server Environment Tier와 Worker Environment Tier가 있습니다.
- Web Server Tier는 HTTP/HTTPS 요청을 처리하는 일반적인 웹 애플리케이션 환경입니다.
- Web Server Tier는 Load Balancer와 Auto Scaling Group을 함께 사용할 수 있습니다.
- Worker Tier는 SQS Queue 기반으로 백그라운드 작업을 처리합니다.
- Worker Tier의 EC2 인스턴스는 SQS 메시지를 Pull해서 처리합니다.
- Worker Tier는 SQS 메시지 수를 기준으로 스케일링할 수 있습니다.
- Web Server Tier에서 SQS Queue로 메시지를 넣고 Worker Tier가 처리하는 구조가 가능합니다.
- 이메일 발송, 이미지 리사이징, 주문 후처리 같은 비동기 작업은 Worker Tier에 적합합니다.

<br>

## 10. 시험 예상 문제

### 문제 1

개발팀이 Elastic Beanstalk에서 사용자의 HTTP 요청을 직접 처리하는 API 서버를 배포하려고 합니다.
가장 적절한 Tier는 무엇입니까?

A. Worker Environment Tier
B. Web Server Environment Tier
C. Database Tier
D. Queue Tier

<br>

### 정답

B. Web Server Environment Tier

<br>

### 해설

Web Server Environment Tier는 사용자의 HTTP 또는 HTTPS 요청을 직접 처리하는 환경입니다.
API 서버, 웹 서버, 일반적인 백엔드 애플리케이션은 Web Server Tier에 적합합니다.
Worker Tier는 SQS 메시지를 기반으로 백그라운드 작업을 처리할 때 사용합니다.

<br>

### 문제 2

Elastic Beanstalk에서 이메일 발송, 이미지 리사이징 같은 비동기 작업을 SQS Queue 기반으로
처리하려고 합니다. 가장 적절한 Tier는 무엇입니까?

A. Web Server Environment Tier
B. Worker Environment Tier
C. Application Version
D. Load Balancer Tier

<br>

### 정답

B. Worker Environment Tier

<br>

### 해설

Worker Environment Tier는 SQS Queue에 쌓인 메시지를 가져와 백그라운드 작업을 처리하는 환경입니다.
사용자가 즉시 응답을 기다릴 필요가 없는 작업, 예를 들어 이메일 발송, 이미지 처리, 주문 후처리 같은 작업에 적합합니다.
Web Server Tier는 사용자의 HTTP 요청을 직접 처리하는 환경입니다.

<br>

### 문제 3

Elastic Beanstalk Worker Environment Tier의 스케일링 기준으로 가장 적절한 것은 무엇입니까?

A. SQS Queue의 메시지 수
B. Route 53 Hosted Zone 개수
C. S3 버킷 개수
D. IAM User 개수

<br>

### 정답

A. SQS Queue의 메시지 수

<br>

### 해설

슬라이드에서는 Worker Environment가 SQS 메시지 수를 기준으로 스케일링할 수 있다고 설명합니다.
작업 메시지가 많이 쌓이면 Worker EC2 인스턴스를 늘려 처리량을 높일 수 있습니다.
Route 53, S3, IAM User 개수는 Worker Tier의 스케일링 기준과 관련이 없습니다.

<br>

### 문제 4

다음 중 Elastic Beanstalk의 Web Server Tier와 Worker Tier에 대한 설명으로 가장 올바른 것은 무엇입니까?

A. Web Server Tier는 SQS 메시지만 처리합니다.
B. Worker Tier는 사용자의 HTTP 요청을 직접 처리하는 웹 서버 환경입니다.
C. Web Server Tier는 HTTP 요청을 처리하고, Worker Tier는 SQS 기반 백그라운드 작업을 처리합니다.
D. Worker Tier는 Load Balancer 없이는 절대 실행할 수 없습니다.

<br>

### 정답

C. Web Server Tier는 HTTP 요청을 처리하고, Worker Tier는 SQS 기반 백그라운드 작업을 처리합니다.

<br>

### 해설

Web Server Tier는 사용자 또는 클라이언트의 HTTP 요청을 처리하는 환경입니다.
Worker Tier는 SQS Queue에 쌓인 메시지를 가져와 백그라운드 작업을 처리하는 환경입니다.
따라서 두 Tier는 역할이 다르며, 애플리케이션의 동기 요청 처리와 비동기 작업 처리를 분리할 때 함께 사용할 수 있습니다.

<br>

## 요약

- Elastic Beanstalk Tier는 Environment의 역할을 구분하는 개념입니다.
- Web Server Tier는 사용자의 HTTP 요청을 처리하는 웹 애플리케이션 환경입니다.
- Worker Tier는 SQS Queue 기반으로 백그라운드 작업을 처리하는 환경입니다.
- DVA-C02에서는 `HTTP 요청 = Web Server Tier`, `SQS + 비동기 작업 = Worker Tier`로 구분하면 됩니다.
