# 🚀 Beanstalk 구성 요소

Elastic Beanstalk를 이해하려면 먼저 Beanstalk가 애플리케이션을 어떤 단위로 관리하는지 알아야
합니다. 슬라이드에서는 Elastic Beanstalk의 핵심 구성 요소를 아래처럼 설명합니다.

```shell
Application
Application Version
Environment
Tier
```

초보자 관점에서는 다음처럼 이해하면 됩니다.

```shell
Application         = 서비스 전체 묶음
Application Version = 배포할 코드 버전
Environment         = 실제 AWS 리소스가 실행되는 환경
Tier                = 웹 요청 처리용인지, 백그라운드 작업 처리용인지 구분하는 환경 유형
```

<br>

## 1. Application

Application은 Elastic Beanstalk에서 관리하는 최상위 애플리케이션 묶음입니다.

예를 들어 쇼핑몰 서비스를 Elastic Beanstalk에 배포한다고 가정해 보겠습니다.

```shell
Application: shopping-app
```

이 `shopping-app` 안에는 여러 코드 버전, 여러 실행 환경, 여러 설정이 포함될 수 있습니다.

```shell
shopping-app
├── Application Version
├── Environment
└── Configuration
```

즉, Application은 실제 서버 하나를 의미하지 않습니다. Beanstalk에서 하나의 서비스를 관리하기 위한
논리적인 상위 단위입니다.

<br>

## 2. Application Version

Application Version은 배포할 애플리케이션 코드의 특정 버전입니다.

예를 들어 같은 쇼핑몰 애플리케이션이라도 코드가 계속 변경될 수 있습니다.

```shell
v1: 최초 배포 버전
v2: 장바구니 기능 추가 버전
v3: 결제 오류 수정 버전
```

Elastic Beanstalk는 이런 코드 묶음을 Application Version으로 관리합니다.

일반적인 흐름은 다음과 같습니다.

```shell
1. 개발자가 새 코드를 준비합니다.
2. 코드를 zip 파일로 패키징합니다.
3. Elastic Beanstalk에 업로드합니다.
4. 새로운 Application Version이 생성됩니다.
5. Environment에 해당 Version을 배포합니다.
```

중요한 점은 Application Version 자체가 실행 환경은 아니라는 점입니다.

Application Version은 "배포 가능한 코드 묶음"이고, 이 코드를 실제로 실행하는 곳은 Environment
입니다.

<br>

## 3. Environment

Environment는 특정 Application Version을 실행하는 AWS 리소스 묶음입니다.

예를 들어 `shopping-app`이라는 Application이 있다면 아래처럼 여러 Environment를 만들 수
있습니다.

```shell
shopping-app
├── dev environment
├── test environment
└── prod environment
```

각 Environment는 실제 AWS 리소스를 가질 수 있습니다.

```shell
Environment
├── EC2
├── Auto Scaling Group
├── Elastic Load Balancer
├── Security Group
└── CloudWatch
```

슬라이드에서 중요한 표현은 다음입니다.

```shell
Collection of AWS resources running an application version
Only one application version at a time
```

즉, Environment는 AWS 리소스들의 묶음이며, 한 번에 하나의 Application Version만 실행합니다.

예를 들어 `prod environment`가 현재 `v1`을 실행 중이라면, 동시에 `v1`과 `v2`를 둘 다 직접
실행하는 구조가 아닙니다.

새 버전을 배포하면 Environment가 실행하는 버전이 `v1`에서 `v2`로 업데이트됩니다.

<br>

## 4. Environment를 여러 개 만드는 이유

Elastic Beanstalk에서는 하나의 Application 아래에 여러 Environment를 만들 수 있습니다.

대표적인 예시는 다음과 같습니다.

| Environment | 목적                                   |
| ----------- | -------------------------------------- |
| dev         | 개발자가 기능을 빠르게 테스트하는 환경 |
| test        | QA나 통합 테스트를 수행하는 환경       |
| prod        | 실제 사용자가 접근하는 운영 환경       |

이렇게 분리하면 새 기능을 바로 운영 환경에 배포하지 않고, 개발 환경과 테스트 환경에서 먼저 검증할 수
있습니다.

```shell
개발자 코드 변경
  ↓
dev environment 배포
  ↓
test environment 검증
  ↓
prod environment 배포
```

운영 관점에서 Environment 분리는 매우 중요합니다. 개발용 설정과 운영용 설정은 보통 다르기 때문입니다.

예를 들어:

- dev는 작은 EC2 인스턴스를 사용할 수 있습니다.
- prod는 Load Balancer와 Auto Scaling을 사용할 수 있습니다.
- dev와 prod는 서로 다른 환경 변수를 사용할 수 있습니다.
- prod는 더 엄격한 보안 그룹과 모니터링 설정이 필요합니다.

<br>

## 5. Tier

Tier는 Elastic Beanstalk Environment의 역할을 구분하는 개념입니다.

슬라이드에서는 두 가지 Tier를 강조합니다.

