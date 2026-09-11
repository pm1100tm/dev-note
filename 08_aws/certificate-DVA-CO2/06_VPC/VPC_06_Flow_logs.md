# 🚀 VPC Flow Logs

| VPC Flow Logs란?

- VPC 내 네트워크 인터페이스(ENI)로 들어오고 나가는 IP 트래픽에 메타데이터를 기록하는 로그
- 패킷 내용(payload)은 기록하지 않음
- 누가 -> 어디로 -> 허용/거부 되었는지만 기록

## Flow Logs를 만들 수 있는 범위

- VPC Flow Logs: VPC 전체 트래픽
- Subnet Flow Logs: 특정 Subnet
- ENI Flow Logs: 특정 EC2 / ALB / RDS

### 어떤 트래픽을 캡쳐하는가?

- Inbound / Outbound IP 트래픽
- VPC 내부 통신
- AWS Managed 서비스 트래픽 포함

### AWS Managed Interface 포함 예시

| 서비스                |
| --------------------- |
| Elastic Load Balancer |
| RDS / Aurora          |
| ElastiCache           |
| NAT Gateway           |
| VPC Endpoint          |

❗ 이게 중요한 이유

- “RDS 연결 안됨” 같은 문제도 Flow Logs로 분석 가능

### 🧑‍💻 Flow Logs로 알 수 있는 것

대표적인 트러블슈팅 케이스

| 상황                            | 확인 포인트  |
| ------------------------------- | ------------ |
| **Subnet → Internet 안됨**      | NACL / Route |
| **Subnet ↔ Subnet 통신 실패**   | NACL / SG    |
| **Internet → Subnet 접근 실패** | NACL / SG    |
| **ALB → EC2 연결 실패**         | SG           |
| **RDS 접속 안됨**               | SG / NACL    |

### 🧵 Flow Logs가 저장되는 곳

- CloudWatch Logs: 실시간 분석 / 필터링
- Kinesis Data Firehose: 실시간 스트리밍 / SIEM
- S3: 장기 보관 / Athena 분석

#### Flow Log 레코드 핵심 필드 (시험 단골)

| 필드         | 의미            |
| ------------ | --------------- |
| **srcaddr**  | 출발지 IP       |
| **dstaddr**  | 목적지 IP       |
| **srcport**  | 출발지 포트     |
| **dstport**  | 목적지 포트     |
| **protocol** | TCP / UDP       |
| **action**   | ACCEPT / REJECT |
| **bytes**    | 전송 바이트     |
| **packets**  | 패킷 수         |

🔥 제일 중요한 필드

- action = ACCEPT / REJECT

### Flow Logs로 알 수 없는 것 (중요 ⚠️)

| 항목                | 이유           |
| ------------------- | -------------- |
| 패킷 내용           | Payload 미기록 |
| DNS 이름            | IP만 기록      |
| Application 오류    | 네트워크까지만 |
| ICMP Ping 허용 여부 | 일부 제한      |

👉 “애플리케이션 문제인지, 네트워크 문제인지” 구분용

### 실전 예시 1 (시험 + 실무)

- ❓ EC2에서 인터넷이 안된다

  - Subnet Flow Logs 확인
  - action = REJECT 발견
  - dstaddr = 0.0.0.0/0
  - 👉 NACL Outbound 문제 확정

이게 왜 문제냐면

- dstaddr 이란
  - 패킷이 향하는 목적지 IP
  - 실제 로그에서는 보통 구체적인 IP가 찍힘
  - 0.0.0.0/0 은 단일 IP가 아니고, 불특정 인터넷 주소로 나가려는 트래픽

```shell
dstaddr = 8.8.8.8
dstaddr = 142.250.206.46
```

### 실전 예시 2 (시험 + 실무)

- ❓ ALB는 정상인데 EC2 연결 실패
  - ENI Flow Logs (EC2)
  - srcaddr = ALB IP
  - dstport = 8080
  - action = REJECT
  - 👉 EC2 Security Group 문제

### 실전 예시 3 (시험 + 실무 + 고급): Flow Logs 는 Accept 인데도 통신이 안된다

핵심 결론 (먼저)

- VPC Flow Logs는 “L3/L4 네트워크 레벨”까지만 본다
- 따라서 라우팅 이후 / OS / 애플리케이션 / 응답 경로 문제는 ACCEPT여도 실패한다.

#### 1️⃣ Route Table 문제 (가장 흔함)

```shell
# 상황
- 패킷은 ENI까지 도달
- Flow Logs: ACCEPT
- 하지만 다음 홉으로 못 감

# 예시
EC2 (Private Subnet)
Route:
0.0.0.0/0 → (❌ 없음)

# 결과
ENI 기준: 트래픽은 허용됨 → ACCEPT
실제로는 인터넷으로 나갈 경로가 없음

# 판단 기준

Flow Logs는 Route Table 검사 안 함
“ENI를 통과했는가”만 기록
```

#### 2️⃣ NAT Gateway / IGW 문제

```shell
Subnet Route는 정상
Flow Logs: ACCEPT
NAT Gateway 없음 / 다운 / AZ 불일치

Private Subnet (AZ-a)
→ NAT Gateway (AZ-b ❌)

# 결과
Flow Logs: ACCEPT
실제 통신: 타임아웃

# 이유
Flow Logs는 NAT 이후 구간을 보지 못함
```

#### 3️⃣ Security Group은 OK, 하지만 “응답 경로”가 다름

```shell
요청:
Client → ALB → EC2

응답:
EC2 → NAT → Internet (❌)

# 결과
Inbound: ACCEPT
Outbound: ACCEPT
Client는 응답을 못 받음
👉 응답이 요청 들어온 경로와 다르면 실패
```

#### 4️⃣ OS / Instance 내부 문제 (Flow Logs 밖)

Flow Logs는 여기부터 모른다 👇

```shell
OS Firewall: iptables, ufw
포트 미리스닝: 서버가 안 떠 있음
프로세스 다운: app crash
Thread 고갈: connection accept 불가
```

#### 5️⃣ DNS 문제 (Flow Logs는 IP만 봄)

```shell
# 상황
DNS 해석 실패
IP로 나가는 트래픽 자체가 없음

# 결과
Flow Logs: 아무 로그도 없음
사용자는 “인터넷 안 됨”이라고 느낌

👉 Flow Logs는 DNS 실패를 기록하지 않는다
```

---

### 한 줄 요약 (암기)

- VPC Flow Logs = 네트워크 CCTV
- 패킷 내용 ❌ / 흐름 기록 ⭕
- ACCEPT / REJECT 확인이 핵심

✅ 실무 Best Practice

```shell
- 기본: Subnet Flow Logs
- 장애 시: ENI Flow Logs
- 장기 분석: S3 + Athena
- 실시간 탐지: Firehose + SIEM
```

---

## 추가적으로 이어서 알아보면 좋을 것

- 🔥 Flow Logs로 SG vs NACL 구분하는 법
- 🔥 Flow Logs + Athena 실전 쿼리
- 🔥 왜 VPC Flow Logs 비용이 나오는지
