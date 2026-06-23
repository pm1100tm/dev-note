# 🚀 Route53 3rd Part Registar

AWS Route 53을 사용할 때 반드시 Route 53에서 도메인을 구매해야 하는 것은 아닙니다.

도메인은 GoDaddy, Namecheap, Gabia, Cloudflare Registrar 같은 외부 도메인 등록기관에서
구매하고, DNS 관리는 Amazon Route 53에서 할 수 있습니다.

즉, 다음과 같은 구성이 가능합니다.

```shell
도메인 구매: GoDaddy, Namecheap, Gabia 등
DNS 관리: Amazon Route 53
서버 연결: EC2, ALB, CloudFront, S3 등
```

**핵심은 Domain Registrar와 DNS Service Provider는 서로 다른 개념이라는 점입니다.**

<br>

## 📚 기본 개념 정리

| 구분                 | 의미                                                     | 예시                                              |
| -------------------- | -------------------------------------------------------- | ------------------------------------------------- |
| Domain Registrar     | 도메인을 구매하고 소유권을 관리하는 곳                   | GoDaddy, Namecheap, Gabia, Cloudflare Registrar   |
| DNS Service Provider | 도메인의 DNS 레코드를 관리하는 곳                        | Amazon Route 53, Cloudflare DNS, Google Cloud DNS |
| Hosted Zone          | Route 53에서 특정 도메인의 DNS 레코드를 관리하는 공간    | example.com Hosted Zone                           |
| NS Record            | "이 도메인의 DNS 관리는 이 네임서버들이 담당한다"는 정보 | ns-123.awsdns-45.com                              |

<br>

## 📚 Domain Registrar란?

Domain Registrar는 도메인을 구매하고 소유권을 관리하는 곳입니다.

예를 들어 example.com이라는 도메인을 구매하려면 도메인 등록기관을 통해 구매해야 합니다.

대표적인 Domain Registrar는 다음과 같습니다.

| Registrar                 | 설명                                                 |
| ------------------------- | ---------------------------------------------------- |
| GoDaddy                   | 해외에서 많이 사용하는 도메인 등록기관               |
| Namecheap                 | 비교적 저렴한 가격으로 많이 사용하는 도메인 등록기관 |
| Gabia                     | 국내에서 많이 사용하는 도메인 등록기관               |
| Cloudflare Registrar      | Cloudflare에서 제공하는 도메인 등록 서비스           |
| Amazon Route 53 Registrar | AWS에서 제공하는 도메인 등록 서비스                  |

도메인을 구매한 곳은 도메인의 소유권, 만료일, 갱신, 등록자 정보 등을 관리합니다.

<br>

## 📚 DNS Service Provider란?

DNS Service Provider는 도메인의 DNS 레코드를 관리하는 서비스입니다.

예를 들어 example.com이라는 도메인을 어떤 서버로 연결할지 설정하는 곳입니다.

```
example.com      → EC2 서버
www.example.com  → CloudFront
api.example.com  → Application Load Balancer
```

이런 연결 정보를 DNS 레코드라고 합니다.

대표적인 DNS Service Provider는 다음과 같습니다.

| DNS Service Provider | 설명                               |
| -------------------- | ---------------------------------- |
| Amazon Route 53      | AWS의 관리형 DNS 서비스            |
| Cloudflare DNS       | Cloudflare에서 제공하는 DNS 서비스 |
| Google Cloud DNS     | GCP에서 제공하는 DNS 서비스        |
| Gabia DNS            | Gabia에서 제공하는 DNS 관리 기능   |
| GoDaddy DNS          | GoDaddy에서 제공하는 DNS 관리 기능 |

중요한 점은 도메인을 구매한 곳과 DNS를 관리하는 곳이 반드시 같을 필요는 없다는 것입니다.

<br>

### Domain Registrar와 DNS Service는 다르다

예를 들어 도메인을 Gabia에서 구매했다고 해보겠습니다

```
Domain Registrar: Gabia
DNS Service Provider: Gabia DNS
```

이렇게 Gabia에서 도메인 구매와 DNS 관리를 모두 할 수도 있습니다.

하지만 AWS 인프라를 사용한다면 다음과 같이 구성할 수도 있습니다.

```
Domain Registrar: Gabia
DNS Service Provider: Amazon Route 53
```

즉, 도메인 소유권은 Gabia에서 관리하고, DNS 레코드는 Route 53에서 관리하는 구조입니다.

<br>

## 3rd Party Registrar에서 구매한 도메인을 Route 53에서 관리하는 방법

외부 도메인 등록기관에서 구매한 도메인을 Route 53에서 DNS 관리하려면 다음 순서로 진행합니다.

