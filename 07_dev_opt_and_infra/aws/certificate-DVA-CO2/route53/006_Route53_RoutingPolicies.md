# 🚀 Route53 Routing Policies

- DNS 레벨에서의 트래픽 분산 전략
- Routing Policy는 Route 53이 DNS 쿼리에 응답할 때 “어떤 IP 또는 리소스를 반환할지” 결정하는 규칙

📌 주의:

- 여기서 “Routing”은 Load Balancer의 트래픽 라우팅이 아님
- DNS는 트래픽을 직접 전달하지 않고, 클라이언트에게 IP만 알려줌

<br>

## ⚙️ Route 53에서 지원하는 Routing Policy 종류

| 정책 유형                            | 설명                                        | 주요 목적              |
| ------------------------------------ | ------------------------------------------- | ---------------------- |
| **Simple**                           | 하나의 리소스로 트래픽 라우팅               | 단일 리소스 (기본형)   |
| **Weighted**                         | 트래픽 비율을 가중치로 분배                 | 테스트, Canary 배포    |
| **Failover**                         | 기본 리소스 장애 시 예비 리소스로 전환      | DR (Disaster Recovery) |
| **Latency-based**                    | 지연시간이 가장 낮은 리전으로 연결          | 글로벌 사용자 대응     |
| **Geolocation**                      | 사용자 지리적 위치 기반 분배                | 지역별 서비스 차별화   |
| **Multi-Value Answer**               | 여러 리소스를 동시에 반환 (간단 로드밸런싱) | 헬스체크 기반 분산     |
| **Geoproximity (Traffic Flow 전용)** | 지리적 거리 + 가중치 기반 분배              | 고급 트래픽 최적화     |

<br>

## ⚙️ Simple Routing Policy (단순 라우팅)

- DNS 이름에 대해 하나의 리소스를 반환하는 가장 기본적인 정책입니다.
- 예: api.example.com → 54.80.100.12

<br>

🧱 특징

| 항목                          | 설명                                                         |
| ----------------------------- | ------------------------------------------------------------ |
| 🎯 **용도**                   | 단일 리소스 연결 (기본 웹사이트 등)                          |
| ⚙️ **작동 방식**              | 하나의 IP(A Record) 또는 AWS 리소스(Alias Record) 반환       |
| ⚡ **여러 값 지정 시**        | 여러 IP를 반환할 수 있지만, 클라이언트가 임의로(random) 선택 |
| ❌ **Health Check 연동 불가** | 단일 레코드라서 장애 감지 불가능                             |
| ✅ **Alias 지원**             | 가능 (ELB, CloudFront 등)                                    |

<br>

### 🧩 예시

```shell
api.example.com  A  54.80.100.12
```

동작:

- → 사용자가 api.example.com 요청
- → Route 53이 54.80.100.12 반환
- → 클라이언트가 해당 IP로 접속

<br>

### 💬 다중 값 예시 (랜덤 반환)

```shell
Name             Type  Value
app.example.com  A     54.80.100.12
app.example.com  A     3.88.200.24
```

결과:

- → Route 53은 두 IP 중 하나를 랜덤하게 반환
- → 클라이언트 측에서 분산이 발생

<br>

## ⚖️ Weighted Routing Policy (가중치 라우팅)

- 여러 리소스에 대해 트래픽 비율(%)을 가중치로 제어하는 정책입니다.
- DNS 수준에서 “부하 분산” 또는 “버전 테스트”에 활용됩니다.

<br>

### 🧩 특징 요약

| 항목                      | 설명                                                                                |
| ------------------------- | ----------------------------------------------------------------------------------- |
| ⚙️ **동작 방식**          | 동일한 도메인 이름 + 레코드 타입에 대해 여러 레코드 생성, 각 레코드에 “Weight” 부여 |
| 🔢 **가중치 합**          | 꼭 100일 필요 없음 (Route 53이 비율로 자동 계산)                                    |
| 💉 **헬스체크 연동 가능** | 장애 리소스 자동 제외 가능                                                          |
| 💡 **트래픽 제어 예시**   | 70% → 기존 서버 / 20% → 새 버전 / 10% → 테스트 환경                                 |
| 🧰 **테스트 용도**        | Blue/Green, Canary 배포 시 자주 사용                                                |

<br>

### 🧱 예시

```shell
예시:
• A 서버 (Weight=7), B 서버 (Weight=2), C 서버 (Weight=1)
  → 트래픽 분배 = 70% : 20% : 10%
```

