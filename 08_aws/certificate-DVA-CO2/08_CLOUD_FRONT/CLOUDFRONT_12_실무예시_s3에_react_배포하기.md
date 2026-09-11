# CloudFront + React + S3

React 앱을 빌드하면 정적 파일이 생성됩니다.

```shell
npm run build
```

생성된 파일을 S3에 업로드합니다.

```shell
aws s3 sync dist/ s3://my-react-app-bucket --delete
```

CloudFront 캐시를 무효화합니다.

```shell
aws cloudfront create-invalidation \
  --distribution-id E1234567890ABC \
  --paths "/*"
```

구조는 다음과 같습니다.

```
사용자
  ↓
www.example.com
  ↓ Route 53 Alias
CloudFront
  ↓
S3 Bucket
```

## 기억해야 할 핵심 문장

CloudFront는 전 세계 Edge Location에 콘텐츠를 캐싱하여 사용자의 읽기 성능을 향상시키고,
Origin 부하를 줄이며, Shield와 WAF를 통해 보안을 강화하는 AWS CDN 서비스다.

시험에서는 아래 키워드가 나오면 CloudFront를 떠올리면 됩니다.

```
CDN
Edge Location
Cache
Low latency
Static content
Global users
S3 acceleration
DDoS protection
AWS WAF
Invalidation
OAC
```
