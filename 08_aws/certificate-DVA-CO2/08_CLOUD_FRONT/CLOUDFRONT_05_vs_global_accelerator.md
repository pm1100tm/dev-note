# CloudFront vs Global Accelerator

둘 다 글로벌 성능 개선 서비스라 헷갈릴 수 있습니다.

| 구분        | CloudFront                   | Global Accelerator          |
| ----------- | ---------------------------- | --------------------------- |
| 목적        | 웹 콘텐츠 캐싱 및 CDN        | 네트워크 경로 최적화        |
| 주 대상     | HTTP/HTTPS 콘텐츠            | TCP/UDP 애플리케이션        |
| 캐싱        | 가능                         | 불가능                      |
| 사용 예시   | 이미지, JS, CSS, API 캐싱    | 게임 서버, VoIP, TCP 서비스 |
| 연결 대상   | S3, ALB, EC2, API Gateway 등 | ALB, NLB, EC2 등            |
| 핵심 키워드 | Edge Cache, CDN              | Anycast IP, Global Routing  |