| 단계 | 작업                                  | 설명                                                             |
| ---- | ------------------------------------- | ---------------------------------------------------------------- |
| 1    | Route 53 Hosted Zone 생성             | Route 53에서 도메인에 대한 DNS 관리 공간을 만든다                |
| 2    | Route 53 NS Record 확인               | Hosted Zone 생성 시 자동으로 생성되는 Name Server를 확인한다     |
| 3    | Registrar 사이트에서 Name Server 변경 | 기존 Registrar의 Name Server를 Route 53 Name Server로 교체한다   |
| 4    | Route 53에서 DNS Record 관리          | A, CNAME, Alias, MX, TXT 등의 DNS 레코드를 Route 53에서 관리한다 |

### 1. Route 53에서 Hosted Zone 생성

먼저 AWS Route 53에서 Hosted Zone을 생성합니다.

예를 들어 example.com 도메인을 Route 53에서 관리하려면 다음과 같이 Public Hosted Zone을 생성합니다.

```
Domain name: example.com
Type: Public hosted zone
```

Hosted Zone을 생성하면 Route 53은 자동으로 NS 레코드를 생성합니다.

```
example.com.  NS  ns-123.awsdns-45.com.
example.com.  NS  ns-678.awsdns-90.net.
example.com.  NS  ns-111.awsdns-22.org.
example.com.  NS  ns-333.awsdns-44.co.uk.
```

이 NS 레코드는 “example.com의 DNS 관리는 이 Route 53 네임서버들이 담당한다”는 의미입니다.

### 2. 3rd Party Registrar에서 Name Server 변경

이제 도메인을 구매한 사이트로 이동합니다.

예를 들어 도메인을 GoDaddy에서 구매했다면 GoDaddy 관리 화면에서 Name Server 설정을 변경합니다.

기존 Name Server가 다음과 같다고 가정해보겠습니다.

```
ns1.godaddy.com
ns2.godaddy.com
```

이를 Route 53에서 제공한 Name Server로 변경합니다.

```
ns-123.awsdns-45.com
ns-678.awsdns-90.net
ns-111.awsdns-22.org
ns-333.awsdns-44.co.uk
```

이 작업을 하면 앞으로 example.com에 대한 DNS 조회는 Route 53으로 전달됩니다.

### 3. Route 53에서 DNS Record 관리

Name Server 변경이 완료되면 이제 DNS 레코드는 Route 53 Hosted Zone에서 관리합니다.

예를 들어 EC2 서버의 퍼블릭 IP로 도메인을 연결하려면 A 레코드를 만들 수 있습니다.

```
example.com.  A  13.209.10.20
```

www.example.com을 루트 도메인으로 연결하려면 CNAME을 사용할 수 있습니다.

```
www.example.com.  CNAME  example.com.
```

AWS 서비스와 연결할 때는 Alias 레코드를 자주 사용합니다.

```
example.com.      A Alias  CloudFront Distribution
api.example.com.  A Alias  Application Load Balancer
```

Route 53의 Alias 레코드는 AWS 서비스와 연결할 때 편리합니다. 특히 CloudFront, ALB, S3 정적 웹사이트 호스팅과 연결할 때 많이 사용됩니다.

### 전체 흐름

외부 도메인 등록기관에서 도메인을 구매하고 Route 53을 DNS 서비스로 사용하는 전체 흐름은 다음과 같습니다.

```
사용자 브라우저
      |
      v
example.com 접속
      |
      v
Registrar에 등록된 Name Server 확인
      |
      v
Route 53 Name Server로 DNS 질의 전달
      |
      v
Route 53 Hosted Zone에서 DNS Record 조회
      |
      v
EC2 / ALB / CloudFront / S3 등 AWS 리소스로 연결
```

<br>

## 📚 왜 Route 53을 DNS 서비스로 사용할까?

도메인 등록기관에서도 기본 DNS 기능을 제공합니다.

예를 들어 Gabia에서 도메인을 구매하면 Gabia DNS에서 A 레코드, CNAME 레코드 등을 설정할 수 있습니다.

그럼에도 AWS 인프라를 사용한다면 Route 53을 사용하는 것이 편리할 수 있습니다.

| Route 53을 사용하는 이유            | 설명                                                                |
| ----------------------------------- | ------------------------------------------------------------------- |
| AWS 서비스와 연동이 쉽다            | ALB, CloudFront, S3 등에 Alias 레코드로 쉽게 연결할 수 있다         |
| 고가용성 DNS 서비스이다             | AWS의 글로벌 DNS 인프라를 사용한다                                  |
| 다양한 라우팅 정책을 지원한다       | Weighted, Latency, Failover, Geolocation 라우팅 등을 사용할 수 있다 |
| Health Check와 연동 가능하다        | 장애 발생 시 다른 엔드포인트로 Failover할 수 있다                   |
| IaC 관리가 쉽다                     | Terraform, CloudFormation, CDK로 DNS 설정을 코드화할 수 있다        |
| AWS 리소스 중심으로 관리하기 편하다 | 인프라가 AWS에 있다면 DNS도 AWS에서 관리하는 것이 운영상 편하다     |

<br>