```shell
Web Server Environment Tier
Worker Environment Tier
```

<br>

### 5.1 Web Server Environment Tier

Web Server Tier는 사용자의 HTTP 요청을 직접 받는 웹 애플리케이션 환경입니다.

일반적인 구조는 다음과 같습니다.

```shell
사용자
  ↓
Route 53
  ↓
Elastic Load Balancer
  ↓
EC2 Web Server
```

예를 들어 다음과 같은 애플리케이션에 적합합니다.

- Spring Boot API 서버
- Node.js 웹 서버
- Python Flask API
- Java Tomcat 애플리케이션
- PHP 웹 애플리케이션

Web Server Tier는 사용자가 브라우저나 API 클라이언트로 요청을 보내면 그 요청을 처리하는 역할을
합니다.

<br>

### 5.2 Worker Environment Tier

Worker Tier는 사용자의 요청을 직접 받기보다 백그라운드 작업을 처리하는 환경입니다.

슬라이드에서는 Worker Tier가 SQS Queue를 기반으로 동작한다고 설명합니다.

```shell
SQS Queue
  ↓
EC2 Worker
  ↓
백그라운드 작업 처리
```

예를 들어 다음 작업에 적합합니다.

- 이메일 발송
- 이미지 리사이징
- 동영상 변환
- 주문 후처리
- 결제 이후 비동기 작업
- 대량 데이터 처리

Worker Environment는 SQS 메시지 수를 기준으로 스케일링할 수 있습니다.

```shell
SQS 메시지 증가
  ↓
Worker EC2 증가
  ↓
처리량 증가
```

또한 Web Server Tier에서 SQS Queue로 메시지를 넣고, Worker Tier가 그 메시지를 처리하는
구조도 가능합니다.

```shell
Web Server Tier
  ↓ 메시지 전송
SQS Queue
  ↓ 메시지 Pull
Worker Tier
```

<br>

## 6. 지원 플랫폼

Elastic Beanstalk는 다양한 언어와 런타임을 지원합니다.

슬라이드 기준 주요 지원 플랫폼은 다음과 같습니다.

| 플랫폼                 | 설명                                       |
| ---------------------- | ------------------------------------------ |
| Go                     | Go 애플리케이션 배포                       |
| Java SE                | 단독 실행 Java 애플리케이션 배포           |
| Java with Tomcat       | Tomcat 기반 Java 웹 애플리케이션 배포      |
| .NET Core on Linux     | Linux 기반 .NET Core 애플리케이션 배포     |
| .NET on Windows Server | Windows Server 기반 .NET 애플리케이션 배포 |
| Node.js                | Node.js 애플리케이션 배포                  |
| PHP                    | PHP 웹 애플리케이션 배포                   |
| Python                 | Python 애플리케이션 배포                   |
| Ruby                   | Ruby 애플리케이션 배포                     |
| Docker                 | 컨테이너 기반 애플리케이션 배포            |

Docker도 지원합니다.

슬라이드에는 아래 Docker 옵션이 포함되어 있습니다.

```shell
Single Container Docker
Multi-container Docker
Preconfigured Docker
```

DVA-C02에서는 Elastic Beanstalk가 특정 언어 하나만 지원하는 서비스가 아니라, 여러 런타임과
Docker 배포를 지원하는 관리형 배포 서비스라는 점을 기억하면 됩니다.

<br>

## 7. 전체 흐름으로 이해하기

Elastic Beanstalk의 구성 요소를 하나의 흐름으로 보면 다음과 같습니다.

```shell
Application 생성
  ↓
Application Version 업로드
  ↓
Environment 생성
  ↓
Environment가 특정 Application Version 실행
  ↓
필요에 따라 Web Server Tier 또는 Worker Tier로 운영
```

예시로 보면 더 쉽습니다.

```shell
Application: shopping-app

Application Versions:
- v1
- v2
- v3

Environments:
- shopping-dev  -> v3 실행
- shopping-test -> v2 실행
- shopping-prod -> v1 실행

Tier:
- shopping-prod-web    -> Web Server Tier
- shopping-prod-worker -> Worker Tier
```

여기서 중요한 점은 Environment마다 서로 다른 Application Version을 실행할 수 있다는 점입니다.

예를 들어 개발 환경은 최신 버전인 `v3`를 실행하고, 운영 환경은 안정화된 `v1`을 계속 실행할 수 있습니다.

<br>

## 8. DVA-C02 시험 포인트

- Application은 Elastic Beanstalk 구성 요소를 묶는 최상위 논리 단위입니다.
- Application Version은 배포 가능한 애플리케이션 코드의 특정 버전입니다.
- Environment는 AWS 리소스 묶음이며, 특정 Application Version을 실행합니다.
- 하나의 Environment는 한 번에 하나의 Application Version만 실행합니다.
- 하나의 Application 아래에 dev, test, prod 같은 여러 Environment를 만들 수 있습니다.
- Web Server Tier는 HTTP 요청을 처리하는 웹 애플리케이션 환경입니다.
- Worker Tier는 SQS Queue 기반으로 백그라운드 작업을 처리하는 환경입니다.
- Worker Tier는 SQS 메시지 수를 기준으로 스케일링할 수 있습니다.
- Elastic Beanstalk는 Java, Node.js, Python, PHP, Ruby, Go, .NET, Docker 등 다양한
  플랫폼을 지원합니다.

