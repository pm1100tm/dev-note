# 🚀 Route53 Health Check

## 🧩 1️⃣ 개념 정리

- Health Check 는 Route53 이 지정한 리소스(Endpoint)가, [정상/비정상] 상태를 자동으로
  모니터링하는 기능입니다.
- Route53 이 전 세계에서 여러 위치에서 HTTP/HTTPS/TCP 요청을 주기적으로 보내, 앤드포인트의
  정상 여부를 판단합니다.
- DNS 수준에서 헬스체크 결과를 기반으로 Failover Routing / Multi-Value Answer Routing
  과 같은 정책과 결합됩니다.

### 🧭 작동 구조

```shell
[Route 53 Health Check]
       ↓ (HTTP/HTTPS/Ping)
  [Target Endpoint]
       ↓
Healthy → 정상 DNS 응답
Unhealthy → Failover / 제외
```

<br>

## ⚙️ 2️⃣ 주요 특징

- 감시방식
  - HTTP / HTTPS / TCP 요청
- 체크주기
  - 기본 30초, 최소 10초
- 평가기준
  - 2xx, 3xx = Healthy
- 통합 서비스
  - AWS CloudWatch 와 연동(지표, 알람 생성 가능)
- 자동 DNS Failover
  - 장애 감지 시 Secondary 리소스로 자동 전환 가능
- 비용
  - 헬스체크당 월 $0.05 ~ $1.00 (유형별 상이)

<br>

## 🌍 3️⃣ 퍼블릭 vs 프라이빗 리소스 모니터링

- 🌐 Public Resource (인터넷 공개)

  - 모니터링 가능
  - HTTP/HTTPS 직접 요청으로 상태 확인 가능

- 🔒 Private Resource (VPC 내부)
  - ❌ 직접 체크 불가 → CloudWatch Alarm 기반 헬스체크 필요

<br>

## 🧱 4️⃣ Health Check 유형

Route 53은 3가지 헬스체크 유형을 지원합니다 👇

- ✅ (1) Endpoint Health Check
- ✅ (2) Calculated Health Check
- ✅ (3) CloudWatch Alarm 기반 Health Check

### Endpoint Health Check

- 외부에서 접근 가능한 퍼블릭 엔드포인트를 직접 감시
  - 예: EC2, ALB, S3 웹사이트, API Gateway

| 항목         | 설명                                  |
| ------------ | ------------------------------------- |
| 대상         | IP주소, DNS                           |
| 프로토콜     | HTTP, HTTPS, TCP                      |
| 응답기준     | 2xx ~ 3xx 정상 응답 시 Healthy        |
| 주 사용 사례 | 웹 서비스, ALB, 정적 사이트 상태 감시 |

### Calculated Health Check

- 다른 헬스체크의 결과를 조합하여 **_그룹 상태_** 를 판정하는 방식
- 여러 개의 하위(Child) 헬스체크 결과를 조합하여, 하나의 “상위(Parent)” 헬스체크 결과로 표현하는
  기능
- 즉, “여러 리전 또는 여러 서비스의 상태를 하나의 헬스체크로 통합”할 때 사용

| 항목      | 설명                                                      |
| --------- | --------------------------------------------------------- |
| 구성      | 여러 개의 Endpoint 헬스체크를 묶어 계산                   |
| 조건식    | “모두 정상(All must pass)” or “하나라도 정상(Any passes)” |
| 사용 사례 | 다중 리전 서비스의 종합 상태 판단, DR 사이트 결정 등      |

```shell
HealthCheckGroup = (Seoul ALB ✅) AND (Tokyo ALB ✅)
→ 모두 Healthy면 “정상”
```

#### 💡 개념

- 여러 개의 헬스체크를 묶어서 AND / OR / NOT 조건으로 조합 가능
- 최대 256개의 Child Health Check를 하나의 Calculated Health Check에 포함 가능
- Parent Check의 상태는 Child Check들의 결과로 자동 계산됨

🧭 구조 예시

```shell
        ┌───────────────┐
        │ Route 53       │
        │ Calculated HC  │
        └──────┬────────┘
               │
  ┌────────────┼─────────────┐
  │                          │
[Child HC #1]           [Child HC #2]
 api.example.com          app.example.com
    (Seoul)                  (Tokyo)
```

- ➡️ Parent 헬스체크 상태 = (Seoul Healthy) AND (Tokyo Healthy)

### CloudWatch Alarm 기반 Health Check

- Health Checker는 VPC 외부(인터넷) 에 존재하기 때문에,
  Private 리소스(VPC 내부 EC2, RDS, 내부 ALB 등)는 직접 모니터링이 불가능
- CloudWatch 지표(예: CPU, RDS 연결, DynamoDB 제한 등)를 기반으로 헬스 상태를 판단하는
  고급 방식

| 항목          | 설명                                                    |
| ------------- | ------------------------------------------------------- |
| **대상**      | CloudWatch Metric / Alarm                               |
| **접근 방식** | 퍼블릭 접근 불필요 (Private 리소스도 가능)              |
| **예시**      | RDS CPU 사용률, DynamoDB Throttling, 사용자 정의 메트릭 |
| **장점**      | 내부 리소스도 감시 가능 + 완전 자동화 가능              |

- CloudWatch Metric을 이용해 리소스 상태를 간접적으로 감시하고,
  그 Alarm 상태를 Route 53 Health Check로 연결할 수 있습니다.

```shell
[Private EC2 / RDS / DynamoDB]
           ↓
  [CloudWatch Metric 생성]
           ↓
  [CloudWatch Alarm (OK / ALARM)]
           ↓
  [Route 53 Health Check (checks Alarm)]
           ↓
  DNS Failover 작동
```

<br>

