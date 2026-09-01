# 🚀 Elastic Beanstalk 환경 복제와 Load Balancer 마이그레이션

Elastic Beanstalk에서는 기존 Environment와 같은 설정을 가진 새 Environment를 만들 수 있습니다.

이 기능을 환경 복제, 즉 Cloning이라고 부릅니다.

또한 Elastic Beanstalk Environment를 만든 뒤에는 Load Balancer 타입을 직접 변경할 수 없으므로, Load Balancer 타입을 바꾸려면 별도 마이그레이션 절차가 필요합니다.

초보자 관점에서는 다음처럼 이해하면 됩니다.

```shell
환경 복제 = 기존 환경 설정을 복사해서 새 환경을 만드는 기능
LB 마이그레이션 = 기존 환경의 Load Balancer 타입을 직접 바꾸지 않고 새 환경으로 옮기는 작업
```

<br>

## 1. 환경 복제란 무엇입니까?

환경 복제는 기존 Elastic Beanstalk Environment와 동일한 설정을 가진 새 Environment를 만드는 기능입니다.

예를 들어 운영 환경과 거의 같은 테스트 환경을 만들고 싶을 때 유용합니다.

```shell
prod environment
  ↓ clone
test environment
```

복제된 환경은 기존 환경과 같은 설정을 기반으로 만들어지지만, 복제 이후에는 필요한 설정을 변경할 수 있습니다.

<br>

## 2. 환경 복제가 유용한 상황

환경 복제는 다음 상황에서 유용합니다.

- 운영 환경과 같은 설정의 테스트 환경이 필요할 때
- 새 애플리케이션 버전을 운영과 유사한 환경에서 검증하고 싶을 때
- 기존 환경 설정을 반복해서 수동 입력하고 싶지 않을 때
- Blue / Green 배포와 유사한 검증 환경이 필요할 때
- 장애 재현용 환경을 빠르게 만들고 싶을 때

예를 들어 운영 환경에 새 버전을 바로 배포하기 전에 복제 환경에서 먼저 확인할 수 있습니다.

```shell
prod 환경 clone
  ↓
stage 환경 생성
  ↓
새 버전 배포
  ↓
검증 후 운영 반영
```

<br>

## 3. 환경 복제 시 보존되는 설정

환경을 복제하면 많은 리소스와 설정이 보존됩니다.

대표적으로 다음 항목이 보존됩니다.

| 항목                  | 설명                                                 |
| --------------------- | ---------------------------------------------------- |
| Load Balancer 타입    | 기존 환경의 Load Balancer 타입이 유지됩니다.         |
| Load Balancer 설정    | 리스너, 헬스 체크 등 관련 설정이 유지될 수 있습니다. |
| RDS 데이터베이스 타입 | RDS 유형 설정은 보존될 수 있습니다.                  |
| 환경 변수             | 애플리케이션 환경 변수가 유지됩니다.                 |
| Beanstalk 환경 설정   | 인스턴스, 스케일링 등 주요 설정이 유지됩니다.        |

단, RDS 데이터 자체는 복제되지 않습니다.

이 점은 시험과 실무 모두에서 중요합니다.

```shell
RDS database type = 보존 가능
RDS data          = 보존되지 않음
```

<br>

## 4. 환경 복제 후 변경 가능한 부분

환경을 복제한 뒤에는 필요한 설정을 변경할 수 있습니다.

예를 들어 다음을 조정할 수 있습니다.

- 환경 이름
- 환경 변수
- 인스턴스 타입
- Auto Scaling 설정
- 애플리케이션 버전
- 일부 Load Balancer 설정

복제는 "완전히 고정된 복사본"이 아니라, 기존 설정을 기반으로 새 환경을 만드는 출발점으로 이해하면 됩니다.

<br>

## 5. Load Balancer 타입 변경 제한

Elastic Beanstalk Environment를 생성한 뒤에는 Elastic Load Balancer 타입을 변경할 수 없습니다.

중요한 점은 "타입은 변경할 수 없지만 설정은 일부 변경할 수 있다"는 것입니다.

예를 들어 Classic Load Balancer를 사용하는 환경을 만든 뒤, 같은 Environment 안에서 Application Load Balancer로 직접 바꾸는 방식은 사용할 수 없습니다.

```shell
기존 Environment
CLB 사용 중
  ↓
같은 Environment에서 ALB로 직접 변경 불가
```

이 제한 때문에 Load Balancer 타입을 바꾸려면 새 Environment를 만들어야 합니다.

