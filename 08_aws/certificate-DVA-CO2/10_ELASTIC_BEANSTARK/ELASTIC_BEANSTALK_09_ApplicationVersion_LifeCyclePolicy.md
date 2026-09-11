# 🚀 Elastic Beanstalk Application Version Lifecycle Policy

Elastic Beanstalk의 Application Version Lifecycle Policy는 오래된 애플리케이션 버전을
자동으로 정리하기 위한 정책입니다.

초보자 관점에서는 다음처럼 이해하면 됩니다.

```shell
Application Version Lifecycle Policy
= 오래된 배포 버전을 계속 쌓아두지 않고 자동으로 정리하는 규칙
```

Elastic Beanstalk는 애플리케이션 코드를 배포할 때마다 Application Version을 만듭니다.

배포를 많이 반복하면 이전 버전들이 계속 쌓입니다.

이전 버전을 정리하지 않으면 나중에 새 버전을 배포하지 못할 수 있습니다.

<br>

## 1. Application Version이란 무엇입니까?

Application Version은 Elastic Beanstalk에 배포할 수 있는 애플리케이션 코드의 특정 버전입니다.

예를 들어 쇼핑몰 애플리케이션을 여러 번 배포하면 다음처럼 버전이 쌓일 수 있습니다.

```shell
shopping-app
├── v1
├── v2
├── v3
├── v4
└── v5
```

각 버전은 특정 시점의 코드 묶음입니다.

예를 들어 다음처럼 구분할 수 있습니다.

```shell
v1 = 최초 배포
v2 = 장바구니 기능 추가
v3 = 결제 오류 수정
v4 = 주문 조회 API 개선
v5 = 쿠폰 기능 추가
```

Elastic Beanstalk는 이런 버전을 저장해 두고, Environment에 특정 Application Version을
배포합니다.

<br>

## 2. 왜 Lifecycle Policy가 필요합니까?

Elastic Beanstalk는 최대 1000개의 Application Version을 저장할 수 있습니다.

이 제한을 넘으면 더 이상 새 버전을 배포할 수 없습니다.

```shell
Application Version 최대 개수 = 1000개
```

예를 들어 CI/CD 파이프라인에서 하루에 여러 번 자동 배포를 수행하면 Application Version이
빠르게 쌓일 수 있습니다.

```shell
1일 10회 배포
  ↓
100일 후 약 1000개 버전
  ↓
새 버전 배포 실패 가능
```

그래서 오래된 Application Version을 주기적으로 정리해야 합니다.

Lifecycle Policy는 이 정리를 자동화해 줍니다.

<br>

## 3. Lifecycle Policy가 정리하는 대상

Lifecycle Policy는 오래된 Application Version을 정리합니다.

여기서 중요한 점은 Application Version과 실제 실행 중인 Environment를 구분해야 하는 점입니다.

```shell
Application Version = 배포 가능한 코드 버전
Environment         = 실제 AWS 리소스에서 실행 중인 환경
```

현재 Environment에서 사용 중인 Application Version은 삭제되지 않습니다.

예를 들어 운영 환경이 `v10`을 실행 중이라면 `v10`은 Lifecycle Policy 대상에서 보호됩니다.

```shell
prod environment -> v10 실행 중

Lifecycle Policy 실행
  ↓
v1 ~ v9 중 오래된 버전 정리 가능
v10은 사용 중이므로 삭제되지 않음
```

<br>

## 4. 시간 기준 정리

Lifecycle Policy는 시간 기준으로 오래된 Application Version을 삭제할 수 있습니다.

예를 들어 30일보다 오래된 버전을 정리하는 방식입니다.

```shell
30일 초과 Application Version 삭제
```

이 방식은 배포가 자주 발생하지만 오래된 버전을 장기간 보관할 필요가 없을 때 적합합니다.

예시:

```shell
v1  = 90일 전 배포
v2  = 60일 전 배포
v3  = 10일 전 배포
v4  = 현재 운영 버전

정책: 30일보다 오래된 버전 정리

결과:
v1, v2 삭제 대상
v3 유지
v4 사용 중이므로 유지
```

<br>

## 5. 개수 기준 정리

Lifecycle Policy는 개수 기준으로 오래된 Application Version을 삭제할 수도 있습니다.

