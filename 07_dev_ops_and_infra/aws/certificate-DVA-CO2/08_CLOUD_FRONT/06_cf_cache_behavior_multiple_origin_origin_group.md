# CloudFront Cache Behavior, Multiple Origin, Origin Group

CloudFront Distribution은 여러 Origin을 가질 수 있고, Cache Behavior를 사용해 요청 경로별로 다른 Origin에
라우팅할 수 있습니다.

시험에서는 아래 키워드가 나오면 Cache Behavior와 Multiple Origin을 떠올리면 됩니다.

- Path pattern
- `/api/*`
- `/images/*`
- `/*`
- Route to multiple origins
- Origin Group
- Failover

## Cache Behavior

Cache Behavior는 특정 URL path pattern에 대해 CloudFront 동작을 다르게 설정하는 규칙입니다.

예를 들어 정적 파일은 S3로 보내고, API 요청은 ALB로 보낼 수 있습니다.

```text
/images/* -> S3 Origin
/api/*    -> ALB Origin
/*        -> Default Cache Behavior
```

Cache Behavior별로 설정할 수 있는 대표 항목은 다음과 같습니다.

| 설정                   | 설명                                                          |
| ---------------------- | ------------------------------------------------------------- |
| Path pattern           | 어떤 URL 경로에 적용할지 결정                                 |
| Target Origin          | 해당 요청을 어느 Origin으로 보낼지 결정                       |
| Cache Policy           | 어떤 Header, Cookie, Query String을 Cache Key에 포함할지 결정 |
| Origin Request Policy  | Origin으로 전달할 Header, Cookie, Query String 결정           |
| Viewer Protocol Policy | HTTP/HTTPS 처리 방식 결정                                     |
| Allowed HTTP Methods   | GET, HEAD, OPTIONS, PUT, POST 등 허용 메서드 결정             |

## Default Cache Behavior

CloudFront Distribution에는 항상 Default Cache Behavior가 있습니다.

Default Cache Behavior의 path pattern은 항상 `/*`입니다.
추가 Cache Behavior가 매칭되지 않으면 마지막에 Default Cache Behavior가 처리합니다.

```text
1. /api/users       -> /api/* behavior
2. /images/logo.png -> /images/* behavior
3. /unknown         -> /* default behavior
```

시험 포인트는 "Default Cache Behavior는 항상 마지막에 처리되며 `/*`이다"입니다.

## Multiple Origin

Multiple Origin은 하나의 CloudFront Distribution에 여러 Origin을 연결하는 구성입니다.

예를 들어 다음처럼 구성할 수 있습니다.

```text
User
  |
  v
CloudFront Distribution
  |-- /api/*    -> Application Load Balancer
  |-- /images/* -> S3 Bucket
  |-- /*        -> S3 Bucket
```

| Path Pattern | Origin      | 예시            |
| ------------ | ----------- | --------------- |
| `/api/*`     | ALB         | 백엔드 API 요청 |
| `/images/*`  | S3          | 이미지 파일     |
| `/static/*`  | S3          | JS, CSS, font   |
| `/*`         | S3 또는 ALB | 기본 요청       |

이 구성은 정적 콘텐츠와 동적 콘텐츠를 분리할 때 자주 사용합니다.

## 정적 콘텐츠와 동적 콘텐츠 분리

캐시 효율을 높이려면 정적 콘텐츠와 동적 콘텐츠를 분리하는 것이 좋습니다.

```text
/static/* -> S3 Origin, 긴 TTL, Cookie/Header 최소화
/api/*    -> ALB Origin, 짧은 TTL 또는 캐싱 비활성화, 필요한 Header 전달
```

정적 콘텐츠는 사용자별로 달라지지 않기 때문에 Header, Cookie, Query String을 최대한 Cache Key에서
제외하는 것이 좋습니다.

동적 콘텐츠는 인증, 세션, Query String에 따라 응답이 달라질 수 있으므로 Cache Policy와 Origin Request
Policy를 더 신중하게 설정해야 합니다.

