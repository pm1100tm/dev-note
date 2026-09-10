# 🚀 Route53 Overview

Amazon Route 53 은 AWS에서 제공하는 고가용성(Highly Available), 확장성(Scalable),
완전관리형(Managed), Authoritative DNS 서비스 입니다.

## ⚙️ 주요 기능 (Features)

- 🧭 Authoritative DNS
  - 사용자가 직접 관리하는 도메인의 공식 DNS 서버 역할 수행
- 🏷️ Domain Registrar
  - 도메인을 직접 등록(구매) 및 관리 가능 (example.com)
- 🔁 Traffic Routing Policy
  - 여러 리전, 서버, 또는 IP로 트래픽 라우팅 제어
- ❤️ Health Checks
  - 리소스의 상태를 모니터링하여 비정상 노드로 트래픽 차단
- 🌎 Global DNS Network
  - 전 세계 AWS DNS 서버를 통해 초저지연 응답
- 🧱 Integration with AWS Services
  - ELB, S3 Static Website, CloudFront, API Gateway 등과 손쉽게 연동 가능
- 📜 100% Availability SLA
  - AWS 서비스 중 유일하게 가용성 100% 보증 제공

### 🧠 Authoritative DNS란?

> “Authoritative” = 최종 진실을 보유한 DNS

즉, 해당 도메인의 DNS 정보(레코드)를 직접 수정할 수 있는 권한이 있는 DNS입니다.

- Authoritative DNS: example.com의 IP 주소를 직접 관리하는 서버 (Route 53)
- Recursive DNS: 사용자의 요청을 대신 찾아주는 서버 (예: ISP, Google DNS)

### 🧩 이름의 의미 “Route 53”

“53”은 전통적인 DNS 서비스 포트 번호 (TCP/UDP Port 53) 를 의미합니다.
즉, “DNS를 라우팅(Route)”한다는 뜻에서 Route 53이라는 이름이 붙었습니다.

### 🧭 주요 구성요소

- Hosted Zone
  - 특정 도메인에 대한 DNS 레코드 집합 (예: example.com)
- Record Set
  - 도메인 이름 → IP 주소 또는 AWS 리소스 매핑 (예: A, CNAME, MX 레코드 등)
- Routing Policies
  - 트래픽 분산 및 제어 방식 정의 (예: Simple, Weighted, Latency 등)
- Health Check
  - 리소스 상태를 모니터링하여 비정상 노드로 트래픽 차단

### 🚦 지원하는 Routing Policy 종류

| 정책                  | 설명                                          | 사용 예시                      |
| --------------------- | --------------------------------------------- | ------------------------------ |
| **Simple Routing**    | 하나의 레코드로 트래픽 라우팅                 | example.com → EC2              |
| **Weighted Routing**  | 가중치 기반 트래픽 분산                       | 80% → Seoul, 20% → Tokyo       |
| **Latency Based**     | 사용자 지연시간(latency)이 가장 낮은 리전으로 | 한국 유저 → ap-northeast-2     |
| **Failover Routing**  | Primary/Secondary 구조로 장애 시 자동 전환    | RDS Multi-AZ 등                |
| **Geolocation**       | 사용자 지리적 위치 기준                       | 미국 → US 서버, 한국 → KR 서버 |
| **Multivalue Answer** | 여러 IP 응답 (간단한 로드밸런싱 효과)         | 여러 EC2 인스턴스 IP 반환      |

### ❤️ Health Checks (상태 점검 기능)

Route 53은 DNS 레벨에서 리소스 상태를 모니터링할 수 있습니다.

- 특정 엔드포인트(EC2, ALB, CloudFront URL 등)에 주기적으로 HTTP/HTTPS 요청
- 비정상 응답 시 해당 레코드로 트래픽 전달 중단
- Failover Routing 정책과 함께 자주 사용됨
- 예:
  - 서울 서버 다운 시 → 도쿄 서버로 자동 DNS Failover

### 🏷️ Domain Registrar 기능

- Route 53은 단순 DNS 호스팅뿐 아니라 도메인 등록 서비스도 제공합니다.
- 즉, example.com을 Route 53에서 직접 구매·등록할 수 있습니다.

> 📌 하지만, 도메인 등록 기능은 일부 TLD(예: .kr 등)는 아직 지원하지 않을 수도 있습니다.