<br>

## 6. 왜 Clone만으로 LB 타입 변경이 어렵습니까?

환경 복제는 기존 설정을 그대로 복사하는 기능입니다.

따라서 기존 환경이 Classic Load Balancer를 사용 중이면, 복제한 환경도 같은 Load Balancer 타입을 유지하려고 합니다.

하지만 목표가 CLB에서 ALB로 바꾸는 것이라면, 기존 환경을 그대로 clone하는 방식은 적합하지 않습니다.

```shell
목표: CLB -> ALB 변경

clone 사용
  ↓
기존 CLB 타입까지 복사될 수 있음
  ↓
목표와 맞지 않음
```

그래서 Load Balancer 타입 변경 마이그레이션에서는 기존 설정을 참고하되, 새 Environment를 별도로 생성해야 합니다.

<br>

## 7. Load Balancer 마이그레이션 절차

Load Balancer 타입을 변경하려면 다음 흐름을 사용합니다.

```shell
1. 기존 환경과 거의 같은 설정으로 새 Environment를 생성합니다.
2. 단, 새 Environment는 원하는 Load Balancer 타입으로 생성합니다.
3. 새 Environment에 애플리케이션을 배포합니다.
4. 새 Environment를 검증합니다.
5. CNAME Swap 또는 Route 53 업데이트로 트래픽을 전환합니다.
6. 기존 Environment를 정리합니다.
```

예를 들어 CLB에서 ALB로 바꾸는 경우는 다음과 같습니다.

```shell
Old Beanstalk Environment
  └── Classic Load Balancer

New Beanstalk Environment
  └── Application Load Balancer

검증 완료
  ↓
CNAME Swap 또는 Route 53 변경
  ↓
트래픽을 New Environment로 전환
```

<br>

## 8. CNAME Swap과 Route 53 업데이트

새 Environment를 만들고 검증한 뒤에는 사용자 트래픽을 새 환경으로 옮겨야 합니다.

이때 대표적인 방법은 두 가지입니다.

```shell
CNAME Swap
Route 53 업데이트
```

CNAME Swap은 Elastic Beanstalk 환경 URL을 서로 바꾸는 방식입니다.

Route 53 업데이트는 DNS 레코드를 새 환경의 엔드포인트로 변경하는 방식입니다.

운영 환경에서는 DNS TTL, 헬스 체크, 롤백 계획을 함께 고려해야 합니다.

<br>

## 9. 운영 환경에서의 주의사항

Load Balancer 마이그레이션은 사용자 트래픽 경로를 바꾸는 작업입니다.

따라서 운영 환경에서는 다음을 확인해야 합니다.

- 새 Environment가 정상적으로 배포되었는지 확인합니다.
- 헬스 체크가 정상인지 확인합니다.
- 보안 그룹과 인바운드 규칙이 올바른지 확인합니다.
- HTTPS 인증서와 리스너 설정을 확인합니다.
- 환경 변수가 누락되지 않았는지 확인합니다.
- Route 53 또는 CNAME 전환 후 롤백 방법을 준비합니다.
- 기존 Environment 삭제 전에 새 환경에서 실제 트래픽 처리가 정상인지 확인합니다.

Load Balancer 타입 변경은 단순 설정 변경이 아니라 새 환경으로 이동하는 마이그레이션 작업으로 봐야 합니다.

<br>

## 10. DVA-C02 시험 포인트

- Elastic Beanstalk는 기존 Environment를 clone할 수 있습니다.
- 환경 복제는 기존 환경과 같은 설정의 새 환경을 만들 때 유용합니다.
- 환경 복제는 테스트 환경 생성에 유용합니다.
- 환경 복제 시 Load Balancer 타입과 설정, 환경 변수 등이 보존될 수 있습니다.
- RDS 데이터베이스 타입은 보존될 수 있지만 RDS 데이터 자체는 보존되지 않습니다.
- Elastic Beanstalk Environment 생성 후 Load Balancer 타입은 변경할 수 없습니다.
- Load Balancer 타입 변경이 필요하면 새 Environment를 만들어야 합니다.
- 새 Environment에 애플리케이션을 배포한 뒤 CNAME Swap 또는 Route 53 업데이트로 트래픽을 전환합니다.
- Load Balancer 타입 변경 목적이면 단순 clone이 적합하지 않을 수 있습니다.

<br>

## 11. 시험 예상 문제

### 문제 1

