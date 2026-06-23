# Route 53 Alias vs CNAME 정리

## 1. 핵심 결론

Route 53에서 **Alias Record**와 **CNAME Record**는 겉으로 보면 둘 다 “도메인명을 다른 도메인명으로 연결하는 것”처럼 보입니다.

예를 들어 다음과 같은 상황입니다.

```text
mycoolcompany.com → my-elb-1234567890.us-west-2.elb.amazonaws.com
```

이렇게 보면 Alias도 CNAME과 비슷해 보입니다.

하지만 중요한 차이는 **DNS 응답 방식**입니다.

```text
CNAME = 다른 도메인 이름을 응답한다.
Alias = Route 53이 AWS 리소스의 실제 IP를 찾아 A/AAAA 레코드처럼 응답한다.
```

즉, Alias는 설정 화면에서는 CNAME처럼 보이지만, 실제 DNS 응답에서는 CNAME처럼 동작하지 않습니다.

---

## 2. CNAME과 Alias의 핵심 차이

| 구분 | CNAME | Alias |
|---|---|---|
| 정체 | DNS 표준 레코드 | Route 53 전용 기능 |
| 루트 도메인 사용 가능 여부 | 불가능 | 가능 |
| 응답 방식 | 다른 도메인 이름을 반환 | 최종 IP 주소를 A/AAAA처럼 반환 |
| 사용 대상 | 일반 도메인 이름 | 주로 AWS 리소스 |
| 예시 | `www.example.com → app.example.com` | `example.com → ALB / CloudFront / S3` |
| DNS 표준 여부 | 표준 DNS 기능 | AWS Route 53 기능 |
| 추가 DNS 조회 | 보통 필요 | Route 53이 내부적으로 처리 |
| AWS 리소스 연결 | 가능하지만 루트 도메인에는 부적합 | 적합 |

---

## 3. CNAME이란?

CNAME은 **Canonical Name**의 약자입니다.

CNAME은 어떤 도메인을 다른 도메인의 별칭으로 만드는 DNS 레코드입니다.

예를 들어 다음과 같이 설정할 수 있습니다.

```text
www.mycoolcompany.com  CNAME  my-elb-1234567890.us-west-2.elb.amazonaws.com
```

이 의미는 다음과 같습니다.

```text
www.mycoolcompany.com 으로 요청이 오면
my-elb-1234567890.us-west-2.elb.amazonaws.com 을 다시 조회하세요.
```

즉, CNAME은 최종 IP 주소를 바로 응답하는 것이 아니라 **다른 도메인 이름을 응답**합니다.

흐름은 다음과 같습니다.

```text
사용자: www.mycoolcompany.com IP 알려줘
DNS: my-elb-1234567890.us-west-2.elb.amazonaws.com 을 조회하세요
사용자 또는 Resolver: 그러면 ELB 도메인의 IP는 무엇인가요?
DNS: 1.2.3.4
```

정리하면 다음 구조입니다.

```text
www.mycoolcompany.com
        |
        v
my-elb-1234567890.us-west-2.elb.amazonaws.com
        |
        v
실제 IP 주소
```

---

## 4. CNAME은 왜 루트 도메인에 사용할 수 없는가?

루트 도메인은 다음과 같이 서브도메인이 없는 도메인입니다.

```text
mycoolcompany.com
example.com
swd.com
```

루트 도메인은 **apex domain**, **zone apex**, **root domain**이라고도 부릅니다.

CNAME은 이런 루트 도메인에는 사용할 수 없습니다.

잘못된 예시는 다음과 같습니다.

```text
mycoolcompany.com  CNAME  my-elb-1234567890.us-west-2.elb.amazonaws.com
```

그 이유는 루트 도메인에는 보통 여러 중요한 DNS 레코드가 함께 존재해야 하기 때문입니다.

예를 들어 루트 도메인에는 다음과 같은 레코드들이 필요할 수 있습니다.

```text
mycoolcompany.com  NS   ns-123.awsdns-45.com
mycoolcompany.com  SOA  ns-123.awsdns-45.com
mycoolcompany.com  MX   aspmx.l.google.com
mycoolcompany.com  TXT  v=spf1 include:_spf.google.com ~all
```

그런데 DNS 표준상 **CNAME이 설정된 이름에는 다른 레코드를 같이 둘 수 없습니다.**

따라서 아래와 같은 구성은 불가능합니다.

```text
mycoolcompany.com  CNAME  my-elb-1234567890.us-west-2.elb.amazonaws.com
mycoolcompany.com  MX     aspmx.l.google.com
mycoolcompany.com  TXT    v=spf1 include:_spf.google.com ~all
```

CNAME은 “이 이름은 완전히 다른 이름의 별칭이다”라는 의미이기 때문에, 같은 이름에 MX, TXT, NS, SOA 같은 다른 레코드를 함께 둘 수 없습니다.

그래서 루트 도메인에는 CNAME을 사용할 수 없습니다.