<br>

## 9. 시험 예상 문제

### 문제 1

Elastic Beanstalk에서 `Application Version`에 대한 설명으로 가장 올바른 것은 무엇입니까?

A. EC2, Load Balancer, Auto Scaling Group을 포함하는 실행 환경입니다.
B. 배포 가능한 애플리케이션 코드의 특정 버전입니다.
C. 여러 AWS 계정을 묶는 최상위 관리 단위입니다.
D. SQS 메시지를 처리하는 백그라운드 작업자입니다.

<br>

### 정답

B. 배포 가능한 애플리케이션 코드의 특정 버전입니다.

<br>

### 해설

Application Version은 Elastic Beanstalk에 배포할 코드 묶음의 특정 버전입니다.

예를 들어 `v1`, `v2`, `2026-09-01-release` 같은 코드 버전을 Application Version으로
관리할 수 있습니다.

EC2, Load Balancer, Auto Scaling Group 같은 실제 AWS 리소스 묶음은 Environment에
해당합니다.

<br>

### 문제 2

Elastic Beanstalk의 `Environment`에 대한 설명으로 가장 올바른 것은 무엇입니까?

A. 하나의 Environment는 동시에 여러 Application Version을 실행합니다.
B. Environment는 AWS 리소스 묶음이며, 한 번에 하나의 Application Version을 실행합니다.
C. Environment는 애플리케이션 소스 코드만 저장하는 S3 버킷입니다.
D. Environment는 Route 53 Hosted Zone을 의미합니다.

<br>

### 정답

B. Environment는 AWS 리소스 묶음이며, 한 번에 하나의 Application Version을 실행합니다.

<br>

### 해설

Environment는 EC2, Auto Scaling Group, Load Balancer, Security Group 같은 AWS
리소스 묶음입니다.

슬라이드에서도 Environment를 특정 Application Version을 실행하는 AWS 리소스 컬렉션으로 설명합니다.

중요한 제한은 하나의 Environment가 한 번에 하나의 Application Version만 실행한다는 점입니다.

<br>

### 문제 3

Elastic Beanstalk에서 백그라운드 작업을 처리하고, SQS Queue의 메시지 수를 기준으로 스케일링하기에
가장 적합한 Tier는 무엇입니까?

A. Web Server Environment Tier
B. Worker Environment Tier
C. Application Version Tier
D. Database Tier

<br>

### 정답

B. Worker Environment Tier

<br>

### 해설

Worker Environment Tier는 SQS Queue에서 메시지를 가져와 백그라운드 작업을 처리하는 환경입니다.

이메일 발송, 이미지 리사이징, 주문 후처리 같은 비동기 작업에 적합합니다.

Web Server Tier는 HTTP 요청을 직접 처리하는 웹 애플리케이션 환경입니다.

<br>

### 문제 4

하나의 Elastic Beanstalk Application 아래에 `dev`, `test`, `prod` Environment를
각각 생성하는 이유로 가장 적절한 것은 무엇입니까?

A. Beanstalk는 하나의 Environment만 만들 수 있기 때문입니다.
B. 개발, 테스트, 운영 환경을 분리하여 서로 다른 설정과 버전을 운영하기 위해서입니다.
C. Application Version은 Environment 없이 직접 실행되기 때문입니다.
D. Worker Tier는 운영 환경에서 사용할 수 없기 때문입니다.

<br>

### 정답

B. 개발, 테스트, 운영 환경을 분리하여 서로 다른 설정과 버전을 운영하기 위해서입니다.

<br>

### 해설

Elastic Beanstalk에서는 하나의 Application 아래에 여러 Environment를 만들 수 있습니다.

dev, test, prod 환경을 분리하면 새 버전을 먼저 개발/테스트 환경에서 검증한 뒤 운영 환경에 배포할
수 있습니다.

또한 환경마다 EC2 인스턴스 타입, 환경 변수, Auto Scaling 설정, Load Balancer 설정 등을 다르게
가져갈 수 있습니다.

<br>

## 요약

- Elastic Beanstalk의 핵심 구성 요소는 Application, Application Version, Environment,
  Tier입니다.
- Application은 서비스 전체를 묶는 최상위 단위입니다.
- Application Version은 배포할 코드의 특정 버전입니다.
- Environment는 해당 Version을 실제로 실행하는 AWS 리소스 묶음입니다.
- Tier는 Environment의 역할을 구분하며, Web Server Tier는 HTTP 요청을 처리하고
  Worker Tier는 SQS 기반 백그라운드 작업을 처리합니다.
