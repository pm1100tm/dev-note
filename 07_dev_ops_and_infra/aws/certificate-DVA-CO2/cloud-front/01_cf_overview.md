# Amazon CloudFront Overview

Amazon CloudFront는 AWS에서 제공하는 CDN 서비스입니다.

CDN은 Content Delivery Network의 약자입니다.

쉽게 말하면, 사용자가 웹사이트나 파일에 접근할 때 원본 서버까지 매번 가지 않고, 사용자와 가까운 위치의
캐시 서버에서 콘텐츠를 받아가도록 해주는 서비스입니다.

<br>

## 📚 CDN이란?

CDN은 전 세계 여러 지역에 콘텐츠를 캐싱해두고, 사용자에게 가장 가까운 위치에서 콘텐츠를 전달하는 네트워크입니다.

예를 들어 원본 서버가 서울 리전에 있다고 가정해보겠습니다.

미국 사용자가 이미지를 요청할 때마다 서울 서버까지 요청이 오면 응답 시간이 느릴 수 있습니다.

CloudFront를 사용하면 미국 사용자에게 가까운 Edge Location에서 캐싱된 이미지를 전달할 수 있습니다.

```
미국 사용자
   |
   v
가까운 CloudFront Edge Location
   |
   v
캐시된 콘텐츠 응답
```

캐시에 없는 경우에는 원본 서버로 요청합니다.

```
미국 사용자

   |

   v

CloudFront Edge Location

   |

   v

캐시에 없음

   |

   v

Origin Server 요청

   |

   v

콘텐츠 받아와서 캐싱 후 사용자에게 응답
```

<br>

## 📚 CloudFront의 핵심 역할

| 기능             | 설명                                                          |
| ---------------- | ------------------------------------------------------------- |
| CDN 제공         | 전 세계 Edge Location을 통해 콘텐츠를 빠르게 전달             |
| 캐싱             | 정적 파일, 이미지, 동영상, API 응답 등을 Edge Location에 저장 |
| 읽기 성능 개선   | Origin Server까지 가지 않고 가까운 캐시에서 응답              |
| 사용자 경험 개선 | 지연 시간 감소로 페이지 로딩 속도 개선                        |
| Origin 부하 감소 | 동일한 요청을 CloudFront가 대신 처리                          |
| 보안 강화        | DDoS 방어, AWS Shield, AWS WAF와 연동 가능                    |

<br>

## 🧐 Edge Location이란?

Edge Location은 CloudFront가 콘텐츠를 캐싱하고 사용자에게 전달하는 AWS의 전 세계 캐시 지점입니다.

사용자가 콘텐츠를 요청하면 CloudFront는 사용자의 위치와 가까운 Edge Location으로 요청을 라우팅합니다.

```
사용자
   |
   v
가장 가까운 Edge Location
   |
   v
캐시된 콘텐츠 반환
```

Edge Location이 많을수록 사용자와 가까운 곳에서 응답할 가능성이 높아지고, 그만큼 지연 시간이 줄어듭니다.

<br>

## 🧐 Origin이란?

CloudFront에서 Origin은 원본 콘텐츠가 저장되어 있는 서버입니다.

| Origin 유형               | 예시                                         |
| ------------------------- | -------------------------------------------- |
| S3 Bucket                 | 정적 이미지, HTML, CSS, JavaScript 파일 저장 |
| Application Load Balancer | 백엔드 애플리케이션 서버 연결                |
| EC2 Instance              | 직접 운영하는 웹 서버                        |
| API Gateway               | API 요청 처리                                |
| MediaPackage              | 동영상 스트리밍 원본                         |
| External HTTP Server      | AWS 외부 서버                                |

예를 들어 다음과 같은 리소스가 Origin이 될 수 있습니다.

```
User → CloudFront → Origin
```

<br>

## 📚 CloudFront 동작 흐름

예를 들어 사용자가 logo.png 이미지를 요청한다고 가정해보겠습니다.

```
https://d123.cloudfront.net/logo.png
```

CloudFront의 동작 흐름은 다음과 같습니다.

```
1. 사용자가 CloudFront 도메인으로 콘텐츠 요청
2. CloudFront가 사용자와 가까운 Edge Location으로 요청 전달
3. Edge Location에 캐시된 콘텐츠가 있는지 확인
4. 캐시에 있으면 바로 사용자에게 응답
5. 캐시에 없으면 Origin으로 요청
6. Origin에서 콘텐츠를 받아옴
7. Edge Location에 콘텐츠를 캐싱
8. 사용자에게 콘텐츠 응답
```

<br>

## 📚 Cache Hit와 Cache Miss

CloudFront에서 중요한 개념이 Cache Hit와 Cache Miss입니다.

| 구분       | 의미                                               | 결과                             |
| ---------- | -------------------------------------------------- | -------------------------------- |
| Cache Hit  | Edge Location에 요청한 콘텐츠가 이미 캐싱되어 있음 | Origin까지 가지 않고 빠르게 응답 |
| Cache Miss | Edge Location에 요청한 콘텐츠가 없음               | Origin으로 요청 후 캐싱하고 응답 |

예를 들어 첫 번째 사용자가 이미지를 요청하면 아직 캐시에 없을 수 있습니다.

```
첫 번째 요청: Cache Miss
User → CloudFront → Origin
```

이후 같은 지역의 다른 사용자가 같은 이미지를 요청하면 Edge Location에서 바로 응답할 수 있습니다.

```
두 번째 요청: Cache Hit
User → CloudFront
```

<br>

## 📚 CloudFront가 성능을 개선하는 이유

CloudFront는 사용자의 요청을 가까운 Edge Location에서 처리합니다.

그래서 다음과 같은 효과가 있습니다.

