# 🚀 VPC Subnet

- VPC를 더 작은 네트워크 영역으로 나눈 것 (AZ 단위 배치)

즉

- VPC = 큰 땅
- Subnet = 그 땅 안에 만든 여러 개의 구역(건물용 대지)

Subnet 특징

- 각 Subnet은 특정 Availability Zone(AZ) 에 연결됨
  - 예: ap-northeast-2a, ap-northeast-2c
- Public Subnet / Private Subnet 으로 역할 구분
- Subnet 별로 Route Table을 통해 트래픽 방향을 정의

<br>

## 🌍 Public Subnet & Private Subnet

| 구분               | 설명                  | AWS 예시                       |
| ------------------ | --------------------- | ------------------------------ |
| **Public Subnet**  | 인터넷에서 접근 가능  | 웹서버, ALB, Bastion           |
| **Private Subnet** | 인터넷 직접 접근 불가 | EC2 App 서버, RDS, ElastiCache |

<br>

Public Subnet의 핵심 차이는 Route Table에 IGW(Internet Gateway) 경로가 있느냐 입니다.

```shell
0.0.0.0/0 → Internet Gateway
```

<br>

## 🗺 VPC & Subnet 구조도

```shell
VPC: 10.0.0.0/16
│
├── Public Subnet (10.0.1.0/24)
│     └── ALB, Bastion, Public EC2
│
└── Private Subnet (10.0.2.0/24)
      ├── EC2 App
      └── RDS / Redis
```