### Route 53에서 자주 사용하는 DNS Record

| Record Type  | 의미                                     | 사용 예시                                 |
| ------------ | ---------------------------------------- | ----------------------------------------- |
| A Record     | 도메인을 IPv4 주소로 연결                | example.com → 13.209.10.20                |
| AAAA Record  | 도메인을 IPv6 주소로 연결                | example.com → 2001:db8::1                 |
| CNAME Record | 도메인을 다른 도메인 이름으로 연결       | www.example.com → example.com             |
| NS Record    | 도메인의 네임서버 정보                   | ns-123.awsdns-45.com                      |
| MX Record    | 메일 서버 정보                           | Google Workspace, Microsoft 365 메일 연결 |
| TXT Record   | 텍스트 검증 정보                         | SPF, DKIM, 도메인 소유권 인증             |
| Alias Record | AWS 리소스에 연결하는 Route 53 전용 기능 | CloudFront, ALB, S3 연결                  |

## 👉 주의할 점

- Name Server를 변경한다고 해서 즉시 전 세계에 반영되는 것은 아닙니다.
  DNS 변경은 전파 시간이 필요합니다.
- 보통 몇 분 안에 반영되기도 하지만, 경우에 따라 몇 시간에서 최대 24~48시간 정도 걸릴 수 있습니다.
- 또한 Name Server를 변경한 뒤에는 기존 Registrar의 DNS 설정이 더 이상 적용되지 않습니다.

예를 들어 Gabia에서 DNS 레코드를 관리하다가 Name Server를 Route 53으로 변경하면, 이후에는 Gabia DNS에 있는 A 레코드나 CNAME 레코드가 아니라 Route 53 Hosted Zone에 있는 레코드가 사용됩니다.

따라서 Name Server를 바꾸기 전에 기존 DNS 레코드를 Route 53에 미리 옮겨두는 것이 좋습니다.

<br>

## 작업 순서

| 순서 | 작업                              | 설명                                                                |
| ---- | --------------------------------- | ------------------------------------------------------------------- |
| 1    | 기존 DNS 레코드 확인              | Gabia, GoDaddy, Namecheap 등에 설정된 A, CNAME, MX, TXT 레코드 확인 |
| 2    | Route 53 Hosted Zone 생성         | example.com용 Public Hosted Zone 생성                               |
| 3    | 기존 DNS 레코드를 Route 53에 복사 | 기존 A, CNAME, MX, TXT 등을 Route 53 Hosted Zone에 동일하게 생성    |
| 4    | Route 53 레코드 검토              | 누락된 레코드가 없는지 확인                                         |
| 5    | Registrar에서 Name Server 변경    | 기존 NS를 Route 53 NS로 교체                                        |
| 6    | DNS 전파 확인                     | dig, nslookup 등으로 정상 조회되는지 확인                           |

### 예시

기존 Gabia DNS에 다음 레코드가 있다고 해보겠습니다.

| Type  | Name            | Value                                |
| ----- | --------------- | ------------------------------------ |
| A     | example.com     | 13.209.10.20                         |
| CNAME | www.example.com | example.com                          |
| MX    | example.com     | 10 aspmx.l.google.com                |
| TXT   | example.com     | v=spf1 include:\_spf.google.com ~all |

그러면 Route 53 Hosted Zone에도 동일하게 만들어둡니다.

| Type  | Name            | Value                                |
| ----- | --------------- | ------------------------------------ |
| A     | example.com     | 13.209.10.20                         |
| CNAME | www.example.com | example.com                          |
| MX    | example.com     | 10 aspmx.l.google.com                |
| TXT   | example.com     | v=spf1 include:\_spf.google.com ~all |

그다음 Registrar에서 Name Server를 Route 53 NS로 변경합니다.

```
기존 Name Server:
ns1.gabia.co.kr
ns2.gabia.co.kr

변경 후 Name Server:
ns-123.awsdns-45.com
ns-678.awsdns-90.net
ns-111.awsdns-22.org
ns-333.awsdns-44.co.uk
```

### 잘못하면 생기는 문제

기존 DNS 레코드를 Route 53에 옮기지 않고 Name Server만 바꾸면 다음 문제가 생길 수 있습니다.

| 누락된 레코드 | 발생 가능한 문제                           |
| ------------- | ------------------------------------------ |
| A Record      | example.com 접속 불가                      |
| CNAME Record  | www.example.com, api.example.com 접속 불가 |
| MX Record     | 이메일 수신 불가                           |
| TXT Record    | SPF, DKIM, Google 인증, 도메인 인증 실패   |
| DKIM Record   | 메일이 스팸 처리될 가능성 증가             |
| CAA Record    | SSL 인증서 발급 제한 문제 발생 가능        |

특히 운영 중인 서비스라면 A, CNAME만 보면 안 됩니다.

메일을 사용하고 있다면 MX, TXT, DKIM 레코드도 반드시 옮겨야 합니다.