---

## 5. Alias란?

Alias는 Amazon Route 53에서 제공하는 특수한 레코드 기능입니다.

겉보기에는 다음과 같이 설정합니다.

```text
mycoolcompany.com  A Alias  my-elb-1234567890.us-west-2.elb.amazonaws.com
```

표면적으로 보면 CNAME과 비슷해 보입니다.

```text
mycoolcompany.com → my-elb-1234567890.us-west-2.elb.amazonaws.com
```

하지만 중요한 차이는 **Alias는 DNS 응답에서 CNAME을 반환하지 않는다**는 점입니다.

Alias는 Route 53이 내부적으로 AWS 리소스의 주소를 찾아서 **A 레코드 또는 AAAA 레코드처럼 IP 주소를 응답**합니다.

흐름은 다음과 같습니다.

```text
사용자: mycoolcompany.com IP 알려줘
Route 53: 이 도메인은 ALB를 가리키는 Alias구나
Route 53: ALB의 현재 IP를 확인해서 응답해야겠다
Route 53: 1.2.3.4
```

사용자 입장에서는 다음처럼 보입니다.

```text
mycoolcompany.com  A  1.2.3.4
```

즉, Alias는 설정할 때는 ELB 도메인명을 넣지만, 응답할 때는 CNAME처럼 “다른 이름을 다시 조회하세요”라고 하지 않습니다.

Route 53이 대신 ELB의 IP를 찾아서 응답합니다.

---

## 6. 가장 중요한 차이: 가리키는 대상이 아니라 응답 방식이 다르다

Alias와 CNAME이 헷갈리는 이유는 둘 다 설정 화면에서는 도메인명을 다른 도메인명에 연결하는 것처럼 보이기 때문입니다.

하지만 DNS 응답 관점에서는 다릅니다.

| 관점 | CNAME | Alias |
|---|---|---|
| 설정 화면 | 도메인명을 다른 도메인명에 연결 | 도메인명을 AWS 리소스 DNS 이름에 연결 |
| 실제 DNS 응답 | 다른 도메인명을 반환 | IP 주소를 반환 |
| 루트 도메인 사용 | 불가능 | 가능 |
| 대표 사용 예 | `www.example.com → external.example.net` | `example.com → ALB / CloudFront` |

예를 들어 ELB에 연결한다고 해보겠습니다.

### CNAME 방식

```text
www.mycoolcompany.com  CNAME  my-elb-1234567890.us-west-2.elb.amazonaws.com
```

DNS 응답은 이런 식입니다.

```text
www.mycoolcompany.com is an alias for my-elb-1234567890.us-west-2.elb.amazonaws.com
my-elb-1234567890.us-west-2.elb.amazonaws.com has address 1.2.3.4
```

### Alias 방식

```text
mycoolcompany.com  A Alias  my-elb-1234567890.us-west-2.elb.amazonaws.com
```

DNS 응답은 이런 식입니다.

```text
mycoolcompany.com has address 1.2.3.4
```

즉, Alias는 설정할 때는 ELB 도메인명을 넣지만 실제 응답은 A 레코드처럼 IP 주소를 반환합니다.

---

## 7. 왜 루트 도메인에는 Alias를 써야 하는가?

예를 들어 사용자가 다음 도메인으로 접속하게 하고 싶다고 가정합니다.

```text
mycoolcompany.com
```

그리고 이 도메인을 ALB에 연결하고 싶습니다.

ALB 주소는 다음과 같습니다.

```text
my-elb-1234567890.us-west-2.elb.amazonaws.com
```

이때 CNAME을 쓰고 싶을 수 있습니다.

```text
mycoolcompany.com  CNAME  my-elb-1234567890.us-west-2.elb.amazonaws.com
```

하지만 이 방식은 루트 도메인에서는 사용할 수 없습니다.

루트 도메인에는 NS, SOA 같은 필수 레코드가 존재해야 하고, CNAME은 같은 이름에 다른 레코드와 공존할 수 없기 때문입니다.

그래서 Route 53에서는 Alias를 사용합니다.

```text
mycoolcompany.com  A Alias  my-elb-1234567890.us-west-2.elb.amazonaws.com
```

이렇게 하면 루트 도메인이 ALB를 가리키면서도 DNS 표준을 위반하지 않습니다.

왜냐하면 실제 DNS 응답은 CNAME이 아니라 A 레코드처럼 처리되기 때문입니다.

---

## 8. www 서브도메인은 CNAME도 가능하다

반면 다음과 같은 `www` 서브도메인은 CNAME을 사용할 수 있습니다.

```text
www.mycoolcompany.com
```

예를 들어 다음 구성은 가능합니다.

```text
www.mycoolcompany.com  CNAME  my-elb-1234567890.us-west-2.elb.amazonaws.com
```

`www.mycoolcompany.com`에는 보통 NS, SOA, MX 같은 루트 도메인 필수 레코드가 없기 때문입니다.

