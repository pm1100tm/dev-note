# 🚀 RDS Proxy

Amazon RDS Proxy 는 RDS 와 Aurora 를 위한 완전관리형 데이터베이스 프록시 서비스입니다.
즉, 애플리케이션과 데이터베이스 사이에 위치하여, DB 연결(Connection)을 대신 관리하고 최적화합니다.

## 🔹 핵심 개념 요약

| 항목        | 설명                                                     |
| ----------- | -------------------------------------------------------- |
| 서비스 유형 | 완전관리형 DB 프록시 (Fully Managed)                     |
| 지원 엔진   | RDS / Aurora (MySQL, PostgreSQL, MariaDB, MS SQL Server) |
| 특징        | Connection pooling, auto-scaling, Multi-AZ               |
| 요금        | 시간당 사용 요금 + 처리된 연결 수 기준                   |
| 보안 통합   | IAM 인증, Secrets Manager 연동                           |
| 공개 여부   | ❌ Public Access 불가 (VPC 내부에서만 접근 가능)         |

여러 애플리케이션 인스턴스가 동시에 DB에 접근할 때, 각각 DB 커넥션을 새로 여는 대신
👉 Proxy가 연결을 미리 유지해 두고 공유(Pooling) 합니다.

![rds-proxy](./assets/rds-proxy.png)

## 🚀 주요 기능

### 1️⃣ Connection Pooling (커넥션 풀링)

- Proxy가 DB 연결을 미리 만들어 유지하고, 앱에서 요청이 오면 즉시 기존 연결을 재사용.
  - DB의 연결 생성/해제 부하 감소 (CPU, RAM 절약)
  - 특히 Lambda처럼 짧은 수명 함수 호출에 매우 유용

### 2️⃣ Connection Multiplexing

- 여러 애플리케이션 연결을 하나의 DB 연결로 매핑하여 DB 인스턴스에 오픈된 연결 수를 최소화

### 3️⃣ 자동 장애 조치 (Failover Optimization)

- RDS 또는 Aurora의 Failover 시간 최대 66% 단축(Proxy가 기존 연결을 유지하고 자동 재연결 수행)

### 4️⃣ 보안 통합

- IAM 인증을 강제할 수 있음
  - → 앱이 DB 계정 ID/PW를 직접 갖지 않아도 됨
- AWS Secrets Manager에 DB 자격 증명을 안전하게 저장
  - → Proxy가 자동으로 인증 정보 관리

### 5️⃣ 고가용성 & Auto Scaling

- Multi-AZ 구조로 자동 복제 및 장애 대응
- 연결 요청량에 따라 자동 확장(Auto Scaling)

### 6️⃣ 코드 수정 거의 불필요

- 대부분의 애플리케이션은 기존 DB Endpoint → Proxy Endpoint로만 교체하면 됨

### 예시(Spring boot 환경)

```java
// 일반
spring:
  datasource:
    url: jdbc:postgresql://myspeech-db.xxxxxx.ap-northeast-2.rds.amazonaws.com:5432/appdb
    username: admin
    password: mypassword

// proxy 사용 시
spring:
  datasource:
    url: jdbc:postgresql://myspeech-proxy.proxy-abcdefghijkl.ap-northeast-2.rds.amazonaws.com:5432/appdb
    username: admin
    password: mypassword

// ✅ 코드 변경 거의 없음
// 단지 DB endpoint 주소만 Proxy endpoint로 교체
```

## 🔒 보안 관련 사항

- IAM Authentication: DB 로그인에 IAM 토큰 사용 가능
- Secrets Manager 연동: 자격증명(Username/Password) 자동 관리
- VPC 내부 접근만 허용: Public Access 불가
- SSL/TLS 암호화 지원: 네트워크 구간 암호화 (In-transit Encryption)

## ⚡ RDS Proxy 도입 시 이점

- ✅ DB 커넥션 효율화: Connection pooling, multiplexing
- ✅ Failover 단축: 장애 시 재연결 자동화
- ✅ Lambda 호환성 향상: 단발성 연결 문제 해결
- ✅ 보안 강화: IAM 인증 + Secrets Manager 통합
- ✅ 비용 절감: 불필요한 DB 연결 감소로 CPU 절약

⚠️ 주의사항

- Publicly Accessible 설정 불가 (VPC 내부 접근만 가능)
- RDS Proxy 사용 시 약간의 지연(Latency) 추가 가능
- Aurora Serverless v1 은 지원되지 않음 (v2부터 가능)

💡 한 줄 요약