예를 들어 최근 200개 버전만 유지하는 방식입니다.

```shell
최근 200개 Application Version 유지
나머지 오래된 Version 정리
```

이 방식은 "얼마나 오래됐는지"보다 "몇 개까지 보관할지"가 더 중요한 경우에 적합합니다.

예시:

```shell
Application Version 총 900개
정책: 최근 200개만 유지

결과:
최근 200개 유지
나머지 오래된 버전 삭제 대상
```

<br>

## 6. 사용 중인 버전은 삭제되지 않습니다

Lifecycle Policy에서 가장 중요한 안전장치입니다.

현재 Environment에서 사용 중인 Application Version은 삭제되지 않습니다.

```shell
dev  -> v30 실행 중
test -> v28 실행 중
prod -> v25 실행 중

Lifecycle Policy 실행
  ↓
v25, v28, v30은 삭제되지 않음
```

이 규칙 덕분에 현재 실행 중인 환경이 갑자기 깨지는 상황을 방지할 수 있습니다.

하지만 사용 중이 아닌 오래된 버전은 정책에 따라 삭제될 수 있습니다.

<br>

## 7. S3 Source Bundle 삭제 옵션

Application Version은 코드 묶음과 연결됩니다.

이 코드 묶음은 S3 source bundle 형태로 저장될 수 있습니다.

Lifecycle Policy를 설정할 때 source bundle을 S3에서 삭제하지 않는 옵션을 선택할 수 있습니다.

```shell
Application Version 메타데이터 삭제
S3 source bundle은 보존
```

이 옵션은 데이터 손실을 방지할 때 유용합니다.

예를 들어 Application Version 목록에서는 오래된 버전을 정리하되, 실제 업로드된 소스 번들은
S3에 남겨 둘 수 있습니다.

운영 환경에서는 감사, 장애 분석, 과거 릴리스 추적이 필요할 수 있으므로 source bundle 삭제 여부를
신중하게 선택해야 합니다.

<br>

## 8. Lifecycle Policy 예시

### 8.1 개발 환경

개발 환경은 배포가 자주 발생하고 오래된 버전 보관 가치가 낮을 수 있습니다.

```shell
환경: dev
정책: 최근 50개 버전만 유지
목적: 불필요한 버전 누적 방지
```

<br>

### 8.2 운영 환경

운영 환경은 롤백과 추적이 중요합니다.

```shell
환경: prod
정책: 최근 200개 버전 유지
S3 source bundle은 삭제하지 않음
목적: 롤백 가능성 및 감사 추적 보존
```

운영에서는 단순히 오래된 버전을 많이 지우는 것보다, 롤백 가능성과 보관 정책을 함께 고려해야 합니다.

<br>

## 9. 운영 관점에서의 주의사항

Lifecycle Policy는 자동 정리 기능이므로 운영 정책과 맞게 설정해야 합니다.

주의할 점은 다음과 같습니다.

- 너무 공격적으로 삭제하면 필요한 과거 버전을 찾기 어려울 수 있습니다.
- 롤백 전략과 버전 보관 정책을 함께 설계해야 합니다.
- source bundle 삭제 여부는 데이터 보존 정책과 맞춰야 합니다.
- CI/CD 배포가 잦은 프로젝트는 1000개 제한에 빨리 도달할 수 있습니다.
- 사용 중인 버전은 삭제되지 않지만, 사용 중이 아닌 롤백 후보 버전은 삭제될 수 있습니다.

<br>

## 10. DVA-C02 시험 포인트

- Elastic Beanstalk는 최대 1000개의 Application Version을 저장할 수 있습니다.
- 오래된 Application Version을 제거하지 않으면 새 배포가 막힐 수 있습니다.
- Lifecycle Policy는 오래된 Application Version을 자동으로 정리하는 정책입니다.
- Lifecycle Policy는 시간 기준으로 오래된 버전을 정리할 수 있습니다.
- Lifecycle Policy는 개수 기준으로 오래된 버전을 정리할 수 있습니다.
- 현재 Environment에서 사용 중인 Application Version은 삭제되지 않습니다.
- source bundle을 S3에서 삭제하지 않는 옵션을 선택할 수 있습니다.
- source bundle 보존 옵션은 데이터 손실 방지에 도움이 됩니다.