하지만 Route 53에서는 `www`에도 Alias를 사용할 수 있습니다.

```text
www.mycoolcompany.com  A Alias  my-elb-1234567890.us-west-2.elb.amazonaws.com
```

AWS 리소스를 대상으로 한다면 보통 Alias를 쓰는 것이 더 자연스럽습니다.

---

## 9. 실무에서 어떻게 선택하면 되는가?

| 상황 | 추천 |
|---|---|
| 루트 도메인을 ALB에 연결 | Alias |
| 루트 도메인을 CloudFront에 연결 | Alias |
| 루트 도메인을 S3 static website에 연결 | Alias |
| `www`를 ALB에 연결 | Alias 또는 CNAME |
| `www`를 CloudFront에 연결 | Alias 또는 CNAME |
| `www`를 외부 SaaS 도메인에 연결 | CNAME |
| AWS 리소스를 대상으로 연결 | Alias 우선 |
| AWS 외부 도메인을 대상으로 연결 | CNAME |

---

## 10. 예시로 비교

### 루트 도메인

루트 도메인은 다음과 같습니다.

```text
mycoolcompany.com
```

ALB로 연결하려면 Alias를 사용합니다.

```text
mycoolcompany.com  A Alias  my-elb-1234567890.us-west-2.elb.amazonaws.com
```

CNAME은 루트 도메인에 사용할 수 없습니다.

```text
mycoolcompany.com  CNAME  my-elb-1234567890.us-west-2.elb.amazonaws.com
```

### www 서브도메인

`www` 서브도메인은 다음과 같습니다.

```text
www.mycoolcompany.com
```

ALB로 연결하려면 CNAME도 가능합니다.

```text
www.mycoolcompany.com  CNAME  my-elb-1234567890.us-west-2.elb.amazonaws.com
```

Route 53 Alias도 가능합니다.

```text
www.mycoolcompany.com  A Alias  my-elb-1234567890.us-west-2.elb.amazonaws.com
```

AWS 리소스를 가리킨다면 Alias가 더 자연스럽습니다.

---

## 11. Alias 대상이 될 수 있는 대표 AWS 리소스

Route 53 Alias는 주로 AWS 리소스를 대상으로 사용합니다.

| AWS 리소스 | Alias 사용 가능 여부 | 사용 예시 |
|---|---|---|
| Application Load Balancer | 가능 | `example.com → ALB` |
| Network Load Balancer | 가능 | `api.example.com → NLB` |
| CloudFront Distribution | 가능 | `example.com → CloudFront` |
| S3 Static Website Endpoint | 가능 | `example.com → S3 static website` |
| API Gateway Custom Domain | 가능 | `api.example.com → API Gateway` |
| Elastic Beanstalk Environment | 가능 | `app.example.com → Elastic Beanstalk` |
| 다른 Route 53 Record | 가능 | `example.com → 다른 Record` |

---

## 12. 머릿속에 이렇게 정리하기

CNAME은 다음과 같이 이해하면 됩니다.

```text
CNAME:
"이 도메인은 다른 도메인의 별칭입니다. 그쪽을 다시 조회하세요."
```

Alias는 다음과 같이 이해하면 됩니다.

```text
Alias:
"이 도메인은 AWS 리소스를 가리킵니다. Route 53이 대신 실제 IP를 찾아서 A/AAAA처럼 응답합니다."
```

더 짧게 정리하면 다음과 같습니다.

```text
CNAME = 다른 이름으로 넘김
Alias = Route 53이 대신 해석해서 IP로 응답
```

---

## 13. 시험용 핵심 정리

| 시험 포인트 | 설명 |
|---|---|
| CNAME은 루트 도메인에 사용할 수 없다 | `example.com`에는 CNAME 불가 |
| Alias는 루트 도메인에 사용할 수 있다 | `example.com → ALB` 가능 |
| Alias는 Route 53 전용 기능이다 | 일반 DNS 표준 레코드가 아니다 |
| Alias는 A/AAAA처럼 응답한다 | CNAME처럼 다른 이름만 반환하지 않는다 |
| AWS 리소스 연결에는 Alias가 적합하다 | ALB, CloudFront, S3 등에 사용 |
| CNAME은 서브도메인에 주로 사용한다 | `www.example.com`, `api.example.com` 등 |
| 루트 도메인에 AWS 리소스를 연결하려면 Alias를 사용한다 | 시험에서 자주 나오는 포인트 |

---

## 14. 한 줄 요약

Alias도 설정 화면에서는 CNAME처럼 도메인명을 다른 도메인명에 연결하는 것처럼 보입니다.

하지만 실제 DNS 응답에서는 Route 53이 AWS 리소스의 실제 IP를 찾아 **A/AAAA 레코드처럼 응답**합니다.

따라서 **CNAME은 루트 도메인에 사용할 수 없지만, Alias는 루트 도메인에 사용할 수 있습니다.**

