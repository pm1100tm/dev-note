# Amazon CloudFront Overview

Amazon CloudFront는 AWS의 CDN(Content Delivery Network) 서비스입니다.

사용자가 S3, ALB, EC2, API 서버 같은 원본 서버에 직접 접근하는 대신, 전 세계에 분산된
Edge Location에 먼저 접근하게 해서 콘텐츠를 더 빠르게 전달합니다.
AWS 공식 문서 기준으로 CloudFront는 정적/동적 웹 콘텐츠를 전 세계 Edge Location을 통해 낮은
지연 시간으로 전달합니다.

시험에서는 아래 키워드가 나오면 CloudFront를 먼저 의심해야 합니다.

- CDN
- Edge Location / Points of Presence
- Read performance 향상
- Global user experience 개선
- Origin 부하 감소
- DDoS protection
- AWS Shield / AWS WAF 연동

## 기존 구조

```
사용자
  ↓
CloudFront Edge Location
  ↓ Cache Hit이면 바로 응답
  ↓ Cache Miss이면 Origin으로 요청
Origin
  - S3
  - ALB
  - EC2
  - API Gateway
  - 외부 HTTP 서버
```

## 요청 흐름

```
미국 사용자
   v
가까운 CloudFront Edge Location
   v
캐시된 콘텐츠 응답
```

캐시에 없는 경우에는 원본 서버로 요청합니다.

```
미국 사용자
   v
CloudFront Edge Location
   v
캐시에 없음
   v
Origin Server 요청
   v
콘텐츠 받아와서 캐싱 후 사용자에게 응답
```

Cache Hit이면 S3, ALB, EC2 등의 Origin까지 요청이 가지 않습니다.
Cache Miss이면 Origin까지 요청이 전달되고, 응답을 받은 뒤 Edge Location에 캐싱합니다.

### 📚 CloudFront 동작 흐름

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

## 주요 개념 정리

| 개념          | 설명                                                                        |
| ------------- | --------------------------------------------------------------------------- |
| CDN           | 콘텐츠를 사용자와 가까운 위치에서 제공하는 네트워크                         |
| Edge Location | 사용자의 요청을 가장 가까이에서 처리하는 CloudFront 캐시 위치               |
| Origin        | CloudFront가 콘텐츠를 가져오는 원본 서버                                    |
| Distribution  | CloudFront의 설정 단위(배포 단위)                                           |
| Cache Key     | CloudFront가 캐시 객체를 구분하는 기준                                      |
| Cache Policy  | Header, Cookie, Query String, TTL 등을 캐시 키에 포함할지 정하는 정책       |
| Cache Hit     | Edge Location에 캐시가 있어 Origin까지 가지 않고 바로 응답하는 경우         |
| Cache Miss    | 캐시가 없어 Origin으로 요청한 후 응답하는 경우                              |
| TTL           | 캐시를 얼마나 오래 유지할지 결정하는 시간(Time To Live)                     |
| Invalidation  | 이미 캐시된 콘텐츠를 강제로 무효화하여 Origin에서 다시 가져오도록 하는 기능 |

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

## 📚 CloudFront의 장점

| 장점                 | 설명                                                          |
| -------------------- | ------------------------------------------------------------- |
| 읽기 성능 향상       | 정적 파일, 이미지, JS, CSS 등을 Edge Location에서 빠르게 제공 |
| 사용자 경험 개선     | 사용자와 가까운 위치에서 응답하므로 지연 시간이 줄어듦        |
| Origin 부하 감소     | 캐시된 요청은 S3, ALB, EC2까지 가지 않음                      |
| 글로벌 서비스에 적합 | 전 세계 사용자에게 안정적으로 콘텐츠 제공                     |
| DDoS 방어            | AWS Shield Standard와 통합되어 기본 DDoS 보호 제공            |
| WAF 연동             | SQL Injection, XSS, IP 차단 등 웹 공격 방어 가능              |
| HTTPS 지원           | ACM 인증서를 연결해 HTTPS 제공 가능                           |

---

## 한 줄 요약

CloudFront는 사용자와 가까운 Edge Location에 콘텐츠를 캐싱해서 응답 속도를 높이고,
원본 서버 부하를 줄이며, 보안까지 강화하는 AWS CDN 서비스입니다.

---

## 용어 정리

### 🧐 Edge Location이란?

- Edge Location은 CloudFront가 콘텐츠를 캐싱하고 사용자에게 전달하는 AWS의 전 세계 캐시 지점
- 사용자가 콘텐츠를 요청하면 CloudFront는 사용자의 위치와 가까운 Edge Location으로 요청을 라우팅
- Edge Location이 많을수록 사용자와 가까운 곳에서 응답할 가능성이 높아지고, 그만큼 지연 시간이 줄어듬

### 🧐 Origin이란?

CloudFront에서 Origin은 원본 콘텐츠가 저장되어 있는 서버입니다.

| Origin 유형               | 예시                                         |
| ------------------------- | -------------------------------------------- |
| S3 Bucket                 | 정적 이미지, HTML, CSS, JavaScript 파일 저장 |
| Application Load Balancer | 백엔드 애플리케이션 서버 연결                |
| Network Load Balancer     | TCP 기반 애플리케이션 연결                   |
| EC2 Instance              | 직접 운영하는 웹 서버                        |
| API Gateway               | API 요청 처리                                |
| MediaPackage              | 동영상 스트리밍 원본                         |
| S3 Website Endpoint       | S3 정적 웹사이트 호스팅 엔드포인트           |
| External HTTP Server      | AWS 외부 서버                                |

S3를 Origin으로 사용할 때는 두 가지 형태를 구분해야 합니다.

| 구분                 | 설명                                                                       |
| -------------------- | -------------------------------------------------------------------------- |
| S3 REST API Endpoint | 일반적인 S3 Origin. OAC로 private bucket 접근 제어 가능                    |
| S3 Website Endpoint  | S3 정적 웹사이트 호스팅을 먼저 활성화해야 하며 Custom HTTP Origin처럼 동작 |

---

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
| Signed URL            | 특정 파일에 제한된 시간 동안 접근 허용                         |
| Signed Cookies        | 여러 파일에 제한된 접근 허용                                   |

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

| 장점                    | 설명                                              |
| ----------------------- | ------------------------------------------------- |
| 글로벌 사용자 성능 개선 | CloudFront Edge Network를 통해 요청 최적화        |
| HTTPS 처리              | CloudFront에서 TLS 처리 가능                      |
| WAF 적용                | CloudFront 앞단에서 보안 규칙 적용                |
| DDoS 방어               | CloudFront + Shield를 통한 방어                   |
| Origin 보호             | ALB에 직접 접근하는 트래픽을 줄일 수 있음         |
| 정적/동적 분리          | 정적 파일은 S3, API는 ALB로 Path 기반 라우팅 가능 |
