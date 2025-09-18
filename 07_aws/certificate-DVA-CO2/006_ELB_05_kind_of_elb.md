# 🚀 AWS 에서 제공하는 로드 벨런서 유형

- Classic Load Balancer (CLB)
- Application Load Balancer (ALB)
- Network Load Balancer (NLB)

## 🔹 Classic Load Balancer (CLB) – 2009 (구세대, v1)

- 지원 계층: Layer 4 (TCP), Layer 7 (HTTP/HTTPS)
- 지원 프로토콜: HTTP, HTTPS, TCP, SSL
- 헬스 체크: TCP 또는 HTTP 기반
- 호스트명: 고정된 형태 → XXX.region.elb.amazonaws.com
- 👉 구세대 로드 밸런서, 단순한 기능만 제공

---

## 🔹 2. Application Load Balancer (ALB) – 2016 (신세대, v2)

- 동작 계층: Layer 7 (HTTP)
- 여러 애플리케이션에 대한 로드 밸런싱 가능
  - 여러 EC2 인스턴스 간 분산
  - 한 머신(서버)의 여러 애플리케이션(컨테이너) 간 분산
- 지원 기능
  - HTTP/2, WebSocket 지원
  - 리다이렉트 지원 (예: HTTP → HTTPS 자동 전환)

👉 고급 라우팅 기능

- URL 경로 기반 라우팅
  - example.com/users → Users 서비스
  - example.com/posts → Posts 서비스
- 호스트 기반 라우팅
  - one.example.com → 서비스 A
  - other.example.com → 서비스 B
- 쿼리 스트링/헤더 기반 라우팅
  - example.com/users?id=123&order=false
  - 요청 헤더 값에 따라 다른 Target Group으로 분기

👉 적합한 워크로드

- 마이크로서비스 아키텍처
- 컨테이너 기반 애플리케이션 (Docker, Amazon ECS)
- 포트 매핑 기능 지원 → ECS 같은 환경에서 동적 포트로 라우팅 가능

👉 CLB였다면, 애플리케이션마다 여러 개 로드 밸런서 필요했을 상황을 → ALB는 하나로 통합 가능

✅ 한 줄 요약

- CLB: 단순, 옛 세대, 기능 제한적
- ALB: 최신, L7 기반, 라우팅 유연 → 마이크로서비스/컨테이너 시대에 최적

### ALB Target Groups (대상 그룹)

ALB 가 트래픽을 보내줄 수 있는 대상들

- [1.] ECS 인스턴스
  - 직접 연결 가능
  - Auto Scaling Group 과 연동해서 자동 확장/축소 관리 가능
  - 프로토콜: HTTP
- [2.] ECS 태스크 (ECS 컨테이너)
  - Amazon ECS가 관리하는 컨테이너
  - 마이크로서비스/컨테이너 아키텍처에 적합
  - 프로토콜: HTTP
- [3.] Lambda 함수
  - HTTP 요청을 JSON 이벤트로 변환해서 Lambda 로 전달
  - 서버리스 애플리케이션 연결 가능
- [4.] IP 주소
  - 대상 그룹에 직접 프라이빗 IP 주소를 등록 가능
  - 온프레미스 서버나 VPS 피어링된 리소스와도 연결 가능

🔹 ALB의 특징

- 하나의 ALB가 여러 Target Group을 동시에 라우팅 가능
- Target Group마다 헬스 체크(Health Check) 설정 가능
- 각 Target Group은 /health 같은 엔드포인트를 주기적으로 체크해서 정상/비정상 인스턴스를 구분

✅ 쉽게 정리

- Target Group = ALB가 트래픽을 보내는 목적지 묶음
- EC2, ECS, Lambda, IP 주소 모두 대상이 될 수 있음
- Target Group 마다 헬스 체크가 있어서 비정상 서버는 자동으로 제외됨

### 📌 Application Load Balancer (ALB) – 알아두면 좋은 점

- 고정된 호스트 이름

  - ALB 는 생성 시 고정된 DNS 이름을 가짐
  - 형식: XXX.region.elb.amazonaws.com
  - IP 주소는 고정되지 않고 변경될 수 있음 -> 항상 DNS 이름으로 접근해야 함

- 클라이언트 IP 문제

  - 애플리케이션 서버(EC2)는 클라이언트의 실제 IP 를 직접 보지 못함
  - ALB 가 대신 요청을 받아서 서버에 전달하기 때문

- X-Forwarded 헤더
  - ALB는 클라이언트의 실제 정보를 HTTP 헤더에 추가해서 전달함
  - X-Forwarded-For → 클라이언트의 실제 IP
  - X-Forwarded-Port → 클라이언트가 접속한 포트
  - X-Forwarded-Proto → 사용한 프로토콜 (HTTP / HTTPS)

✅ 쉽게 설명

- ALB는 항상 DNS로 접근해야 한다 (IP 고정 아님)
- 서버에서 보이는 IP는 ALB의 IP이지, 클라이언트의 진짜 IP가 아님
- 따라서 실제 클라이언트 정보는 X-Forwarded 헤더에서 확인해야 함

---

## 🔹 3. Network Load Balancer (NLB) – 2017 (신세대, v2)

