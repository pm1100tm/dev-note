# 🚀 Sticky Sessions (Session Affinity)

Sticky sessions (세션 고정)

## 📌 정의

- 로드 벨런서 뒤에 여러 개의 인스턴스가 있을 때, 같은 클라이언트가 항상 같은 인스턴스로 연결되도록
  하는 기능
- 즉, 사용자가 요청할 때마다 서버가 바뀌지 않고 동일한 서버에서 세션을 유지

## 📌 적용 가능 로드 벨런서

- CLB
- ALB
- NLB

### 동작 방식

- CLB & ALB -> 쿠키(cookie)를 사용
- 로드 벨런서가 생성하는 전용 쿠키 또는 애플리케이션 쿠키 사용
- 쿠키에는 만료 시간을 설정할 수 있음
- 클라이언트는 이후 요청 시 쿠키를 포함해서 보내고, 로드 벨런서는 해당 쿠키 값을 기반으로
  동일 인스턴스로 트래픽을 라우팅

## 🔹 사용 사례

- 웹 애플리케이션에서 세션 데이터가 서버 로컬에 저장되는 경우
- 예: 로그인 상태, 장바구니, 사용자 진행 상황 등이 특정 서버에 저장됨
- 세션이 다른 서버로 분선되면 데이터 손실/로그아웃 문제 발생 -> Skicky Session 필요

### 단점

- 특정 사용자가 몰리면 일부 서버만 과부하가 걸리고, 다른 서버는 한가해지는 부하 불균형(Imbalance)
  문제가 생길 수 있음
- 따라서 확장성과 부하 분산 측면에서는 불리

✅ 쉽게 요약

- Sticky Session = 특정 사용자를 항상 같은 서버로 보내는 기능
- 장점: 사용자 경험 유지 (세션 데이터 보존)
- 단점: 부하 분산이 고르지 않게 됨

---

## 📌 Sticky Sessions – 쿠키 이름과 종류

### 🔹 1. Application-based Cookies (애플리케이션 기반 쿠키)

- [1.] Custom Cookie(사용자 정의 쿠키)
  - 애플리케이션(Target)에서 직접 생성
  - 필요한 속성(예: 로그인 정보, 사용자 ID 등)을 자유롭게 포함 가능
  - Target Group별로 개별적으로 쿠키 이름을 지정해야 함
  - 단, AWS 예약 쿠키 이름(AWSALB, AWSALBAPP, AWSALBTG)은 사용 불가
- [2.] Application Cookie (로드 밸런서 생성)
  - 로드 밸런서가 생성
  - 쿠키 이름은 고정: AWSALBAPP

### 🔹 2. Duration-based Cookies (기간 기반 쿠키)

- 로드 밸런서가 생성하는 쿠키
- 지정된 기간 동안 세션을 같은 인스턴스에 고정
- 지속 시간은 로드 벨런서 자체에서 생성됨
- 쿠키 이름:
  - ALB → AWSALB
  - CLB → AWSELB

### 🔹 3. Skicky Session 활성화 방법

- AWS에서 스티키 세션(Sticky Session)을 활성화하려면 주로
  **Elastic Load Balancer(ELB)**에서 설정 진행
- 타겟 그룹 -> Edit attribute -> Target selection configuration
  - Turn on skickiness 체크박스 체크
    - 로드 벨런서 생성 쿠키 / 애플리케이션 기반 쿠키 선택 가능

⸻

✅ 쉽게 요약

- Custom Cookie → 애플리케이션이 직접 만듦 (쿠키 이름 자유 설정)
- Application Cookie → ALB가 생성 (AWSALBAPP)
- Duration Cookie → 로드 밸런서가 기간 기반으로 생성
- ALB = AWSALB
- CLB = AWSELB