Amazon RDS Proxy는 “DB 앞단의 연결 관리자(Connection Manager)”로서,
**커넥션 풀링, 장애 복구, 보안 강화**를 자동으로 처리해주는 완전관리형 고가용성 프록시 서비스

---

## 질문

### ❓1️⃣ “Proxy 설정을 하면, 로컬에서 public하게 접속이 안되네?”

Amazon RDS Proxy는 항상 VPC 내부 전용 서비스입니다. 즉, Public Access 불가 —
외부(로컬 PC, 외부 네트워크 등)에서는 직접 연결할 수 없습니다.

🔒 이유

- RDS Proxy는 보안을 위해 퍼블릭 IP를 부여하지 않습니다.
- 오직 VPC 내부 또는 VPC 피어링 / PrivateLink / VPN / Direct Connect 등을 통해서만 접근 가능.

🔧 실무 해결 방법

- 🧩 EC2 Bastion Host: VPC 내부의 EC2에서 Proxy에 접근 (SSH Tunnel)
- 🔁 AWS Systems Manager Session Manager: 로컬 → VPC 내부 프록시로 터널링
- 로컬 개발 환경: Proxy 대신 직접 RDS 엔드포인트로 연결 (테스트용)

> 💡 즉, 개발/테스트 환경에서는 RDS 일반 엔드포인트를 사용하는 경우도 있습니다.

### ❓2️⃣ “Proxy 설정을 하면 비용이 추가로 발생하는 것인가?”

추가 요금이 발생합니다.

- RDS Proxy는 별도의 관리형 서비스이기 때문에, RDS 인스턴스 요금과는 별도로 청구됩니다.

#### 💰 요금 구조 (서울 리전 기준 예시)

- 활성 연결 시간당 요금: 약 $0.015 ~ $0.03 per vCPU-hour 수준
- 데이터 처리량: GB당 $0.01 수준 (작음)
- 추가 요금 없음: 연결 요청 수나 스토리지 용량에는 영향 없음

요금은 실제로 프록시가 처리한 연결 수 및 시간 기준으로 계산됩니다.

> ⚙️ AWS Console → RDS → Proxies → Monitoring 에서 사용량 확인 가능

📈 실제 체감

- 트래픽이 적은 경우 비용은 미미합니다.
- Lambda + RDS 조합처럼 커넥션 수가 폭증하는 구조에서는

Proxy를 써서 DB CPU 사용률 절감 효과 > 추가 요금 인 경우가 많습니다.

### ❓3️⃣ “Proxy를 쓰면 IAM이나 Secrets Manager도 코드에서 대응해야 하나?”

✅ 기본적으로는 “필수는 아니지만, 권장”입니다.

#### 🔹 (1) 단순히 Proxy만 쓴다면 — 코드 변경 거의 없음

기존 RDS 연결처럼 username/password를 application.yml에 넣어도 됩니다.

```java
spring:
  datasource:
    url: jdbc:postgresql://mydb-proxy.proxy-xxxxx.ap-northeast-2.rds.amazonaws.com:5432/appdb
    username: admin
    password: mypassword
```

Proxy는 내부적으로 RDS와 연결을 관리하기 때문에 애플리케이션 코드는 변경하지 않아도 작동합니다.

#### 🔹 (2) 보안을 강화하려면 — IAM 인증 or Secrets Manager 연동 사용

✅ A. IAM Authentication

- RDS에서 IAM 인증 활성화 후,
- Spring 애플리케이션이 AWS SDK를 통해 임시 인증 토큰(Token) 을 받아 DB에 연결.
- 이 토큰은 약 15분 유효하며, Proxy가 이를 검증.

- 장점: DB 비밀번호를 코드나 환경변수에 저장하지 않아도 됨
- 단점: 토큰을 주기적으로 갱신해야 하므로 SDK 코드 필요

```java
String token = RdsIamAuthTokenGenerator.builder()
    .credentialsProvider(DefaultCredentialsProvider.create())
    .region(Region.AP_NORTHEAST_2)
    .hostname("mydb-proxy.proxy-xxxxx.ap-northeast-2.rds.amazonaws.com")
    .port(5432)
    .username("admin")
    .build()
    .getAuthToken();
```

✅ B. Secrets Manager 연동

- RDS Proxy가 AWS Secrets Manager 에서 DB 자격증명을 자동으로 가져옴.
- 애플리케이션은 여전히 username/password를 사용하지만,
  Proxy가 백그라운드에서 실제 비밀번호를 대체 관리함.

즉, 앱은 여전히 기존처럼 동작하지만, AWS가 DB 자격증명을 대신 관리 (자동 회전 포함)