| Name            | Type | Value        | Weight |
| --------------- | ---- | ------------ | ------ |
| api.example.com | A    | 54.80.100.12 | 70     |
| api.example.com | A    | 3.88.200.24  | 20     |
| api.example.com | A    | 18.203.4.11  | 10     |

<br>

✅ 트래픽 분배:

- 70% → 기존 서버
- 20% → 신규 배포 서버
- 10% → 테스트 서버

<br>

### 🎉 추가 기능

| 기능                         | 설명                                            |
| ---------------------------- | ----------------------------------------------- |
| **Weight=0**                 | 임시로 트래픽 제외 (비활성화 상태로 유지 가능)  |
| **Health Check 연동**        | 장애 리소스는 DNS 응답에서 제외됨               |
| **Blue/Green 배포 시나리오** | 점진적 트래픽 전환 (예: 새 버전으로 10%씩 늘림) |

<br>

---

## ⚠️ 1️⃣ Failover Routing Policy (장애 조치 라우팅)

- 주요 목적:
  - 장애(Health Check 실패) 발생 시 자동으로 백업(Secondary) 리소스로 트래픽 전환.

💡 개념

- “Active-Passive” 구조로 운영
- Primary 리소스가 정상일 땐 모든 요청을 처리,
  장애 시 Route 53이 Secondary 리소스로 자동 Failover

### 🧭 구조 예시

```shell
               ┌────────────┐
Request  --->  │ Route 53   │
               └────┬───────┘
                    │
         ┌──────────┴──────────┐
         │                     │
   [Primary - Healthy]   [Secondary - DR Site]
     www.example.com → ALB     www.example.com → S3/EC2
```

<br>

### ✅ 주요 특징

| 항목                | 설명                                   |
| ------------------- | -------------------------------------- |
| **구성 방식**       | Primary + Secondary Record             |
| **헬스체크**        | ✅ Primary에 반드시 연결 필요          |
| **사용 사례**       | DR (Disaster Recovery), 장애 자동 복구 |
| **Primary 장애 시** | Secondary Record로 DNS 응답            |
| **복귀 시**         | Health Check 회복 후 자동 복귀         |

<br>

### ✅ 예시 요약

| 유형            | 리소스 | 역할           | 헬스체크     |
| --------------- | ------ | -------------- | ------------ |
| www.example.com | A      | ALB            | ✅ Primary   |
| www.example.com | A      | S3 Static Site | ❌ Secondary |

<br><br>

## ⚡ 2️⃣ Latency-Based Routing Policy (지연시간 기반 라우팅)

- 주요 목적:
  - 사용자의 위치에서 지연시간(latency) 이 가장 낮은 리전의 리소스로 응답.

💡 개념

- 전 세계 여러 리전에 동일 애플리케이션이 배포되어 있을 때 사용
- 사용자의 위치 → Route 53이 DNS 수준에서 최적의 리전 선택

### 🧭 구조 예시

```shell
User in Seoul → AP-Northeast-2 (서울)
User in Paris → EU-West-3 (파리)
User in US → US-East-1 (버지니아)
```

### ✅ 주요 특징

| 항목              | 설명                                          |
| ----------------- | --------------------------------------------- |
| **리소스 이름**   | 동일 (예: api.example.com)                    |
| **리전별 레코드** | 각기 다른 AWS 리전의 리소스                   |
| **헬스체크**      | ✅ 지원                                       |
| **주요 목적**     | 글로벌 트래픽 최적화, 사용자 체감 지연 최소화 |
| **Alias 지원**    | ✅ 가능 (ALB, CloudFront 등)                  |

### ✅ 예시 요약

| Name            | Type | Region         | Target          |
| --------------- | ---- | -------------- | --------------- |
| api.example.com | A    | ap-northeast-2 | ALB in Seoul    |
| api.example.com | A    | us-east-1      | ALB in Virginia |

<br><br>

## 🌍 Geolocation Routing Policy (사용자 위치 기반 라우팅)

- 주요 목적:
  - 사용자의 지리적 위치(국가, 대륙, IP 지역) 기반으로 특정 리소스로 응답.

💡 개념

- 사용자의 “요청 위치(IP)” 기준으로 특정 리전 또는 국가로 매핑
- 예: 한국 유저는 서울, 미국 유저는 버지니아

### 🧭 구조 예시

```shell
Korea → ap-northeast-2 ALB
Japan → ap-northeast-1 ALB
US → us-east-1 ALB
Default → us-west-2 ALB
```

### ✅ 주요 특징

