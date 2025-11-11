# 🚀 Route53 Hosted Zone

Hosted Zone 은 하나의 도메인(예: example.com)과 그 하위 도메인들에 대한
모든 DNS Record를 담는 컨테이너(Container) 입니다.

**즉, 하나의 Hosted Zone = “그 도메인의 DNS 설정집” 입니다.**

```shell
Hosted Zone: example.com
├─ A record → 13.124.55.10
├─ CNAME record → www → example.com
├─ MX record → mail.example.com
└─ TXT record (SPF 인증)
```

## 🧭 Hosted Zone 의 종류

- Public Hosted Zone
- Private Hosted Zone

### Public Hosted Zone

| 구분           | 설명                                                       |
| -------------- | ---------------------------------------------------------- |
| 목적           | 전 세계 인터넷 사용자에게 DNS 응답                         |
| 도메인 형태    | app.mycompany.com 같은 공개 도메인                         |
| 접근 가능 범위 | 전 세계 모든 DNS 리졸버 (공개 DNS)                         |
| 사용 시나리오  | 웹사이트, CloudFront, S3 정적 사이트 등 인터넷 공개 리소스 |
| 예시           | api.mypublicdomain.com                                     |
| 요금           | $0.50 / 월 (Hosted Zone 당)                                |
| 연결 대상      | Public DNS 리졸버                                          |

### Private Hosted Zone

| 구분           | 설명                                                   |
| -------------- | ------------------------------------------------------ |
| 목적           | AWS 내부 VPC 안에서만 DNS 응답                         |
| 도메인 형태    | app.company.internal 같은 내부 도메인                  |
| 접근 가능 범위 | 연결된 VPC 내의 인스턴스만                             |
| 사용 시나리오  | 내부 마이크로서비스, 사내 API, VPC 내부 DB 접속 도메인 |
| 예시           | db.company.internal                                    |
| 요금           | $0.50 / 월 (Hosted Zone 당)                            |
| 연결 대상      | AWS VPC 내 리졸버 (AmazonProvidedDNS)                  |

## 💡 예시 시나리오

- Public Hosted Zone
  - → www.myapp.com 을 전 세계 사용자에게 노출
  - → A Record 로 ALB 또는 CloudFront 연결
- Private Hosted Zone
  - → db.internal.myapp.com 을 VPC 내 EC2 나 Lambda에서만 사용
  - → RDS, ECS 같은 내부 리소스에 보안 접근 용도

## 비용

- Hosted Zone 유지비: $0.50 / 월 (도메인 당)
- 쿼리 요금: DNS 조회량에 따라 별도 과금 (약 $0.40 / 백만 쿼리)

## 🕒 TTL (Time To Live)

- TTL 은 “DNS 리졸버가 해당 Record 를 캐싱하는 시간(초)” 을 의미합니다.
- TTL이 지나면 DNS 리졸버가 다시 Route 53에 질의하여 최신 정보를 받습니다.

### ⏰ TTL 값의 효과

| TTL 값                 | 장점                               | 단점                                     |
| :--------------------- | :--------------------------------- | :--------------------------------------- |
| **High TTL (예: 24h)** | 🔹 쿼리 트래픽 감소 → 비용 절감    | ⚠️ 레코드 변경 전파 느림, 오래된 값 유지 |
| **Low TTL (예: 60s)**  | ⚡ 변경 즉시 적용 → 빠른 전환 가능 | 💰 쿼리 트래픽 증가 → 비용 상승          |

💡 운영 TIP :

- 테스트 / 배포 직전 : TTL 낮게 (60 ~ 300초)
- 운영 안정기 : TTL 길게 (1 시간 ~ 하루)

### 🧠 Alias Record 예외

- Route 53의 Alias Record (예: CloudFront or ALB 연결) 은
  AWS가 자동으로 TTL을 관리하므로 직접 TTL 설정 불필요.

### 📘 TTL 적용 예시

```shell
Name: www.example.com
Type: A
Value: 13.124.55.10
TTL: 300   # 5분간 리졸버 캐싱
```

- 사용자가 처음 요청 → Route 53 쿼리 → 응답 캐싱
- TTL(300초) 동안 다른 요청은 DNS 조회 없이 캐시 사용
- 5분 후 자동으로 갱신 필요

## ✅ 정리 요약

| 항목                    | 설명                                                                  |
| :---------------------- | :-------------------------------------------------------------------- |
| **Hosted Zone**         | 도메인과 서브도메인의 모든 DNS 레코드를 담는 컨테이너                 |
| **Public Hosted Zone**  | 🌐 인터넷 전체를 대상으로 DNS 해석을 수행                             |
| **Private Hosted Zone** | 🔒 VPC 내부에서만 DNS 해석을 수행                                     |
| **비용**                | 💵 $0.50 / Hosted Zone / 월                                           |
| **TTL (Time To Live)**  | ⏱️ DNS 결과를 리졸버가 캐싱하는 시간 (짧을수록 변경 빠름 / 비용 증가) |
| **Alias Record 특징**   | 🔗 AWS 리소스 연결용, TTL 자동 관리 (수동 입력 불필요)                |
