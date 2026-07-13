# Amazon CloudFront Caching

CloudFront의 캐시는 각 Edge Location에 저장됩니다.
사용자가 같은 콘텐츠를 반복 요청하면 Origin까지 가지 않고 Edge Location에서 응답할 수 있습니다.

CloudFront 캐싱 문제는 대부분 아래 질문으로 정리됩니다.

> 이 값을 Cache Key에 포함할 것인가, 아니면 Origin으로만 전달할 것인가?

목표는 Cache Hit Ratio를 높여 Origin 요청 수를 줄이는 것입니다.

## Cache Key

Cache Key는 CloudFront가 Edge Location에 저장된 캐시 객체를 구분하는 기준입니다.

기본적으로 CloudFront는 다음 값을 기준으로 객체를 구분합니다.

- Hostname
- URL path

예를 들어 기본 설정에서는 아래 두 요청이 같은 캐시 객체로 취급될 수 있습니다.

```text
/products?id=1
/products?id=2
```

하지만 Query String을 Cache Key에 포함하면 서로 다른 객체로 캐시됩니다.

```text
/products?id=1 -> cache object A
/products?id=2 -> cache object B
```

Cache Key에 값을 많이 포함할수록 캐시는 세밀해지지만 Cache Hit Ratio는 낮아질 수 있습니다.

## Cache Policy

Cache Policy는 어떤 값을 Cache Key에 포함할지 정하는 정책입니다.

즉, 응답 결과가 달라지는 기준만 Cache Policy에 넣어야 합니다.

| 항목 | 선택지 | 의미 |
| --- | --- | --- |
| HTTP Headers | None / Whitelist | 지정한 Header만 Cache Key에 포함 |
| Cookies | None / Whitelist / Include All-Except / All | Cookie를 Cache Key에 포함 |
| Query Strings | None / Whitelist / Include All-Except / All | Query String을 Cache Key에 포함 |
| TTL | Minimum / Default / Maximum | 캐시 유지 시간 제어 |

Cache Key에 포함한 Header, Cookie, Query String은 Origin Request에도 자동으로 포함됩니다.

## Origin Request Policy

Origin Request Policy는 Origin으로 전달할 값은 정하지만, Cache Key에는 포함하지 않는 정책입니다.

즉, Origin에는 필요하지만 캐시를 나눌 기준은 아닌 값을 전달할 때 사용합니다.

| 항목 | 선택지 | 의미 |
| --- | --- | --- |
| HTTP Headers | None / Whitelist / All viewer headers options | Origin으로 전달할 Header 선택 |
| Cookies | None / Whitelist / All | Origin으로 전달할 Cookie 선택 |
| Query Strings | None / Whitelist / All | Origin으로 전달할 Query String 선택 |
| CloudFront Headers | 선택 가능 | CloudFront가 추가한 Header를 Origin으로 전달 |
| Custom Headers | 선택 가능 | Origin 요청에 사용자 정의 Header 추가 |

## Cache Policy vs Origin Request Policy

| 구분 | Cache Policy | Origin Request Policy |
| --- | --- | --- |
| 목적 | 캐시 객체를 나누는 기준 설정 | Origin으로 전달할 값 설정 |
| Cache Key 포함 | 포함함 | 포함하지 않음 |
| Cache Hit Ratio 영향 | 직접 영향 있음 | 직접 영향 없음 |
| Origin 전달 | Cache Key에 포함된 값은 자동 전달 | 전달만 하고 캐시는 나누지 않음 |
| 사용 예 | 언어별/디바이스별/Query String별 응답 캐싱 | 로그, 인증, Origin 내부 처리용 값 전달 |

판단 기준은 단순합니다.

| 질문 | 선택 |
| --- | --- |
| 값에 따라 응답 본문이 달라지는가? | Cache Policy |
| Origin에는 필요하지만 캐시를 나눌 필요는 없는가? | Origin Request Policy |

## Header 예시

언어별 콘텐츠를 제공하는 경우입니다.

```text
Accept-Language: ko-KR
Accept-Language: en-US
```

언어별로 다른 HTML을 캐싱해야 하므로 Cache Policy에 `Accept-Language` Header를 포함합니다.

반면 `User-Agent`를 단순히 Origin 로그에 남기기 위한 목적이라면 Cache Key에 포함할 필요가 없습니다.
이 경우 Origin Request Policy로만 전달합니다.

| 설정 | 결과 |
| --- | --- |
| Header None | Header를 Cache Key에 포함하지 않음. 캐시 효율 좋음 |
| Header Whitelist | 지정한 Header만 Cache Key에 포함하고 Origin으로 전달 |

## Query String 예시

이미지 크기가 Query String으로 달라지는 경우입니다.

```text
/image/cat.jpg?size=small
/image/cat.jpg?size=large
```

이때 `size`를 Cache Key에 포함하지 않으면 작은 이미지와 큰 이미지가 같은 캐시 객체로 취급될 수 있습니다.
정답은 Cache Policy에 `size` Query String을 포함하는 것입니다.

반대로 아래처럼 추적용 Query String은 응답 결과에 영향을 주지 않습니다.

```text
/blog/post.html?utm_source=newsletter
```

이런 값은 Cache Key에 포함하지 않는 것이 좋습니다.
필요하다면 Origin Request Policy로 Origin에만 전달합니다.