- 지원 프로토콜: TCP, TLS(보안 TCP), UDP
- 네트워크 계층(4계층) 로드 밸런싱
  - TCP & UDP 트래픽을 직접 인스턴스(EC2)로 전달
- 특징:
  - 초당 수백만 요청 처리 가능(초고성능)
  - 최저지연(Ultra-low-latency) 제공 -> 지연에 민감한 서비스에 적합
    - 지연(latency) 최소화 (수 밀리초 단위)
  - IP 주소 관련
    - AZ(가용 영역)마다 고정된 1개 IP 제공
    - 원한다면 EIP 할당 가능 -> 특정 IP 화이트리스트 관리할 때 유용
- 사용 사례
  - 극한의 성능이 필요한 서비스
  - HTTP가 아닌 TCP/UDP 기반 트래픽
  - 예: 금융 거래, 게임 서버, VoIP, 실시간 스트리밍

### NLB 의 타겟 그룹(대상 그룹)

- 🔹 트래픽을 보낼 수 있는 대상
  - EC2 인스턴스
    - NLB가 직접 EC2 인스턴스로 TCP/UDP 트래픽 전달
  - IP 주소
    - 프라이빗 IP만 허용 (퍼블릭 IP 불가)
    - 온프레미스 서버나 VPC 피어링된 리소스 대상으로 지정 가능
  - Application Load Balancer (ALB)
    - NLB가 ALB로 트래픽을 전달할 수도 있음
    - 흔히 하이브리드 구조에서 사용됨 (예: TCP 연결은 NLB → ALB → EC2)

✅ 쉽게 설명

- NLB Target Group = NLB가 트래픽을 보내는 목적지 묶음
- 대상은 EC2, 프라이빗 IP, ALB가 될 수 있음
- 각 Target Group에 헬스 체크를 설정해서, 비정상 서버는 자동 제외

⸻

🔹 Health Check (헬스 체크)
• 지원 프로토콜: TCP, HTTP, HTTPS
• 즉, 단순 TCP 연결 확인부터, HTTP 상태 코드 기반 확인까지 가능

## 🔹 4. Gateway Load Balancer (GWLB) – 2020

- 동작 계층: Layer 3(네트워크 계층, IP 패킷 레벨) 에서 동작
- 트래픽을 패킷 단위로 처리

- 주요 기능
  - Transparent Network Gateway
    - 모든 트래픽이 지나가는 단일 입구/출구 역할
    - 사용자 입장에서는 투명하게 동작 (별도 설정 필요 없음)
  - Load Balancer 기능
    - 들어온 트래픽을 여러 보안 어플라이언스로 분산 전달
- 프로토콜
  - GENEVE 프로토콜(포트 6081) 사용
  - 네트워크 가상화에 최적화된 프로토콜 → 다양한 보안 장비와 연동 가능

### GWLB 의 타겟 그룹

- EC2 instances
- IP Address - must be private IPs

✅ 쉽게 설명

- GWLB = 네트워크 보안 장비 전용 로드 밸런서
- 방화벽, IDS/IPS 같은 장비들을 자동으로 확장/분산해서 트래픽을 처리
- 일반 사용자는 존재를 의식하지 않아도, 모든 트래픽은 GWLB → 보안 장치 → 다시 GWLB 흐름으로 지나감

⸻

📌 공통 특징

- 모든 로드 밸런서는 내부(Private) 또는 외부(Public) 용도로 설정 가능
- 최신 세대(ALB, NLB, GWLB)는 더 많은 기능과 유연성을 제공
- 따라서 신규 구축 시 ALB/NLB 사용이 권장됨

⸻

✅ 쉽게 요약

- CLB (구식, 이제 잘 안 씀)
- ALB (웹/앱 트래픽, 7계층)
- NLB (초고성능, 4계층, 금융/게임)
- GWLB (보안 장비용, 3계층)

## 📌 AWS Load Balancer 종류별 지원 프로토콜

| 로드 밸런서                         | 세대      | 지원 계층                     | 지원 프로토콜                            |
| ----------------------------------- | --------- | ----------------------------- | ---------------------------------------- |
| **CLB (Classic Load Balancer)**     | v1 (2009) | L4, L7                        | **TCP, SSL (보안 TCP), HTTP, HTTPS**     |
| **ALB (Application Load Balancer)** | v2 (2016) | L7 (애플리케이션 계층)        | **HTTP, HTTPS, WebSocket**               |
| **NLB (Network Load Balancer)**     | v2 (2017) | L4 (네트워크 계층, 전송 계층) | **TCP, TLS (보안 TCP), UDP**             |
| **GWLB (Gateway Load Balancer)**    | v2 (2020) | L3 (네트워크 계층)            | **IP 패킷 (GENEVE 프로토콜, 포트 6081)** |

---

✅ 요약

- **CLB**: 구세대, TCP/HTTP 둘 다 지원 (범용)
- **ALB**: 웹/애플리케이션 전용 (HTTP/HTTPS/WebSocket)
- **NLB**: 초고성능 네트워크 트래픽 (TCP/UDP/TLS)
- **GWLB**: 보안 장비용, IP 패킷 단위 전달 (GENEVE)
