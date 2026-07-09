# Amazon CloudFront Origin

CloudFront는 여러 종류의 Origin과 연결할 수 있습니다.

| Origin         | 사용 예시                                         |
| -------------- | ------------------------------------------------- |
| S3             | 이미지, 정적 웹사이트, 파일 다운로드              |
| ALB            | Spring Boot, Django, Node.js 같은 웹 애플리케이션 |
| EC2            | 직접 운영하는 웹 서버                             |
| API Gateway    | API 응답 캐싱 및 글로벌 배포                      |
| MediaPackage   | 영상 스트리밍                                     |
| 외부 HTTP 서버 | AWS 외부의 서버도 Origin으로 사용 가능            |

---

## S3 + CloudFront 예시

정적 파일 배포에서 가장 자주 나오는 구조입니다.

```
사용자
  ↓
Route 53
  ↓
CloudFront
  ↓
S3 Bucket
```

예를 들어 React 빌드 파일을 S3에 올리고 CloudFront로 배포할 수 있습니다.

```
S3 Bucket
- index.html
- main.js
- style.css
- logo.png
```

사용자는 S3 URL이 아니라 CloudFront URL로 접근합니다.

```
https://d123456abcdef.cloudfront.net
```

또는 Route 53 Alias를 연결해서 커스텀 도메인으로 접근할 수 있습니다.

```
https://www.example.com
```

## CloudFront vs S3 직접 접근

| 구분                | S3 직접 접근             | CloudFront 사용                        |
| ------------------- | ------------------------ | -------------------------------------- |
| 속도                | S3 리전에 따라 지연 발생 | 사용자와 가까운 Edge Location에서 응답 |
| 캐싱                | 제한적                   | 강력한 CDN 캐싱                        |
| 보안                | Public Bucket 위험       | OAC로 S3 비공개 가능                   |
| DDoS 보호           | 상대적으로 약함          | AWS Shield, AWS WAF 연동 가능          |
| 커스텀 도메인 HTTPS | 설정이 비교적 복잡       | CloudFront + ACM으로 쉽게 구성         |
| 글로벌 서비스       | 부적합할 수 있음         | 매우 적합                              |