| 개선 요소               | 설명                                      |
| ----------------------- | ----------------------------------------- |
| 지연 시간 감소          | 사용자와 가까운 캐시 서버에서 응답        |
| 다운로드 속도 향상      | 전 세계 네트워크 최적화 사용              |
| Origin 서버 부하 감소   | 반복 요청을 CloudFront가 대신 처리        |
| 글로벌 사용자 경험 개선 | 여러 국가의 사용자에게 안정적인 성능 제공 |

예를 들어 한국에 Origin Server가 있고 미국, 유럽, 일본 사용자가 접속한다면 CloudFront를 사용했을 때 각 사용자와 가까운 Edge Location에서 콘텐츠를 전달할 수 있습니다.

<br>

## 📚 CloudFront와 보안

CloudFront는 성능뿐만 아니라 보안 측면에서도 중요합니다.

| 보안 기능             | 설명                                                           |
| --------------------- | -------------------------------------------------------------- |
| DDoS 보호             | 전 세계 Edge Network를 통해 대규모 트래픽을 흡수               |
| AWS Shield 연동       | 기본적으로 AWS Shield Standard 보호 적용                       |
| AWS WAF 연동          | SQL Injection, XSS, IP 차단 등 웹 공격 방어                    |
| HTTPS 지원            | ACM 인증서를 사용해 TLS 적용 가능                              |
| Origin Access Control | S3 Origin을 직접 공개하지 않고 CloudFront를 통해서만 접근 가능 |
| Geo Restriction       | 특정 국가에서의 접근 허용 또는 차단 가능                       |

<br>

## 📚 CloudFront와 AWS Shield

AWS Shield는 DDoS 보호 서비스입니다.

CloudFront는 AWS Shield와 통합되어 DDoS 공격에 대한 기본 보호를 받을 수 있습니다.

```
사용자 또는 공격 트래픽

        |

        v

CloudFront Edge Network

        |

        v

AWS Shield가 DDoS 트래픽 완화

        |

        v

정상 요청만 Origin으로 전달
```

AWS Shield에는 두 가지 종류가 있습니다.

- AWS Shield Standard: 기본 제공 DDoS 보호
- AWS Shield Advanced: 유료 고급 DDoS 보호, 비용 보호 및 상세 대응 지원

CloudFront를 사용하면 Origin Server가 직접 인터넷 트래픽을 받는 구조보다 DDoS에 더 강한 구조를 만들 수 있습니다.

<br>

## 📚 CloudFront와 AWS WAF

AWS WAF는 Web Application Firewall입니다.

CloudFront와 WAF를 연동하면 HTTP/HTTPS 요청을 검사해서 악성 요청을 차단할 수 있습니다.

예를 들어 다음과 같은 규칙을 만들 수 있습니다.

| WAF 규칙           | 설명                                        |
| ------------------ | ------------------------------------------- |
| IP 차단            | 특정 IP 주소에서 오는 요청 차단             |
| Rate-based Rule    | 일정 시간 동안 과도한 요청을 보내는 IP 차단 |
| SQL Injection 차단 | SQL Injection 패턴 탐지                     |
| XSS 차단           | Cross-Site Scripting 공격 탐지              |
| 특정 국가 차단     | 특정 국가에서 오는 요청 제한                |

```
User
  |
  v
CloudFront
  |
  v
AWS WAF 검사
  |
  v
Origin Server
```

정확히는 WAF가 CloudFront 배포에 연결되어 CloudFront 레벨에서 요청을 검사한다고 보면 됩니다.

<br>

## CloudFront를 사용하는 대표적인 경우

| 사용 사례          | 설명                                                       |
| ------------------ | ---------------------------------------------------------- |
| 정적 웹사이트 배포 | S3 + CloudFront로 React, Vue, Next.js Static Export 배포   |
| 이미지/동영상 캐싱 | 이미지, CSS, JS, 동영상 파일을 빠르게 제공                 |
| API 가속           | API Gateway, ALB 앞에 CloudFront를 두어 글로벌 요청 최적화 |
| 보안 강화          | WAF, Shield와 연동하여 웹 공격 및 DDoS 방어                |
| HTTPS 적용         | CloudFront에 ACM 인증서를 연결하여 HTTPS 제공              |
| S3 비공개 접근     | OAC를 사용해 S3 Bucket을 직접 공개하지 않음                |

### 예시 구조: S3 + CloudFront

정적 웹사이트를 S3에 올리고 CloudFront로 배포하는 구조입니다.

```
User
  |
  v
CloudFront
  |
  v
S3 Bucket
```

이 구조에서는 사용자가 S3에 직접 접근하지 않고 CloudFront를 통해 접근합니다.

보안을 강화하려면 S3 Bucket은 private으로 두고, CloudFront OAC를 통해서만 접근하도록 설정합니다.

```
User
  |
  v
CloudFront
  |
  v
OAC
  |
  v
Private S3 Bucket
```

### 예시 구조: ALB + CloudFront

백엔드 API 서버 앞에 CloudFront를 둘 수도 있습니다.

```
User
  |
  v
CloudFront
  |
  v
Application Load Balancer
  |
  v
EC2 / ECS / EKS
```

이 구조의 장점은 다음과 같습니다.

| 장점                    | 설명                                       |
| ----------------------- | ------------------------------------------ |
| 글로벌 사용자 성능 개선 | CloudFront Edge Network를 통해 요청 최적화 |
| HTTPS 처리              | CloudFront에서 TLS 처리 가능               |
| WAF 적용                | CloudFront 앞단에서 보안 규칙 적용         |
| DDoS 방어               | CloudFront + Shield를 통한 방어            |
| Origin 보호             | ALB에 직접 접근하는 트래픽을 줄일 수 있음  |
