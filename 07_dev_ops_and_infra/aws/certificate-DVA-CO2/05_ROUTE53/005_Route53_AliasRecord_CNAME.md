# 🚀 Route53 AliasRecord

CNAME vs Alias Record 두 개념 모두 “도메인을 다른 이름으로 매핑하는 기능”이지만,
CNAME은 표준 DNS 기능, Alias는 AWS 전용 확장 기능입니다.

| 구분                  | **CNAME (Canonical Name)**                     | **Alias Record (AWS 확장 기능)**                  |
| :-------------------- | :--------------------------------------------- | :------------------------------------------------ |
| **정의**              | 하나의 도메인 이름 → 다른 도메인 이름으로 매핑 | AWS 리소스에 직접 연결하는 Route 53 전용 레코드   |
| **표준 여부**         | ✅ 표준 DNS 기능 (RFC 표준)                    | ⚙️ AWS 전용 기능 (Route 53에서만 지원)            |
| **지원 대상**         | 모든 외부 호스트 이름                          | AWS 리소스 (ELB, CloudFront, API Gateway 등)      |
| **Root Domain 지원**  | ❌ 불가 (예: `example.com`)                    | ✅ 가능 (`example.com` 포함)                      |
| **레코드 타입**       | CNAME                                          | A(IPv4) / AAAA(IPv6)                              |
| **TTL 설정**          | ⏱️ 사용자 지정 가능                            | ❌ AWS가 자동 관리                                |
| **비용**              | 💰 표준 DNS 쿼리 요금 부과                     | ✅ Route 53 Alias 쿼리는 무료                     |
| **IP 자동 갱신**      | ❌ 수동 변경 필요                              | ✅ AWS 리소스 IP 변경 시 자동 반영                |
| **Health Check 통합** | ❌ 직접 설정 필요                              | ✅ Route 53 네이티브 헬스체크 지원                |
| **적용 예시**         | `www.example.com → api.external.com`           | `example.com → myapp.us-east-1.elb.amazonaws.com` |

## 👀 CNAME Record 동작 방식

> “하나의 도메인 이름을 다른 도메인 이름으로 단순히 연결.”

예를 들어 👇

```shell
www.myapp.com  →  myapp123.us-east-1.elb.amazonaws.com
```

⚠️ 제한 사항

- CNAME은 루트 도메인(Zone Apex) 에서 사용 불가
  (예: myapp.com ❌, www.myapp.com ✅)
- 이유: 루트 도메인에는 NS, SOA 같은 다른 필수 레코드가 이미 존재하기 때문

## ⚙️ Alias Record 동작 방식 (Route 53 전용 기능)

> “도메인을 AWS 리소스 자체로 직접 매핑하는 Route 53의 확장 DNS 기능.”

예를 들어 👇

```shell
myapp.com → myapp-lb-123456789.us-east-1.elb.amazonaws.com
```

이때 Route 53은 ELB의 IP 주소를 자동으로 추적하고, DNS 응답을 A/AAAA Record 형식으로 반환합니다.

## ✅ Alias Record의 주요 특징

| 항목                 | 설명                                                       |
| :------------------- | :--------------------------------------------------------- |
| **Root Domain 지원** | ✅ 가능 example.com                                        |
| **IP 자동 추적**     | 🔄 ELB, CloudFront 등 AWS 리소스의 IP가 바뀌어도 자동 갱신 |
| **요금**             | 💰 Alias Record는 DNS 쿼리 무료                            |
| **TTL**              | ⏱️ 수동 설정 불가 – AWS가 자동으로 관리                    |
| **헬스체크**         | 🩺 Route 53 Health Check 통합 가능                         |
| **레코드 타입**      | ⚙️ 실제로는 A(IPv4) 또는 AAAA(IPv6) 레코드로 동작          |
| **AWS 리소스 전용**  | 🚫 EC2 인스턴스의 직접 DNS 이름에는 사용 불가              |

---

## 🧱 Alias Record Target 가능한 AWS 리소스

| 리소스                                      | 설명                                                                        |
| :------------------------------------------ | :-------------------------------------------------------------------------- |
| **Elastic Load Balancer (ALB / NLB / CLB)** | 🌐 가장 일반적인 대상 – 트래픽 분산용 로드 밸런서                           |
| **CloudFront Distribution**                 | 🚀 전 세계 CDN 연결용                                                       |
| **API Gateway Custom Domain**               | 🔗 REST / HTTP API 엔드포인트 연결                                          |
| **Elastic Beanstalk 환경**                  | 🧩 애플리케이션 환경 주소                                                   |
| **S3 Website Endpoint**                     | 🪣 정적 웹사이트 호스팅 (예: `mybucket.s3-website-us-east-1.amazonaws.com`) |
| **VPC Interface Endpoint**                  | 🔒 PrivateLink 기반 서비스 엔드포인트                                       |
| **AWS Global Accelerator**                  | ⚡ 글로벌 트래픽 라우팅 가속기                                              |
| **Route 53 Record (같은 Hosted Zone 내)**   | 🔁 다른 레코드로의 Alias 매핑 가능                                          |

> ❌ 주의: EC2 인스턴스의 Public DNS (ec2-xx-xx-xx-xx.compute.amazonaws.com) 는
> Alias 대상 불가

## 🧩 동작 예시 시나리오

### ALB에 연결하는 예시

| 구성              | 값                                                   |
| :---------------- | :--------------------------------------------------- |
| **도메인**        | 🌐 `myapp.com`                                       |
| **AWS 리소스**    | 🖥️ ALB: `myapp-lb-1234.us-east-2.elb.amazonaws.com`  |
| **Route 53 설정** | ⚙️ Alias Record (A Record) → 대상: 해당 ALB          |
| **결과**          | ✅ 사용자가 `myapp.com`에 접속하면 ALB로 자동 라우팅 |

- ✅ ALB IP 주소 변경 시에도 자동 추적 → 관리 불필요
- ✅ DNS 쿼리 비용 무료

### CloudFront에 연결하는 예시

| 구성              | 값                                               |
| :---------------- | :----------------------------------------------- |
| **도메인**        | 🌐 `cdn.myapp.com`                               |
| **Target**        | 🚀 CloudFront: `d111111abcdef8.cloudfront.net`   |
| **Route 53 설정** | ⚙️ Alias Record (A / AAAA)                       |
| **결과**          | ✅ `cdn.myapp.com` → CloudFront 배포로 자동 연결 |

### Root Domain 연결 비교 요약

| 도메인                                        | CNAME 사용 가능?           | Alias 사용 가능? |
| :-------------------------------------------- | :------------------------- | :--------------- |
| **example.com**                               | ❌ 불가능 (Zone Apex 제한) | ✅ 가능          |
| **[www.example.com](http://www.example.com)** | ✅ 가능                    | ✅ 가능          |
| **api.example.com**                           | ✅ 가능                    | ✅ 가능          |

---

💡 한 줄 요약

- CNAME → 일반 DNS 표준, 루트 도메인 불가, 외부 도메인용
- Alias → AWS 전용 확장 기능, 루트 도메인 지원, IP 자동 갱신, 무료
