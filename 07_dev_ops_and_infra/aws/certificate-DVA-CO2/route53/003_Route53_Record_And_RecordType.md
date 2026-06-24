# 🚀 Route53 Record And RecordType

## DNS Record란?

- DNS Record 는 “특정 도메인 이름을 어떤 IP 주소나 다른 도메인으로 매핑할 것인지 정의하는
  설정 단위”입니다.
- 하나의 도메인(example.com)은 여러 개의 레코드 (Record Set)를 가질 수 있습니다.

## ⚙️ Route 53 Record의 구성요소

- **Name**

  - 도메인 또는 서브도메인 이름
  - example.com, api.example.com

- **Type**

  - DNS 레코드 유형
  - A, AAAA, CNAME, NS 등

- **Value**

  - 레코드가 가리키는 IP 또는 대상
  - 203.0.113.5 (IPv4) 또는 dualstack.myALB.amazonaws.com

- **Routing Policy**

  - 트래픽 라우팅 방식 (단순, 가중치, 지연, Failover 등)
  - Simple, Latency, Weighted 등

- **TTL (Time To Live)**
  - DNS 리졸버가 결과를 캐싱하는 시간(초)
  - 300 초 = 5분

## 📚 Route 53 지원 Record Type 요약

**기본 (Must Know)** :

| Record Type | 설명                                             |
| ----------- | ------------------------------------------------ |
| **A**       | 도메인 → IPv4 주소 매핑                          |
| **AAAA**    | 도메인 → IPv6 주소 매핑                          |
| **CNAME**   | 도메인 → 다른 도메인 이름으로 매핑               |
| **NS**      | Hosted Zone의 Name Server 정보 (DNS 권한 위임용) |

<br>

**고급 (Advanced)**:

| Record Type   | 설명                                                             |
| ------------- | ---------------------------------------------------------------- |
| **CAA**       | 어떤 CA가 SSL 인증서를 발급할 수 있는지 정의                     |
| **DS**        | DNSSEC 연계 정보 (Delegation Signer)                             |
| **MX**        | 이메일 서버 지정 (Mail Exchange)                                 |
| **NAPTR**     | 전화번호/VoIP 서비스용 매핑                                      |
| **PTR**       | 역방향 DNS 조회 (아이피 → 도메인)                                |
| **SOA**       | Zone 정보 시작 (Start of Authority) – 각 도메인의 기본 정보 포함 |
| **TXT / SPF** | 텍스트 정보 – 보통 도메인 인증 (SPF, DKIM)에 사용                |
| **SRV**       | 특정 서비스용 엔드포인트 정의 (예: `_voip._tcp.example.com`)     |

---

## 주요 Record Type 상세 설명

### A Record

- Hostname → IPv4 주소 매핑
- 가장 기본적인 DNS 레코드
- 예:

```shell
Name: example.com
Type: A
Value: 203.0.113.10
```

### AAAA Record

- Hostname → IPv6 주소 매핑
- 예:

```shell
Name: ipv6.example.com
Type: AAAA
Value: 2001:db8:1234::1
```

### CNAME Record (Canonical Name)

- 하나의 도메인을 다른 도메인 이름으로 연결
- Target은 반드시 A 또는 AAAA Record를 가진 도메인이어야 함
- ❌ Zone Apex (루트 도메인) 에서는 사용 불가
  - 즉, example.com에는 CNAME 불가
  - ✅ 대신 www.example.com에는 가능

```shell
Name: www.example.com
Type: CNAME
Value: example.com
```

### NS Record (Name Server)

- Hosted Zone의 권한 네임서버를 지정
- 도메인 등록기관(Registrar) 에서 이 NS 레코드를 사용해 DNS 권한 위임

```shell
Name: example.com
Type: NS
Value:
  ns-103.awsdns-15.com
  ns-921.awsdns-51.net
  ns-1206.awsdns-21.org
  ns-1818.awsdns-35.co.uk
```

### 🧭 TTL (Time To Live)

> TTL은 “DNS 리졸버가 해당 레코드를 얼마나 오래 캐싱할지”를 결정하는 값입니다.

- 짧을수록 (예: 60초) 변경 즉시 반영 가능하지만 DNS 트래픽 증가
- 길수록 (예: 86400 초 = 1일) 트래픽은 감소하지만 변경 반영이 느림

💡 운영 TIP:

- 테스트 or Failover 환경 → TTL 짧게 (60 ~ 300초)
- 안정된 프로덕션 환경 → TTL 길게 (1 시간 ~ 1 일)
