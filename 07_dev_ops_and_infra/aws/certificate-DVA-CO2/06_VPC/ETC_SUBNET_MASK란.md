# 🚀 subnet mask 란?

Subnet Mask는 IP 주소에서 어디까지가 네트워크 영역이고, 어디부터가 호스트 영역인지 구분하는 값이다.

쉽게 말하면:

- 네트워크 영역: 같은 네트워크에 속하는 주소 범위
- 호스트 영역: 그 네트워크 안에서 EC2, RDS, ENI 등에 할당될 수 있는 개별 주소

AWS VPC에서 Subnet CIDR을 이해하려면 Subnet Mask 개념이 필요하다.

```shell
IP 주소:       10.0.1.25
Subnet CIDR:  10.0.1.0/24
Subnet Mask:  255.255.255.0
```

위 예시에서 `/24`는 앞의 24비트가 네트워크 영역이라는 뜻이다.

<br>

## 🧠 Subnet Mask와 CIDR의 관계

CIDR의 `/숫자`는 Subnet Mask를 짧게 표현한 것이다.

```shell
| CIDR | Subnet Mask       | 전체 IP 개수 |
|------|-------------------|--------------|
| /16  | 255.255.0.0       | 65,536개     |
| /24  | 255.255.255.0     | 256개        |
| /26  | 255.255.255.192   | 64개         |
| /28  | 255.255.255.240   | 16개         |
```

핵심 규칙:

- Prefix 숫자가 작을수록 네트워크가 크다.
- Prefix 숫자가 클수록 네트워크가 작다.
- IPv4는 총 32비트이다.
- 전체 IP 개수 = `2^(32 - prefix)`

예를 들어 `/24`라면:

```shell
32 - 24 = 8비트
2^8 = 256개
```

즉, `10.0.1.0/24`는 `10.0.1.0 ~ 10.0.1.255` 범위를 가진다.

### 계산 방법

10.0.1.0/24가 10.0.1.0 ~ 10.0.1.255 범위를 가지는 이유는 /24가
“앞의 24비트는 네트워크 주소로 고정한다”는 뜻이기 때문입니다.

```shell
IPv4 주소는 총 32비트입니다.

10.0.1.0

이걸 4개의 숫자로 나누면 각각 8비트입니다.

10       . 0        . 1        . 0
8비트    . 8비트    . 8비트    . 8비트

/24는 앞에서부터 24비트를 네트워크 영역으로 고정한다는 뜻입니다.

10       . 0        . 1        . x
8비트    . 8비트    . 8비트    . 8비트
고정       고정       고정       변경 가능

즉 앞의 세 칸인 10.0.1은 고정되고, 마지막 한 칸만 바뀔 수 있습니다.

마지막 한 칸은 8비트라서 만들 수 있는 값이 0 ~ 255입니다.

그래서 전체 범위가 이렇게 됩니다.

10.0.1.0
10.0.1.1
10.0.1.2
...
10.0.1.255

결론:

10.0.1.0/24
= 앞 24비트, 즉 10.0.1 까지는 고정
= 마지막 8비트만 변경 가능
= 마지막 숫자는 0 ~ 255 가능
= 범위는 10.0.1.0 ~ 10.0.1.255

단, AWS Subnet에서는 이 범위 전체를 인스턴스에 다 쓸 수는 없습니다. AWS가 5개 IP를 예약하기 때문에
/24의 실제 사용 가능 IP는 256 - 5 = 251개입니다.
```

<br>

## 🏗 AWS VPC Subnet에서 왜 중요한가?

VPC는 큰 네트워크 주소 공간이고, Subnet은 그 안을 더 작게 나눈 네트워크이다.

```shell
VPC CIDR: 10.0.0.0/16

Public Subnet A:  10.0.1.0/24
Public Subnet B:  10.0.2.0/24
Private Subnet A: 10.0.10.0/24
Private Subnet B: 10.0.20.0/24
```

여기서 `/24`가 각 Subnet의 크기를 결정한다.

즉, Subnet Mask를 잘못 잡으면:

- 필요한 EC2, Lambda ENI, RDS, NAT Gateway 등을 배치할 IP가 부족할 수 있다.
- 나중에 Subnet 크기를 쉽게 키우기 어렵다.
- VPC Peering, VPN, Direct Connect 구성 시 CIDR 충돌 문제가 생길 수 있다.

<br>

## ⚠️ AWS에서 꼭 기억할 점: Subnet마다 IP 5개는 예약된다

AWS는 각 Subnet에서 IPv4 주소 5개를 예약한다.

예를 들어 `10.0.1.0/24` Subnet이 있다면:

```shell
10.0.1.0   네트워크 주소
10.0.1.1   VPC 라우터
10.0.1.2   AWS DNS 서버
10.0.1.3   AWS 예약
10.0.1.255 브로드캐스트 주소
```

따라서 `/24`의 전체 IP는 256개지만, AWS에서 실제 사용 가능한 IP는 다음과 같다.

```shell
256 - 5 = 251개
```

시험에서 자주 헷갈리는 부분이다.

<br>

## 🔢 자주 나오는 계산 예시

### `/28`

```shell
전체 IP 개수 = 2^(32 - 28)
             = 2^4
             = 16개

AWS 사용 가능 IP = 16 - 5 = 11개
```

`/28`은 AWS IPv4 Subnet에서 만들 수 있는 가장 작은 크기이다.

<br>

### `/24`

```shell
전체 IP 개수 = 2^(32 - 24)
             = 2^8
             = 256개

AWS 사용 가능 IP = 256 - 5 = 251개
```

<br>

### `/16`

```shell
전체 IP 개수 = 2^(32 - 16)
             = 2^16
             = 65,536개

AWS 사용 가능 IP = 65,536 - 5 = 65,531개
```

단, 실제로는 `/16` 전체를 하나의 Subnet으로 쓰기보다 여러 Subnet으로 나누어 사용한다.

<br>

## ✅ DVA-C02 시험 포인트

- VPC CIDR은 VPC 전체 주소 범위이다.
- Subnet CIDR은 VPC CIDR 안에 포함되어야 한다.
- Subnet은 하나의 Availability Zone(AZ)에만 속한다.
- Subnet CIDR끼리는 겹치면 안 된다.
- AWS는 각 Subnet에서 IPv4 주소 5개를 예약한다.
- Subnet 크기가 너무 작으면 EC2, Lambda ENI, ECS Task, RDS 등에 할당할 IP가 부족해질 수 있다.
- Public Subnet 여부는 Subnet Mask가 아니라 Route Table에 `0.0.0.0/0 -> Internet Gateway` 경로가 있는지로 결정된다.

<br>

## 🧪 시험 예상 문제

### 문제 1

개발자가 AWS VPC 안에 `10.0.1.0/28` CIDR을 가진 Subnet을 생성했다. 이 Subnet에서 EC2 인스턴스 등에 실제로 할당 가능한 IPv4 주소 개수는 몇 개인가?

A. 11개
B. 14개
C. 16개
D. 251개

<br>

### 정답

A. 11개

<br>

### 해설

`/28`은 전체 IPv4 주소가 16개이다.

```shell
2^(32 - 28) = 2^4 = 16
```

하지만 AWS는 각 Subnet에서 IPv4 주소 5개를 예약한다.

```shell
16 - 5 = 11
```

따라서 실제로 EC2, ENI 등에 할당 가능한 IPv4 주소는 11개이다.

<br>

## 🧪 시험 예상 문제 2

다음 중 Public Subnet을 결정하는 가장 정확한 기준은 무엇인가?

A. Subnet CIDR이 `/24`이면 Public Subnet이다.
B. Subnet Mask가 `255.255.255.0`이면 Public Subnet이다.
C. Route Table에 `0.0.0.0/0 -> Internet Gateway` 경로가 연결되어 있으면 Public Subnet이다.
D. Subnet이 여러 AZ에 걸쳐 있으면 Public Subnet이다.

<br>

### 정답

C. Route Table에 `0.0.0.0/0 -> Internet Gateway` 경로가 연결되어 있으면 Public Subnet이다.

<br>

### 해설

Subnet Mask 또는 CIDR 크기는 Subnet의 주소 범위를 정할 뿐이다.

Public Subnet인지 Private Subnet인지는 인터넷으로 나가는 기본 경로가 있는지로 판단한다.

```shell
0.0.0.0/0 -> Internet Gateway
```

이 경로가 연결된 Route Table을 사용하는 Subnet은 Public Subnet이 될 수 있다.

반대로 이 경로가 없고 NAT Gateway 등을 통해서만 외부로 나간다면 일반적으로 Private Subnet으로 본다.

<br>

## 요약

- Subnet Mask는 IP 주소에서 네트워크 영역과 호스트 영역을 구분하는 값이다.
- CIDR의 `/24`, `/28` 같은 Prefix는 Subnet Mask를 짧게 표현한 것이다.
- AWS Subnet의 실제 사용 가능 IPv4 수는 `전체 IP 개수 - 5`로 계산한다.
- Public/Private Subnet 여부는 Subnet Mask가 아니라 Route Table 구성으로 결정된다.
- DVA-C02에서는 Subnet CIDR 계산, AWS 예약 IP 5개, Public Subnet 조건을 꼭 기억해야 한다.
