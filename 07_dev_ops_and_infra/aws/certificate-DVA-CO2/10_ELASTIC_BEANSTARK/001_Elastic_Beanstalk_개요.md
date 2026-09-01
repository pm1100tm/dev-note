# 🚀 Elastic Beanstalk 개요

Elastic Beanstalk는 AWS에서 애플리케이션을 쉽고 안전하게 배포하기 위한 관리형 서비스이다.

한마디로 말하면:

> 개발자는 애플리케이션 코드에 집중하고, Elastic Beanstalk가 EC2, Auto Scaling,
> Load Balancer 같은 인프라 구성을 대신 관리해주는 서비스

<br>

## 1. 왜 Elastic Beanstalk가 필요한가?

AWS에 웹 애플리케이션을 직접 배포하려면 생각보다 많은 인프라를 구성해야 한다.

예를 들어 일반적인 3-Tier 웹 애플리케이션은 아래와 같은 구조를 가진다.

```shell
Route 53
  ↓
Elastic Load Balancer
  ↓
EC2 Auto Scaling Group
  ↓
Application Server
  ↓
RDS / Elastic Cache
```

개발자가 직접 해야 하는 작업은 다음과 같다.

- EC2 인스턴스 생성
- 보안 그룹 설정
- Load Balancer 구성
- Auto Scaling Group 구성
- RDS 연결 설정
- 애플리케이션 배포
- 로그 확인
- 헬스 체크 설정
- 장애 발생 시 복구
- 트래픽 증가 시 스케일링

초보자 입장에서는 애플리케이션 코드보다 인프라 설정에서 더 많이 막힐 수 있다.

Elastic Beanstalk는 이런 반복적인 인프라 작업을 자동화해서, 개발자가 코드를 배포하고 실행하는 데
집중할 수 있게 해준다.

<br>

## 2. Elastic Beanstalk가 해주는 일

Elastic Beanstalk는 내부적으로 여러 AWS 서비스를 조합해서 애플리케이션 실행 환경을 만든다.

대표적으로 아래 서비스를 사용한다.

| 내부에서 사용하는 AWS 서비스 | 역할                             |
| ---------------------------- | -------------------------------- |
| EC2                          | 애플리케이션이 실행되는 서버     |
| Auto Scaling Group           | 트래픽에 따라 EC2 수를 자동 조절 |
| Elastic Load Balancer        | 요청을 여러 EC2로 분산           |
| Security Group               | 네트워크 접근 제어               |
| CloudWatch                   | 로그, 메트릭, 헬스 모니터링      |
| RDS                          | 필요 시 데이터베이스 구성        |

즉, Elastic Beanstalk는 새로운 컴퓨팅 기술이라기보다 기존 AWS 서비스들을 개발자 친화적으로
묶어주는 배포 플랫폼에 가깝다.

<br>

## 3. 개발자의 책임과 Beanstalk의 책임

Elastic Beanstalk를 사용할 때 책임 범위를 구분하는 것이 중요하다.

| 구분              | 책임                                                          |
| ----------------- | ------------------------------------------------------------- |
| 개발자            | 애플리케이션 코드, 의존성 파일, 환경 변수, 주요 설정          |
| Elastic Beanstalk | 인프라 생성, 배포 실행, 로드 밸런싱, 오토 스케일링, 헬스 체크 |
| 사용자            | 하위 리소스 비용 관리, 운영 설정 검토, 보안 구성 확인         |

슬라이드의 핵심 표현은 다음과 같다.

```shell
Just the application code is the responsibility of the developer
```

즉, 개발자의 핵심 책임은 애플리케이션 코드이다.

다만 모든 것이 완전히 자동이라는 뜻은 아니다. Beanstalk는 관리형 서비스지만, 사용자는 여전히 설정을
직접 제어할 수 있다.

예를 들어:

- EC2 인스턴스 타입
- Auto Scaling 설정
- Load Balancer 설정
- 환경 변수
- 배포 방식
- 로그 설정
- VPC/Subnet 설정

이런 설정은 필요에 따라 직접 조정할 수 있다.

<br>

## 4. Elastic Beanstalk는 서버리스인가?

Elastic Beanstalk는 서버리스 서비스가 아니다.

서버를 직접 만들지 않는 것처럼 보이지만, 실제 내부에서는 EC2 인스턴스, Load Balancer,
Auto Scaling Group 등이 생성된다.

따라서 Beanstalk 자체 사용 요금은 없지만, Beanstalk가 생성한 하위 AWS 리소스 비용은 발생한다.

```shell
Elastic Beanstalk 자체 = 무료
EC2, ELB, RDS, EBS 등 하위 리소스 = 과금
```

DVA-C02 시험에서도 이 부분은 자주 헷갈릴 수 있다.

<br>

## 5. Elastic Beanstalk 주요 구성 요소

Elastic Beanstalk는 크게 Application, Application Version, Environment로 이해하면 된다.

<br>

### 5.1 Application

Application은 Elastic Beanstalk에서 관리하는 애플리케이션의 최상위 단위이다.

예를 들어 `my-shopping-app`이라는 서비스를 Beanstalk에 올린다면, 이 전체 묶음이 Application이다.

Application 안에는 여러 버전과 여러 환경이 포함될 수 있다.

```shell
Application: my-shopping-app
```

<br>

### 5.2 Application Version

Application Version은 배포할 애플리케이션 코드의 특정 버전이다.

예를 들어 아래처럼 여러 버전을 가질 수 있다.

```shell
v1
v2
v3
2026-09-01-release
```

새 코드를 배포하면 새로운 Application Version이 만들어지고, Environment는 그 버전을 실행한다.

<br>

### 5.3 Environment

Environment는 실제 AWS 리소스가 실행되는 환경이다.

예를 들어 하나의 Application에 대해 아래처럼 여러 Environment를 만들 수 있다.

```shell
dev
test
prod
```

각 Environment는 EC2, Load Balancer, Auto Scaling Group 같은 AWS 리소스 묶음을 가진다.

중요한 점:

- 하나의 Environment는 한 번에 하나의 Application Version만 실행한다.
- 여러 Environment를 만들어 개발, 테스트, 운영 환경을 분리할 수 있다.

<br>

## 6. Elastic Beanstalk의 기본 배포 흐름

Elastic Beanstalk의 기본 흐름은 다음과 같다.

```shell
1. Application 생성
2. Application Version 업로드
3. Environment 생성
4. Environment에 Application Version 배포
5. 헬스 체크와 로그를 보며 운영
6. 새 버전이 나오면 Environment의 Version 업데이트
```

조금 더 쉽게 표현하면:

```shell
코드 업로드
  ↓
Beanstalk가 인프라 준비
  ↓
EC2에 애플리케이션 배포
  ↓
Load Balancer로 트래픽 연결
  ↓
헬스 체크와 스케일링 수행
```

<br>

## 7. 초보자 관점의 비유

직접 AWS 인프라를 구성하는 것은 식당을 열기 위해 건물, 전기, 수도, 주방 설비, 직원 배치까지 직접
준비하는 것과 비슷하다.

Elastic Beanstalk는 이미 기본 설비가 갖춰진 주방을 빌리는 것에 가깝다.

개발자는 요리, 즉 애플리케이션 코드에 집중하고, Beanstalk는 서버, 로드 밸런서, 스케일링 같은
운영 환경을 대신 준비한다.

하지만 주방 사용료는 내야 한다. Beanstalk 자체는 무료지만, EC2나 Load Balancer 같은 실제 리소스
비용은 발생하기 때문이다.

<br>

## 8. DVA-C02 시험 포인트