| 항목               | 설명                                              |
| ------------------ | ------------------------------------------------- |
| **기준**           | 사용자의 지리적 위치 (GeoIP)                      |
| **설정 단위**      | 대륙 / 국가 / 주 (State)                          |
| **Default 설정**   | 지정되지 않은 지역 요청을 처리할 기본 리소스 필요 |
| **헬스체크**       | ✅ 가능                                           |
| **주요 사용 사례** | 지역별 콘텐츠, 국가 제한 서비스, 언어별 웹사이트  |

### ✅ 예시 요약

| Name        | Type | Location | Target        |
| ----------- | ---- | -------- | ------------- |
| example.com | A    | Korea    | ALB in Seoul  |
| example.com | A    | Japan    | ALB in Tokyo  |
| example.com | A    | Default  | ALB in Oregon |

<br><br>

## 🧩 Multi-Value Answer Routing Policy (다중 응답 라우팅)

- 주요 목적:
  - 여러 리소스를 동시에 반환하여 간단한 로드밸런싱 효과를 제공.

💡 개념

- Route 53이 최대 8개의 IP 주소(A Record) 를 한 번에 반환.
  하나의 DNS 이름에 대해 여러 개의 IP 주소 또는 리소스를 동시에 반환하는 정책.
- 클라이언트가 임의로 하나를 선택해 접속
- 단순 부하분산 + 헬스체크 연동 가능
- 즉, 간단한 DNS 레벨 부하분산 효과를 제공. ELB처럼 세션 관리나 동적 트래픽 제어는 수행하지 않음

### 🧭 구조 예시

```shell
DNS Query → Route 53
Response → [54.1.1.1, 54.1.1.2, 54.1.1.3, 54.1.1.4]
Client → Randomly picks one healthy IP

          User Request
               ↓
        [Route 53 Resolver]
               ↓
   Returns multiple healthy IPs
               ↓
     [54.10.1.1, 54.10.1.2, 54.10.1.3, ...]
               ↓
   Client randomly picks one IP and connects
```

### ✅ 주요 특징

| 항목           | 설명                                          |
| -------------- | --------------------------------------------- |
| **반환 수**    | 최대 8개의 IP                                 |
| **헬스체크**   | ✅ 가능 (비정상 IP 제외)                      |
| **Alias 지원** | ❌ 불가능                                     |
| **주요 목적**  | 간단한 DNS 레벨 부하분산 (LB 대체용)          |
| **주의사항**   | 진짜 “로드밸런서”가 아님 (DNS 캐시 영향 존재) |

### ✅ 예시 요약

| Name            | Type | Value    | 헬스체크 |
| --------------- | ---- | -------- | -------- |
| app.example.com | A    | 54.1.1.1 | ✅       |
| app.example.com | A    | 54.1.1.2 | ✅       |
| app.example.com | A    | 54.1.1.3 | ❌       |

<br><br>

## 🧭 Geoproximity Routing Policy (지리적 거리 + 가중치 라우팅)

- 주요 목적:
  - 사용자의 위치 + 리소스 위치 간 물리적 거리를 기반으로 가장 가까운 리소스로 응답
    (단, Route 53 Traffic Flow 기능을 사용해야 함)

💡 개념

- Traffic Flow 콘솔(시각적 라우팅 편집기) 로 설정
- 단순 거리 기반이 아니라, 각 리전에 가중치(Bias) 를 부여 가능
  → 특정 리전으로 더 많은 트래픽 유도 가능 (+Bias)

### 🧭 구조 예시

```shell
Global Users
     ↓
Route 53 Traffic Flow
     ↓
[Tokyo] Bias +20 → 더 많은 트래픽 유도
[Seoul] Bias 0 → 기본
[Oregon] Bias -10 → 일부만 라우팅
```

### ✅ 주요 특징

| 항목          | 설명                                          |
| ------------- | --------------------------------------------- |
| **작동 방식** | 사용자 위치 + 리소스 위치 거리 기반           |
| **Bias 설정** | ± 값으로 트래픽 가중치 조절 가능              |
| **헬스체크**  | ✅ 지원                                       |
| **필수 조건** | Route 53 Traffic Flow 사용                    |
| **주요 목적** | 글로벌 서비스 트래픽 최적화, 리전별 부하 조절 |

### ✅ 예시 요약

| Name        | Type | Region         | Bias | Target     |
| ----------- | ---- | -------------- | ---- | ---------- |
| example.com | A    | ap-northeast-1 | +20  | ALB Tokyo  |
| example.com | A    | ap-northeast-2 | 0    | ALB Seoul  |
| example.com | A    | us-west-2      | -10  | ALB Oregon |