Elastic Beanstalk에서 운영 환경과 동일한 설정을 가진 테스트 환경을 빠르게 만들고 싶습니다. 가장 적절한 기능은 무엇입니까?

- A. Environment Cloning
- B. S3 Versioning
- C. IAM Access Analyzer
- D. Route Table Propagation

<br>

### 정답

A. Environment Cloning

<br>

### 해설

Environment Cloning은 기존 Elastic Beanstalk Environment와 동일한 설정을 가진 새 Environment를 만드는 기능입니다.

운영 환경과 유사한 테스트 환경을 만들 때 유용합니다.

복제 후 필요한 설정을 변경할 수도 있습니다.

<br>

### 문제 2

Elastic Beanstalk 환경을 생성한 뒤 Classic Load Balancer에서 Application Load Balancer로 타입을 변경하려고 합니다. 가장 적절한 방법은 무엇입니까?

- A. 기존 Environment에서 Load Balancer 타입만 직접 변경합니다.
- B. 새 Environment를 원하는 Load Balancer 타입으로 생성하고 애플리케이션을 배포한 뒤 CNAME Swap 또는 Route 53 업데이트를 수행합니다.
- C. 기존 EC2 인스턴스에 SSH로 접속해 Load Balancer 타입을 수정합니다.
- D. Security Group 이름을 변경하면 Load Balancer 타입이 자동 변경됩니다.

<br>

### 정답

B. 새 Environment를 원하는 Load Balancer 타입으로 생성하고 애플리케이션을 배포한 뒤 CNAME Swap 또는 Route 53 업데이트를 수행합니다.

<br>

### 해설

Elastic Beanstalk Environment 생성 후에는 Load Balancer 타입을 직접 변경할 수 없습니다.

따라서 원하는 Load Balancer 타입으로 새 Environment를 생성하고, 애플리케이션을 배포한 뒤 트래픽을 새 환경으로 전환해야 합니다.

전환 방법으로 CNAME Swap 또는 Route 53 업데이트를 사용할 수 있습니다.

<br>

### 문제 3

Elastic Beanstalk Environment를 clone할 때 RDS와 관련해 가장 올바른 설명은 무엇입니까?

- A. RDS 데이터베이스 타입은 보존될 수 있지만 데이터 자체는 보존되지 않습니다.
- B. RDS 데이터와 모든 트랜잭션 로그가 항상 자동 복제됩니다.
- C. RDS는 clone된 Environment에서 절대 사용할 수 없습니다.
- D. RDS 데이터가 S3 버킷으로 자동 변환됩니다.

<br>

### 정답

A. RDS 데이터베이스 타입은 보존될 수 있지만 데이터 자체는 보존되지 않습니다.

<br>

### 해설

환경 복제 시 RDS 데이터베이스 타입 같은 설정은 보존될 수 있습니다.

하지만 실제 데이터베이스 데이터 자체가 그대로 복제되는 것은 아닙니다.

운영 데이터가 필요한 테스트 환경이라면 별도 스냅샷 복원이나 데이터 복제 전략을 고려해야 합니다.

<br>

### 문제 4

Load Balancer 타입 변경을 위해 새 Elastic Beanstalk Environment를 만들었습니다. 새 환경 검증 후 사용자 트래픽을 전환하는 대표 방법은 무엇입니까?

- A. CNAME Swap 또는 Route 53 업데이트
- B. IAM Password Policy 변경
- C. S3 Multipart Upload
- D. EBS Snapshot 삭제

<br>

### 정답

A. CNAME Swap 또는 Route 53 업데이트

<br>

### 해설

새 Environment를 만들고 애플리케이션 검증까지 완료한 뒤에는 사용자 트래픽을 새 환경으로 옮겨야 합니다.

Elastic Beanstalk에서는 CNAME Swap을 사용할 수 있고, DNS를 Route 53에서 관리할 경우 레코드 업데이트로 트래픽을 전환할 수 있습니다.

<br>

## 요약

Elastic Beanstalk 환경 복제는 기존 Environment와 같은 설정을 가진 새 Environment를 만드는 기능입니다.

테스트 환경 생성이나 새 버전 검증에 유용하지만, RDS 데이터 자체까지 자동 복제되는 것은 아닙니다.

Load Balancer 타입은 Environment 생성 후 직접 변경할 수 없으므로, 새 Environment를 만들고 애플리케이션을 배포한 뒤 CNAME Swap 또는 Route 53 업데이트로 트래픽을 전환해야 합니다.