- Elastic Beanstalk는 개발자 중심의 애플리케이션 배포 서비스이다.
- 내부적으로 EC2, Auto Scaling Group, Elastic Load Balancer, RDS 등을 사용할 수 있다.
- Beanstalk는 관리형 서비스이며, capacity provisioning, load balancing, scaling,
  health monitoring 등을 자동 처리한다.
- 개발자의 주요 책임은 애플리케이션 코드이다.
- Beanstalk는 무료지만, 하위 리소스 비용은 발생한다.
- Beanstalk는 서버리스가 아니다. 내부적으로 EC2 기반 리소스를 생성할 수 있다.
- Application은 Beanstalk 구성요소의 전체 묶음이다.
- Application Version은 애플리케이션 코드의 특정 버전이다.
- Environment는 특정 Application Version을 실행하는 AWS 리소스 묶음이다.
- 하나의 Environment는 한 번에 하나의 Application Version만 실행한다.
- dev, test, prod처럼 여러 Environment를 만들 수 있다.

<br>

## 9. 시험 예상 문제

### 문제 1

한 개발팀은 AWS에서 웹 애플리케이션을 배포하려고 한다. 팀은 EC2, Auto Scaling Group,
Load Balancer, 헬스 체크 등을 직접 구성하는 부담을 줄이고, 애플리케이션 코드 배포에 집중하고 싶다.
이 요구사항에 가장 적합한 AWS 서비스는 무엇인가?

A. AWS Lambda
B. AWS Elastic Beanstalk
C. Amazon S3
D. Amazon Route 53

<br>

### 정답

B. AWS Elastic Beanstalk

<br>

### 해설

Elastic Beanstalk는 개발자 중심의 애플리케이션 배포 관리형 서비스이다.

개발자가 애플리케이션 코드를 업로드하면, Beanstalk가 내부적으로 EC2, Auto Scaling Group,
Elastic Load Balancer 등을 사용해서 실행 환경을 구성한다.

Lambda는 서버리스 함수 실행 서비스이고, S3는 객체 스토리지, Route 53은 DNS 서비스이므로 문제의
요구사항과 가장 직접적으로 맞지 않는다.

<br>

### 문제 2

Elastic Beanstalk에 대한 설명으로 올바른 것은?

A. Elastic Beanstalk는 항상 서버리스 방식으로만 실행된다.
B. Elastic Beanstalk 자체는 무료이지만, 생성되는 EC2나 Load Balancer 같은 하위 리소스 비용은 발생한다.
C. 하나의 Environment는 동시에 여러 Application Version을 실행해야 한다.
D. Elastic Beanstalk를 사용하면 사용자는 모든 설정을 변경할 수 없다.

<br>

### 정답

B. Elastic Beanstalk 자체는 무료이지만, 생성되는 EC2나 Load Balancer 같은 하위 리소스 비용은 발생한다.

<br>

### 해설

Elastic Beanstalk는 관리형 서비스이며 자체 추가 요금은 없다. 하지만 Beanstalk가 생성하는
EC2, ELB, RDS, EBS 등의 하위 리소스는 일반 AWS 요금에 따라 과금된다.

또한 Beanstalk는 서버리스가 아니며, 사용자는 EC2 타입, 스케일링, 로드 밸런서, 환경 변수 등 다양한
설정을 제어할 수 있다.

<br>

## 요약

Elastic Beanstalk는 애플리케이션을 AWS에 쉽고 일관되게 배포하기 위한 관리형 서비스이다.

개발자는 코드와 주요 설정에 집중하고, Beanstalk는 EC2, Auto Scaling Group, Load Balancer,
헬스 체크, 스케일링 같은 인프라 운영 요소를 자동으로 처리한다.

DVA-C02에서는 Elastic Beanstalk를 "개발자 중심 배포 서비스",
"하위 AWS 리소스를 자동 구성하는 관리형 서비스", "Beanstalk 자체는 무료지만 하위 리소스는 과금"이라는
관점으로 기억하면 된다.
