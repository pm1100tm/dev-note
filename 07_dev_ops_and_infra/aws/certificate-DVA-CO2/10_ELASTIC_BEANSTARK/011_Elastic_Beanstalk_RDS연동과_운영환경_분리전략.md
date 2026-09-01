# 🚀 Elastic Beanstalk RDS 연동과 운영 환경 분리 전략

Elastic Beanstalk는 Environment를 만들 때 RDS 데이터베이스를 함께 생성할 수 있습니다.

이 방식은 개발 환경이나 테스트 환경에서는 편리합니다.

하지만 운영 환경에서는 RDS를 Elastic Beanstalk Environment와 분리해서 관리하는 편이 더 안전합니다.

초보자 관점에서는 다음처럼 이해하면 됩니다.

```shell
개발 / 테스트 = Beanstalk가 RDS까지 같이 만들어도 편리함
운영          = RDS는 별도로 만들고 Beanstalk에는 연결 정보만 전달하는 편이 안전함
```

<br>

## 1. Elastic Beanstalk에서 RDS를 함께 생성하는 방식

Elastic Beanstalk는 애플리케이션 실행 환경을 만들면서 RDS도 함께 프로비저닝할 수 있습니다.

구조는 다음과 같습니다.

```shell
Elastic Beanstalk Environment
├── Load Balancer
├── EC2 Instance
└── RDS
```

이 방식은 처음 학습하거나 빠르게 테스트 환경을 만들 때 편리합니다.

애플리케이션 서버와 데이터베이스를 한 번에 만들 수 있기 때문입니다.

<br>

## 2. 개발 / 테스트 환경에서 편리한 이유

개발 환경이나 테스트 환경에서는 빠른 생성과 삭제가 중요합니다.

Beanstalk가 RDS를 함께 만들어주면 다음 장점이 있습니다.

- 콘솔에서 빠르게 전체 환경을 만들 수 있습니다.
- 애플리케이션과 데이터베이스 연결을 쉽게 구성할 수 있습니다.
- 테스트 후 Environment를 삭제하면서 관련 리소스를 함께 정리할 수 있습니다.
- 학습용이나 임시 테스트용 환경에 적합합니다.

예를 들어 기능 검증용 환경을 빠르게 만들고, 테스트가 끝나면 전체 환경을 삭제하는 경우에는 편리합니다.

```shell
테스트 Environment 생성
  ↓
EC2 + RDS 함께 생성
  ↓
기능 테스트
  ↓
Environment 삭제
```

<br>

## 3. 운영 환경에서 위험한 이유

운영 환경에서는 RDS를 Beanstalk Environment와 함께 생성하는 방식이 위험할 수 있습니다.

핵심 이유는 데이터베이스 생명주기가 Beanstalk Environment 생명주기와 묶일 수 있기 때문입니다.

```shell
Beanstalk Environment 삭제
  ↓
Environment에 묶인 RDS도 영향을 받을 수 있음
```

- 애플리케이션 서버는 새로 만들거나 교체할 수 있습니다.
- 하지만 운영 데이터베이스는 쉽게 삭제되거나 재생성되면 안 됩니다.
- 운영 DB에는 사용자 정보, 주문 정보, 결제 정보, 로그성 데이터 등 중요한 데이터가 들어갈 수 있습니다.
- 따라서 운영 환경에서는 RDS를 Beanstalk와 분리해서 관리하는 편이 안전합니다.

<br>

## 4. 운영 환경 권장 구조

운영 환경에서는 RDS를 Elastic Beanstalk 밖에서 별도로 생성하고, Beanstalk 애플리케이션에는
연결 문자열만 전달하는 방식이 권장됩니다.

```shell
RDS 별도 생성
  ↓
RDS Endpoint / Username / Password 준비
  ↓
Elastic Beanstalk 환경 변수로 전달
  ↓
애플리케이션이 외부 RDS에 연결
```

구조는 다음과 같습니다.