## 📊 5️⃣ CloudWatch 통합

- Route 53 헬스체크 결과는 자동으로 CloudWatch Metrics 에 연동됩니다.

| 지표                             | 설명                                 |
| -------------------------------- | ------------------------------------ |
| **HealthCheckStatus**            | 1 = Healthy / 0 = Unhealthy          |
| **ChildHealthCheckHealthyCount** | 하위 헬스체크 중 Healthy 상태의 개수 |
| **ConnectionTime**               | 응답 시간 (ms 단위)                  |

- ➡️ CloudWatch 대시보드나 Alarm 규칙을 사용하여, 비정상 시 Slack / SNS / Lambda
  알림 전송 가능

<br>

## 🔁 6️⃣ Health Check + Routing Policy 통합 예시

```shell
[Primary ALB] ← Health Check 연결
     ↓
Unhealthy → Route 53 → Secondary S3 Site
```

→ Failover Routing Policy 와 결합 시 완전 자동화된 장애 복구 가능

<br>
<br>

---

## 🩺 Amazon Route 53 – Health Checks (Monitor an Endpoint)

Route 53이 전 세계 여러 위치에서 HTTP/HTTPS/TCP 요청을 주기적으로 보내어,
엔드포인트의 “정상 여부”를 판단하는 헬스체크입니다.

- ✅ 퍼블릭 리소스(API, ALB, EC2, S3 Static Website 등)에 사용 가능
- ❌ Private VPC 내부 리소스에는 직접 접근 불가 (→ CloudWatch 기반 헬스체크 사용 필요)

### 🌍 2️⃣ 글로벌 헬스체커 구조

Route 53은 전 세계 약 15개 글로벌 헬스체커(Health Checker) 노드를 통해,
엔드포인트의 상태를 지속적으로 감시합니다.

```shell
   ┌────────────┐
   │ 15 Health  │  ← 분산된 AWS 리전별 헬스체커
   │  Checkers  │
   └────┬───────┘
        │
        ▼
   [Endpoint Target]
     ├─ EC2, ALB, S3 Website 등
     └─ 응답 코드(2xx, 3xx) 확인
```

## ⚙️ 3️⃣ 동작 메커니즘

- 1️⃣ Route 53 헬스체커가 엔드포인트에 HTTP/HTTPS 요청 전송
- 2️⃣ 2xx~3xx 응답 코드 수신 시 Healthy 로 판단
- 3️⃣ 15개 노드 중 18% 이상이 성공 보고 시 전체를 Healthy 로 간주
- 4️⃣ 반대로 과반 이상이 실패하면 Unhealthy
- 5️⃣ Failover / Weighted / Multi-Value Routing 정책과 연동되어 트래픽을 자동 전환

## 🔧 4️⃣ 주요 설정 파라미터

- Protocol

  - HTTP / HTTPS / TCP 중 선택
  - 기본 값: HTTP

- Interval

  - 헬스체크 주기
  - 기본 값: 30초 (또는 10초 – 고비용)

- Healthy Threshold

  - 몇 번 연속 성공 시 Healthy 로 판단
  - 기본 값: 3회

- Unhealthy Threshold

  - 몇 번 연속 실패 시 Unhealthy 로 판단
  - 기본 값: 3회

- Evaluation Criteria

  - 응답 코드 (2xx / 3xx만 통과)

- Response Text Match

  - 응답 본문 앞 5120바이트 내 특정 텍스트 포함 시 Pass
  - 기본 값: 선택적

- Check Locations

  - 헬스체커의 지리적 위치 직접 선택 가능
  - 기본 값: 전역 기본값

- Firewall Rule
  - 헬스체커 IP 대역 허용 필요
  - 기본 값: Route 53 IP Range 설정 필요

💡 설정 Example:

```shell
Endpoint: https://api.example.com/health
Protocol: HTTPS
Interval: 30초
Healthy Threshold: 3회
Unhealthy Threshold: 3회
Success Code: 200~399
Text Match: "OK" (응답 본문에 포함되어야 함)
```

## 🧱 5️⃣ 상태 판정 기준

- ✅ Healthy: 15개 중 18% 이상 헬스체커가 정상 응답 보고
- ❌ Unhealthy: 대부분의 헬스체커가 실패 보고
- 📉 DNS 영향: Routing Policy (Failover, Weighted 등)에 따라 비정상 리소스가 자동 제외됨

## 🔐 6️⃣ 방화벽(Firewall) 설정 주의

- 엔드포인트가 사설망 뒤에 있을 경우,
  - 반드시 Route 53 Health Checker IP 대역을 허용해야 합니다.

AWS 공식 문서에서 "Route53 Health Checker IP Range" JSON 을 주기적으로 업데이트 해줍니다.

- https://ip-ranges.amazonaws.com/ip-ranges.json

```shell
# 예시 – Security Group 허용
aws ec2 authorize-security-group-ingress \
  --group-id sg-xxxxxx \
  --protocol tcp \
  --port 443 \
  --cidr 52.94.0.0/15   # Route 53 Health Checker IP 대역
```

## 🧠 7️⃣ CloudWatch 연동

모든 Health Check는 자동으로 CloudWatch Metrics 에 기록됩니다.

| Metric Name           | 설명                                      |
| --------------------- | ----------------------------------------- |
| **HealthCheckStatus** | 1 = Healthy / 0 = Unhealthy               |
| **ConnectionTime**    | 응답 시간 (ms 단위)                       |
| **SSLHandshakeTime**  | SSL/TLS 연결 시간                         |
| **FailureReason**     | 실패 원인 (Timeout, ConnectionRefused 등) |

- ➡️ CloudWatch Alarm 생성 시
- → SNS, Lambda, Slack 등으로 장애 알림 자동 전송 가능
