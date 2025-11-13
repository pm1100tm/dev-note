# 🚀 VPC NAT Gateway

🧩 NAT Gateway란?

- Private Subnet 내부의 리소스(EC2 등)가 인터넷으로 나갈 수 있게 해주는 AWS 관리형 서비스
- 하지만 반대로 인터넷에서 Private Subnet으로 직접 들어올 수는 없음 (INBOUND 불가)
- 즉, “나갈 수만 있고 들어올 수 없는 일방통행 게이트웨이”.

<br>

## 🧭 NAT Gateway가 필요한 이유

Private Subnet에는 IGW 경로가 없기 때문에, 아무리 Public IP를 줘도 직접 인터넷 접근이 불가능함.

➡️ 그래서 **Private Subnet 리소스**가

- OS 패치
- 패키지 설치
- 외부 API 호출
  을 하려면 NAT Gateway가 필요함.

<br>

## 🌍 NAT Gateway 아키텍처 흐름

```shell
Private Subnet EC2
      ↓
Route Table: 0.0.0.0/0 → NAT Gateway
      ↓
NAT Gateway (Public Subnet에 위치)
      ↓
Internet Gateway(IGW)
      ↓
Internet
```

<br>

**핵심 규칙**

- NAT Gateway는 반드시 Public Subnet에 있어야 한다
- Private Subnet → NAT Gateway → IGW → 인터넷
- 인터넷 → NAT Gateway → Private Subnet 은 불가능

<br>

## ⚙️ NAT Gateway 설정 시 필수 조건

- NAT Gateway 위치

  - 필요 여부: Public Subnet 필요
  - 설명: NAT GW는 Public Subnet에 위치

- NAT GW의 Elastic IP

  - 필요 여부: 필수
  - 설명: NAT 가 외부와 통신하려면 EIP 필요

- Private Subnet Route Table
  - 필요 여부: 0 0.0.0.0/0 → NAT GW
  - 설명: 모든 외부 트래픽 NAT GW로 보냄

<br>

## 💸 NAT Gateway 요금 (중요)

| 항목                        | 비용                      |
| --------------------------- | ------------------------- |
| **NAT GW 시간당 비용**      | 약 $0.058 (리전마다 다름) |
| **NAT GW 데이터 처리 비용** | 약 $0.048 / GB            |

- ➡️ 실무에서 사람이 비용 폭탄을 가장 많이 맞는 서비스 중 하나…
- ➡️ 대안: NAT Instance, VPC Endpoint (특히 S3/DynamoDB)