```shell
Elastic Beanstalk Environment
├── Load Balancer
└── EC2 Instance
      ↓
Separate RDS Database
```

이렇게 구성하면 Beanstalk Environment를 삭제하거나 새로 만들어도 RDS 생명주기를 별도로
보호할 수 있습니다.

<br>

## 5. 연결 문자열 전달 방식

별도로 만든 RDS에 연결하려면 애플리케이션에 DB 접속 정보를 전달해야 합니다.

일반적으로 환경 변수를 사용합니다.

예시는 다음과 같습니다.

```shell
DB_HOST=mydb.xxxxxx.ap-northeast-2.rds.amazonaws.com
DB_PORT=5432
DB_NAME=app
DB_USERNAME=app_user
DB_PASSWORD=secure_password
```

Spring Boot 애플리케이션이라면 환경 변수를 통해 datasource 설정을 주입할 수 있습니다.

```shell
SPRING_DATASOURCE_URL=jdbc:postgresql://mydb.xxxxxx.ap-northeast-2.rds.amazonaws.com:5432/app
SPRING_DATASOURCE_USERNAME=app_user
SPRING_DATASOURCE_PASSWORD=secure_password
```

운영 환경에서는 비밀번호 같은 민감 정보 관리에 주의해야 합니다.

가능하면 Secrets Manager, Parameter Store, IAM Role 기반 접근, 보안 그룹 제한을 함께 고려해야 합니다.

<br>

## 6. 이미 Beanstalk에 묶인 RDS를 분리하는 전략

이미 Elastic Beanstalk Environment와 함께 RDS를 만들었다면, 운영 안정성을 위해 RDS를 분리하는 절차를 고려할 수 있습니다.

권장 흐름은 다음과 같습니다.

```shell
1. RDS DB Snapshot을 생성합니다.
2. RDS 삭제 방지 설정을 활성화합니다.
3. RDS 없이 새 Elastic Beanstalk Environment를 생성합니다.
4. 새 Environment의 애플리케이션이 기존 RDS를 바라보도록 연결 정보를 설정합니다.
5. CNAME Swap 또는 Route 53 업데이트로 트래픽을 전환합니다.
6. 새 환경이 정상 동작하는지 확인합니다.
7. 기존 Environment를 종료합니다.
8. 남은 CloudFormation Stack 상태를 확인하고 정리합니다.
```

이 절차의 핵심은 애플리케이션 환경을 새로 만들되 데이터베이스는 유지하는 것입니다.

<br>

## 7. 단계별 설명

### 7.1 RDS Snapshot 생성

가장 먼저 RDS Snapshot을 생성합니다.

Snapshot은 장애나 실수에 대비한 안전장치입니다.

```shell
기존 RDS
  ↓
Snapshot 생성
```

운영 DB 작업 전에는 항상 복구 지점을 확보해야 합니다.

<br>

### 7.2 RDS 삭제 방지 설정

RDS 콘솔에서 deletion protection을 활성화합니다.

이 설정은 실수로 DB가 삭제되는 위험을 줄입니다.

```shell
RDS Deletion Protection = Enabled
```

<br>

### 7.3 RDS 없는 새 Beanstalk Environment 생성

새 Elastic Beanstalk Environment를 만들 때 RDS를 함께 생성하지 않습니다.

대신 기존 RDS의 endpoint를 애플리케이션 설정에 넣습니다.

```shell
New Beanstalk Environment
  ↓
Existing RDS Endpoint 사용
```

<br>

### 7.4 트래픽 전환

새 환경에서 애플리케이션이 기존 RDS에 정상 연결되는지 확인한 뒤 트래픽을 전환합니다.

대표적인 방법은 다음과 같습니다.

```shell
CNAME Swap
Route 53 업데이트
```

이 방식은 Blue / Green 배포와 비슷하게 운영 트래픽을 새 환경으로 옮기는 전략입니다.

<br>

### 7.5 기존 Environment 종료