## Sign In Page 예시

로그인 페이지나 프리미엄 콘텐츠 영역에는 별도 Cache Behavior를 둘 수 있습니다.

```text
/login -> EC2 또는 ALB Origin
/*     -> S3 Origin
```

또는 인증 후 Signed Cookies를 발급해 여러 프리미엄 파일에 접근하도록 구성할 수 있습니다.

```text
1. 사용자가 /login 접근
2. 애플리케이션이 인증 수행
3. 애플리케이션이 Signed Cookies 발급
4. 사용자가 CloudFront의 보호된 콘텐츠 접근
```

## Origin Group

Origin Group은 고가용성을 위해 Primary Origin과 Secondary Origin을 묶는 기능입니다.

Primary Origin이 실패하면 CloudFront가 Secondary Origin으로 failover합니다.

```text
User
  |
  v
CloudFront
  |
  v
Origin Group
  |-- Primary Origin
  |-- Secondary Origin
```

| 구성 요소         | 설명                                      |
| ----------------- | ----------------------------------------- |
| Primary Origin    | 기본으로 요청을 보내는 Origin             |
| Secondary Origin  | Primary가 실패했을 때 사용하는 Origin     |
| Failover Criteria | 어떤 HTTP 상태 코드에서 failover할지 결정 |

## Origin Group Failover 흐름

```text
1. 사용자가 CloudFront로 요청
2. CloudFront가 Primary Origin으로 요청
3. Primary Origin이 실패 응답 반환
4. CloudFront가 같은 요청을 Secondary Origin으로 재시도
5. Secondary Origin이 정상 응답 반환
6. CloudFront가 사용자에게 응답
```

대표적인 failover 조건은 5xx 계열 Origin 장애입니다.

```text
Primary Origin -> 500 / 502 / 503 / 504
Secondary Origin으로 failover
```

## S3 + CloudFront Region-Level High Availability

S3 Origin을 여러 리전에 두고 Origin Group으로 묶으면 리전 수준의 고가용성 구조를 만들 수 있습니다.

```text
CloudFront
  |
  v
Origin Group
  |-- Primary: S3 bucket in us-east-1
  |-- Secondary: S3 bucket in ap-northeast-2
```

이때 두 S3 버킷의 콘텐츠 동기화는 별도로 구성해야 합니다.
예를 들어 S3 Replication을 사용할 수 있습니다.

## Cache Behavior vs Origin Group

| 구분        | Cache Behavior                                    | Origin Group                            |
| ----------- | ------------------------------------------------- | --------------------------------------- |
| 목적        | Path pattern에 따라 요청을 다른 Origin으로 라우팅 | Origin 장애 시 다른 Origin으로 failover |
| 기준        | URL path pattern                                  | Origin 장애 상태 코드                   |
| 예시        | `/api/*`는 ALB, `/static/*`은 S3                  | Primary S3 장애 시 Secondary S3 사용    |
| 핵심 키워드 | Multiple Origin, Path pattern                     | High availability, Failover             |

## 시험 함정

| 문제 표현                                                       | 정답 방향                        |
| --------------------------------------------------------------- | -------------------------------- |
| "Route `/api/*` to ALB and `/images/*` to S3"                   | Cache Behavior + Multiple Origin |
| "Default behavior"                                              | `/*`, 마지막에 처리              |
| "Different cache settings by URL path"                          | Cache Behavior                   |
| "Fail over to another origin when primary origin returns error" | Origin Group                     |
| "Increase high availability of CloudFront origin"               | Origin Group                     |
| "Static and dynamic content should use different cache rules"   | Separate Cache Behaviors         |

## 한 줄 요약

Cache Behavior는 path pattern에 따라 요청을 다른 Origin과 캐시 설정으로 라우팅하고, Origin Group은
Primary Origin 장애 시 Secondary Origin으로 failover하는 고가용성 기능입니다.
