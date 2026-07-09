# Amazon CloudFront Caching

CloudFront는 기본적으로 URL, Header, Query String, Cookie 등을 기준으로 캐시 키를 만들 수 있습니다.

예를 들어 아래 요청은 서로 다른 캐시로 취급될 수 있습니다.

```
/products?page=1
/products?page=2
```

Query String을 캐시 키에 포함하면 다른 응답으로 캐시됩니다.

## TTL

TTL은 캐시 유지 시간입니다.

- Minimum TTL: 최소 캐시 시간
- Default TTL: 기본 캐시 시간
- Maximum TTL: 최대 캐시 시간

예를 들어 이미지 파일은 오래 캐시해도 됩니다.

```
/logo.png → TTL 1일 또는 7일
```

하지만 자주 바뀌는 HTML은 짧게 잡는 경우가 많습니다.

```
/index.html → TTL 짧게 설정
```

## Invalidation

이미 캐시된 파일을 강제로 삭제하는 기능입니다.

예를 들어 index.html이 바뀌었는데 CloudFront가 예전 파일을 계속 보여주면 Invalidation을 실행합니다.

다만 실무에서는 모든 파일을 자주 Invalidation하기보다, 정적 파일명에 해시를 붙이는 방식이 좋습니다.

```
app.a1b2c3.js
style.f9e8d7.css
```

이렇게 하면 새 배포 때 파일명이 바뀌므로 기존 캐시와 충돌하지 않습니다.