새 환경이 정상 동작하면 기존 Beanstalk Environment를 종료합니다.

이때 RDS deletion protection을 설정해 두었기 때문에 RDS가 함께 삭제되는 위험을 줄일 수 있습니다.

기존 CloudFormation Stack이 `DELETE_FAILED` 상태로 남을 수 있으므로 이후 상태를 확인하고 정리해야 합니다.

<br>

## 8. 운영 환경에서 반드시 확인할 사항

RDS를 Beanstalk와 분리할 때는 다음을 확인해야 합니다.

- RDS Snapshot이 생성되었는지 확인합니다.
- RDS deletion protection이 활성화되었는지 확인합니다.
- 새 Beanstalk Environment가 RDS 없이 생성되었는지 확인합니다.
- 애플리케이션 환경 변수에 RDS 연결 정보가 올바르게 들어갔는지 확인합니다.
- RDS 보안 그룹이 새 Beanstalk EC2 보안 그룹의 접근을 허용하는지 확인합니다.
- 새 환경에서 DB 읽기/쓰기 동작이 정상인지 확인합니다.
- Route 53 또는 CNAME 전환 후 롤백 방법을 준비합니다.
- 기존 Environment 종료 후 CloudFormation Stack 상태를 확인합니다.

<br>

## 9. 보안 그룹 관점

RDS를 별도로 만들면 네트워크 접근 제어를 직접 확인해야 합니다.

일반적으로 RDS 보안 그룹은 Beanstalk EC2 인스턴스의 보안 그룹에서 오는 DB 포트 접근만 허용하는 편이 안전합니다.

예를 들어 PostgreSQL이면 다음처럼 제한합니다.

```shell
RDS Security Group Inbound
Source: Beanstalk EC2 Security Group
Port: 5432
```

MySQL이면 다음처럼 설정할 수 있습니다.

```shell
RDS Security Group Inbound
Source: Beanstalk EC2 Security Group
Port: 3306
```

운영 DB를 `0.0.0.0/0`에 열어두는 방식은 피해야 합니다.

<br>

## 10. DVA-C02 시험 포인트

- Elastic Beanstalk는 RDS를 함께 프로비저닝할 수 있습니다.
- RDS를 Beanstalk와 함께 만드는 방식은 개발 / 테스트 환경에 편리합니다.
- 운영 환경에서는 RDS 생명주기가 Beanstalk Environment와 묶이는 점이 위험합니다.
- 운영 환경에서는 RDS를 별도로 생성하고 Beanstalk 애플리케이션에 connection string을 제공하는 방식이 권장됩니다.
- 기존 Beanstalk RDS를 분리하기 전 RDS Snapshot을 생성해야 합니다.
- RDS deletion protection을 활성화해 삭제 위험을 줄여야 합니다.
- RDS 없는 새 Beanstalk Environment를 만들고 기존 RDS에 연결해야 합니다.
- CNAME Swap 또는 Route 53 업데이트로 트래픽을 새 환경으로 전환할 수 있습니다.
- 기존 Environment 종료 후 CloudFormation Stack이 `DELETE_FAILED` 상태로 남을 수 있습니다.

<br>

## 11. 시험 예상 문제

### 문제 1

운영 환경에서 Elastic Beanstalk 애플리케이션과 RDS를 구성하려고 합니다. 가장 권장되는 방식은 무엇입니까?

- A. RDS를 Beanstalk Environment와 함께 생성하고 운영 데이터도 함께 저장합니다.
- B. RDS를 별도로 생성하고 Beanstalk 애플리케이션에는 connection string을 제공합니다.
- C. RDS 대신 항상 S3만 사용합니다.
- D. 운영 환경에서는 데이터베이스 연결을 사용하지 않습니다.

<br>

### 정답

B. RDS를 별도로 생성하고 Beanstalk 애플리케이션에는 connection string을 제공합니다.

<br>

### 해설