| 설정 | 결과 |
| --- | --- |
| None | Query String을 Cache Key에 포함하지 않고 Origin에도 전달하지 않음 |
| Whitelist | 지정한 Query String만 Cache Key에 포함하고 전달 |
| Include All-Except | 지정한 Query String만 제외하고 나머지를 포함하고 전달 |
| All | 모든 Query String을 Cache Key에 포함하고 전달. 캐시 효율이 가장 나쁠 수 있음 |

## Cookie 예시

정적 파일 요청에 세션 Cookie를 Cache Key로 포함하면 사용자마다 캐시가 분리됩니다.

```text
/static/app.js
Cookie: session_id=user-a

/static/app.js
Cookie: session_id=user-b
```

정적 파일의 응답이 사용자별로 달라지지 않는다면 Cookie를 Cache Key에 넣지 않아야 합니다.

반대로 사용자별 HTML을 캐싱하려는 경우라면 Cookie를 포함할 수 있지만, 개인정보와 캐시 오염 위험을
함께 검토해야 합니다.

## 요구사항별 선택

| 요구사항 | 선택 |
| --- | --- |
| `Accept-Language`에 따라 한국어/영어 HTML이 달라짐 | Cache Policy |
| `size=small`, `size=large`에 따라 이미지가 달라짐 | Cache Policy |
| 로그인 세션 Cookie에 따라 사용자별 화면이 달라짐 | Cache Policy 또는 캐싱 비활성화 검토 |
| Origin 로그 분석을 위해 `User-Agent` 전달 필요 | Origin Request Policy |
| Origin에서 인증 검증을 위해 `Authorization` Header 필요 | Origin Request Policy |
| `utm_source`는 Origin에 전달하지만 캐시는 나누고 싶지 않음 | Origin Request Policy |

## TTL

TTL은 캐시 유지 시간입니다.

- Minimum TTL: 최소 캐시 시간
- Default TTL: 기본 캐시 시간
- Maximum TTL: 최대 캐시 시간

TTL은 CloudFront 설정으로 제어할 수도 있고, Origin의 HTTP Header로 제어할 수도 있습니다.

- `Cache-Control`
- `Expires`

예를 들어 이미지 파일은 오래 캐시해도 됩니다.

```text
/logo.png -> TTL 1일 또는 7일
```

하지만 자주 바뀌는 HTML은 짧게 잡는 경우가 많습니다.

```text
/index.html -> TTL 짧게 설정
```

## Invalidation

Invalidation은 이미 캐시된 파일을 강제로 삭제하는 기능입니다.

예를 들어 `index.html`이 바뀌었는데 CloudFront가 예전 파일을 계속 보여주면 Invalidation을 실행합니다.

전체 파일을 무효화할 수도 있고, 특정 경로만 무효화할 수도 있습니다.

```shell
aws cloudfront create-invalidation \
  --distribution-id E1234567890ABC \
  --paths "/*"
```

```shell
aws cloudfront create-invalidation \
  --distribution-id E1234567890ABC \
  --paths "/index.html" "/images/*"
```

다만 실무에서는 모든 파일을 자주 Invalidation하기보다, 정적 파일명에 해시를 붙이는 방식이 좋습니다.

```text
app.a1b2c3.js
style.f9e8d7.css
```

이렇게 하면 새 배포 때 파일명이 바뀌므로 기존 캐시와 충돌하지 않습니다.

## Cache Hit Ratio 높이는 방법

CloudFront는 Cache Key가 세밀해질수록 캐시 객체가 많이 나뉩니다.
그만큼 Cache Hit Ratio가 낮아질 수 있습니다.

| 방법 | 이유 |
| --- | --- |
| 불필요한 Header를 Cache Key에서 제외 | Header 값이 다양하면 캐시가 쪼개짐 |
| 응답에 영향 없는 Query String 제외 | 추적 파라미터 때문에 캐시가 분산되는 것을 방지 |
| 세션 Cookie를 정적 파일 캐시에 포함하지 않음 | 사용자별 캐시 분리를 방지 |
| 정적 콘텐츠와 동적 콘텐츠를 분리 | 정적 파일은 긴 TTL, API는 짧은 TTL 또는 no-cache 적용 |

정적 콘텐츠와 동적 콘텐츠는 별도 Cache Behavior 또는 별도 Distribution으로 분리하면 캐시 효율을 높이기 쉽습니다.

```text
/static/* -> S3 Origin, 긴 TTL
/api/*    -> ALB Origin, 필요한 Header/Cookie만 전달
```

## 시험 함정

| 문제 표현 | 정답 방향 |
| --- | --- |
| "Cache based on query string/header/cookie" | Cache Policy |
| "Forward values to origin without affecting cache key" | Origin Request Policy |
| "Maximize cache hit ratio" | 불필요한 Header/Cookie/Query String을 Cache Key에서 제외 |
| "Different response by language/device/query parameter" | Cache Policy에 해당 값 포함 |
| "Origin needs Authorization header" | Origin Request Policy. 단, 응답이 사용자별이면 캐싱 전략 재검토 |
| "All query strings included" | 캐시 효율이 낮아질 수 있음 |
| "Force refresh cached content before TTL expires" | Invalidation |

## 한 줄 요약

CloudFront 캐싱에서 Cache Policy는 "캐시를 어떻게 나눌지"를 정하고, Origin Request Policy는
"Origin에 무엇을 보낼지"를 정합니다.