<br>

## 11. 시험 예상 문제

### 문제 1

Elastic Beanstalk에서 Application Version이 계속 쌓여 최대 저장 개수 제한에 가까워지고
있습니다. 오래된 버전을 자동으로 정리하기 위해 사용해야 하는 기능은 무엇입니까?

- A. Lifecycle Policy
- B. Security Group
- C. Route Table
- D. Placement Group

<br>

### 정답

A. Lifecycle Policy

<br>

### 해설

Lifecycle Policy는 오래된 Elastic Beanstalk Application Version을 자동으로 정리하기 위한
기능입니다. Elastic Beanstalk는 최대 1000개의 Application Version을 저장할 수 있습니다.
오래된 버전을 정리하지 않으면 새 배포가 막힐 수 있으므로 Lifecycle Policy를 사용해야 합니다.

<br>

### 문제 2

Elastic Beanstalk Application Version Lifecycle Policy에 대한 설명으로 가장 올바른 것은
무엇입니까?

- A. 현재 Environment에서 사용 중인 Application Version도 항상 삭제됩니다.
- B. 오래된 Application Version을 시간 기준 또는 개수 기준으로 정리할 수 있습니다.
- C. Lifecycle Policy는 EC2 인스턴스 타입을 자동으로 변경하는 기능입니다.
- D. Lifecycle Policy는 Load Balancer 타입을 변경하는 기능입니다.

<br>

### 정답

B. 오래된 Application Version을 시간 기준 또는 개수 기준으로 정리할 수 있습니다.

<br>

### 해설

Lifecycle Policy는 오래된 Application Version을 정리하기 위한 정책입니다.
정리 기준은 시간 기반 또는 개수 기반으로 설정할 수 있습니다.
현재 Environment에서 사용 중인 Application Version은 삭제되지 않습니다.

<br>

### 문제 3

Elastic Beanstalk에서 현재 운영 Environment가 사용 중인 Application Version에 대해
Lifecycle Policy가 실행되었습니다. 어떤 동작이 맞습니까?

- A. 사용 중인 Application Version은 삭제되지 않습니다.
- B. 사용 중인 Application Version은 즉시 삭제됩니다.
- C. Environment가 자동으로 종료됩니다.
- D. RDS 데이터베이스가 자동으로 삭제됩니다.

<br>

### 정답

A. 사용 중인 Application Version은 삭제되지 않습니다.

<br>

### 해설

Lifecycle Policy는 오래된 Application Version을 정리하지만, 현재 Environment에서
사용 중인 버전은 삭제하지 않습니다.
이 안전장치 덕분에 실행 중인 환경이 갑자기 깨지는 상황을 방지할 수 있습니다.

<br>

### 문제 4

Elastic Beanstalk에서 오래된 Application Version은 정리하되, 실제 source bundle을
S3에 남겨 데이터 손실을 방지하고 싶습니다. 어떤 설정을 고려해야 합니까?

- A. source bundle을 S3에서 삭제하지 않는 옵션
- B. 모든 Environment 강제 종료
- C. Load Balancer 삭제
- D. Auto Scaling Group 최소 용량 0 설정

<br>

### 정답

A. source bundle을 S3에서 삭제하지 않는 옵션

<br>

### 해설

Lifecycle Policy에는 source bundle을 S3에서 삭제하지 않는 옵션이 있습니다.
이 옵션을 사용하면 Application Version 목록에서는 오래된 버전을 정리하면서도, 실제 소스 번들은
S3에 보존할 수 있습니다.
운영 환경에서 감사, 장애 분석, 과거 릴리스 추적이 필요한 경우 유용합니다.

<br>

## 요약

- Application Version Lifecycle Policy는 Elastic Beanstalk의 오래된 애플리케이션 버전을
  자동으로 정리하는 정책입니다.
- Elastic Beanstalk는 최대 1000개의 Application Version을 저장할 수 있으므로, 배포가 잦은
  프로젝트에서는 반드시 정리 정책을 고려해야 합니다.
- DVA-C02에서는 `최대 1000개`, `시간 기준 정리`, `개수 기준 정리`,
  `사용 중인 버전은 삭제되지 않음`, `S3 source bundle 보존 옵션`을 핵심으로 기억하면 됩니다.