운영 환경에서는 RDS 생명주기를 Beanstalk Environment와 분리하는 편이 안전합니다.
Beanstalk Environment를 삭제하거나 재생성하더라도 운영 데이터베이스가 영향을 받지 않도록 RDS는
별도 리소스로 관리하고, 애플리케이션에는 연결 정보를 전달하는 방식이 권장됩니다.

<br>

### 문제 2

Elastic Beanstalk에서 RDS를 함께 생성하는 방식이 개발 / 테스트 환경에 적합한 이유로 가장 올바른 것은 무엇입니까?

- A. 빠르게 애플리케이션과 데이터베이스를 함께 만들고 테스트할 수 있기 때문입니다.
- B. 운영 DB의 삭제 위험이 완전히 사라지기 때문입니다.
- C. Beanstalk와 함께 만든 RDS는 비용이 전혀 발생하지 않기 때문입니다.
- D. RDS가 자동으로 여러 AWS 계정에 복제되기 때문입니다.

<br>

### 정답

A. 빠르게 애플리케이션과 데이터베이스를 함께 만들고 테스트할 수 있기 때문입니다.

<br>

### 해설

개발 / 테스트 환경에서는 빠른 생성과 삭제가 중요합니다.
Beanstalk가 RDS를 함께 만들면 애플리케이션과 데이터베이스를 빠르게 구성해 테스트할 수 있습니다.
하지만 운영 환경에서는 RDS 생명주기가 Environment와 묶일 수 있어 위험합니다.

<br>

### 문제 3

이미 Elastic Beanstalk Environment와 함께 생성된 RDS를 운영 환경에서 분리하려고 합니다.
가장 먼저 수행해야 할 안전 조치는 무엇입니까?

- A. RDS DB Snapshot을 생성합니다.
- B. Route 53 Hosted Zone을 삭제합니다.
- C. 기존 Environment를 즉시 삭제합니다.
- D. Auto Scaling Group 최소 용량을 0으로 변경합니다.

<br>

### 정답

A. RDS DB Snapshot을 생성합니다.

<br>

### 해설

RDS 분리 작업 전에는 데이터 보호를 위해 Snapshot을 먼저 생성해야 합니다.
Snapshot은 작업 중 실수나 장애가 발생했을 때 복구 지점 역할을 합니다.
운영 DB 작업에서 백업 없이 바로 Environment를 삭제하는 방식은 위험합니다.

<br>

### 문제 4

RDS를 분리한 새 Elastic Beanstalk Environment로 트래픽을 전환하는 대표 방법은 무엇입니까?

- A. CNAME Swap 또는 Route 53 업데이트
- B. IAM User 삭제
- C. S3 Bucket Versioning 비활성화
- D. CloudWatch Logs 삭제

<br>

### 정답

A. CNAME Swap 또는 Route 53 업데이트

<br>

### 해설

RDS 없이 새 Beanstalk Environment를 만들고 기존 RDS에 정상 연결되는지 확인한 뒤,
CNAME Swap 또는 Route 53 업데이트로 사용자 트래픽을 새 환경으로 전환할 수 있습니다.
이 방식은 Blue / Green 배포와 유사하게 기존 환경에서 새 환경으로 안전하게 이동하는 전략입니다.

<br>

## 요약

- Elastic Beanstalk에서 RDS를 함께 생성하는 방식은 개발 / 테스트 환경에서는 편리합니다.
- 하지만 운영 환경에서는 RDS 생명주기가 Beanstalk Environment와 묶일 수 있어 위험합니다.
- 운영에서는 RDS를 별도로 생성하고, Beanstalk 애플리케이션에는 connection string을 전달하는
  방식이 더 안전합니다.
- 이미 Beanstalk와 묶인 RDS를 분리할 때는 Snapshot 생성, deletion protection 활성화,
  RDS 없는 새 Environment 생성, 기존 RDS 연결, CNAME Swap 또는 Route 53 업데이트 순서로 진행해야 합니다.
