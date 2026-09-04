# Apache 와 Nginx 의 차이점

## 한 줄 요약

- **Apache HTTP Server**는 오래된 웹 호스팅, PHP, `.htaccess`, 모듈 기반 처리에 강한
  전통적인 웹 서버입니다.
- **Nginx**는 정적 파일 처리, 리버스 프록시, 로드 밸런싱, 많은 동시 접속 처리에 강한 이벤트 기반
  웹 서버입니다.

실무에서 신규 백엔드 서비스를 구성한다면 보통은 아래 구조를 많이 사용합니다.

```text
Client
  -> Nginx
  -> Spring Boot / Node.js / Django / FastAPI / PHP-FPM
```

## 비교표

| 구분           | Apache HTTP Server                        | Nginx                                        |
| -------------- | ----------------------------------------- | -------------------------------------------- |
| 기본 구조      | 프로세스/스레드 기반                      | 이벤트 기반 비동기 구조                      |
| 동시 접속 처리 | 요청마다 프로세스나 스레드가 붙는 방식    | 적은 워커로 많은 연결 처리                   |
| 정적 파일 처리 | 가능하지만 Nginx보다 상대적으로 무거운 편 | 매우 강함                                    |
| 동적 처리      | PHP 모듈 등 서버 내부 처리 방식이 강함    | 보통 FastCGI, uWSGI, 프록시로 외부 앱에 전달 |
| 설정 방식      | `.htaccess` 지원                          | 중앙 설정 중심, `.htaccess` 없음             |
| 리버스 프록시  | 가능                                      | 매우 많이 사용됨                             |
| 로드 밸런싱    | 가능                                      | 간단하고 성능 좋음                           |
| 메모리 사용    | 접속 수 증가 시 상대적으로 증가           | 대량 연결에서 효율적                         |
| 레거시 호환성  | 오래된 PHP/웹호스팅 환경에 강함           | 현대적인 프록시/정적 서빙 구조에 강함        |

## 구조적 차이

### Apache

Apache는 전통적으로 요청을 처리할 때 프로세스 또는 스레드를 사용하는 방식입니다.

Apache의 처리 방식은 MPM, 즉 Multi-Processing Module 설정에 따라 달라집니다.

- `prefork`
  - 요청마다 별도 프로세스를 사용하는 방식입니다.
  - 오래된 `mod_php` 환경에서 많이 사용되었습니다.
- `worker`
  - 프로세스 안에서 여러 스레드를 사용합니다.
- `event`
  - keep-alive 연결 처리 효율을 개선한 방식입니다.

전통적인 PHP 서비스에서는 다음과 같은 구조가 흔했습니다.

```text
Client
  -> Apache
  -> mod_php
  -> PHP 실행
```

- Apache의 강점은 모듈 생태계와 `.htaccess`입니다.
- 디렉터리별로 rewrite, auth, access control 같은 설정을 바꿀 수 있어 공유 호스팅이나 오래된
  CMS 환경에서 유리합니다.

### Nginx

- Nginx는 이벤트 기반 비동기 구조로 동작합니다.
- 적은 수의 워커 프로세스가 많은 연결을 처리할 수 있도록 설계되었습니다.

일반적인 구조는 다음과 같습니다.

```text
Client
  -> Nginx
  -> Application Server
```

예를 들어 Spring Boot 서비스에서는 보통 이렇게 둡니다.

```text
Client
  -> Nginx : 80/443
  -> Spring Boot : 8080
```

Nginx는 애플리케이션 코드를 직접 실행하기보다 앞단에서 다음 역할을 담당하는 경우가 많습니다.

- HTTPS 종료
- 정적 파일 서빙
- 리버스 프록시
- 로드 밸런싱
- gzip, brotli 압축
- 요청 크기 제한
- rate limit
- WebSocket 프록시
- upstream health check 또는 장애 우회

## 실무 선택 기준

### Apache를 선택하기 좋은 경우

- 오래된 PHP 서비스 또는 CMS를 운영하는 경우
- WordPress, Drupal, Joomla 같은 레거시 환경을 그대로 유지해야 하는 경우
- `.htaccess` 기반 설정에 의존하는 경우
- 공유 호스팅처럼 사용자별 디렉터리 설정이 중요한 경우
- 기존 운영팀의 Apache 설정 자산과 운영 경험이 많은 경우
- 특정 Apache 모듈에 강하게 의존하는 경우

### Nginx를 선택하기 좋은 경우

- Spring Boot, Node.js, Django, FastAPI 같은 애플리케이션 서버 앞단에 둘 경우
- React, Vue, Next.js 정적 빌드 결과물을 서빙해야 하는 경우
- 많은 동시 접속을 효율적으로 처리해야 하는 경우
- 리버스 프록시와 로드 밸런싱이 중요한 경우
- Docker, Kubernetes 환경에서 단순하고 일관된 프록시 구성이 필요한 경우
- TLS, 압축, 캐시 헤더, 업로드 제한 등을 앞단에서 제어하고 싶은 경우

## Spring Boot 기준 실무 구성

Spring Boot 애플리케이션을 외부에 직접 노출할 수도 있습니다.

```text
Client
  -> Spring Boot : 8080
```

하지만 운영 환경에서는 보통 Nginx를 앞에 둡니다.

```text
Client
  -> Nginx : 80/443
  -> Spring Boot : 8080
```

Nginx 설정 예시는 다음과 같습니다.

```nginx
server {
    listen 80;
    server_name example.com;

    location / {
        proxy_pass http://127.0.0.1:8080;

        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```

Spring Boot에서는 프록시 헤더를 인식하도록 설정하는 것이 중요합니다.

```properties
server.forward-headers-strategy=framework
```

이 설정이 없으면 다음 문제가 생길 수 있습니다.

- 실제 요청은 HTTPS인데 애플리케이션이 HTTP로 인식함
- redirect URL이 `http://`로 생성됨
- 클라이언트 IP를 제대로 알 수 없음
- 인증, 세션, OAuth callback URL 처리에서 문제가 생김

## 성능 관점

단순히 "Nginx가 Apache보다 무조건 빠르다"라고 정리하면 부정확합니다.

정확히는 다음에 가깝습니다.

- 정적 파일 처리, 대량 동시 연결, 리버스 프록시는 Nginx가 대체로 유리합니다.
- `.htaccess`, Apache 모듈, 전통적인 PHP 호스팅은 Apache가 운영상 유리합니다.
- 현대 Apache도 `event MPM`과 PHP-FPM을 사용하면 과거보다 훨씬 효율적입니다.
- 실제 서비스 병목은 웹 서버보다 DB, 애플리케이션 로직, 외부 API, 커넥션 풀에서 더 자주 발생합니다.

운영 장애는 웹 서버 자체 성능보다 아래 설정 문제에서 더 자주 발생합니다.

- Nginx 또는 Apache timeout이 애플리케이션 timeout과 맞지 않음
- 로드밸런서 idle timeout과 웹 서버 keep-alive timeout 불일치
- `X-Forwarded-*` 헤더 처리 누락
- 업로드 크기 제한 불일치
- WebSocket 프록시 설정 누락
- gzip, cache-control 설정 실수
- WAS thread pool 또는 DB connection pool 고갈
- health check 경로가 실제 애플리케이션 상태를 제대로 반영하지 않음

## 운영 명령 차이

### Nginx

```shell
# 설정 문법 검사
nginx -t

# 설정 반영
systemctl reload nginx

# 재시작
systemctl restart nginx
```

### Apache

```shell
# 설정 문법 검사
apachectl configtest

# 설정 반영
systemctl reload apache2

# 재시작
systemctl restart apache2
```

CentOS, Amazon Linux 계열에서는 서비스명이 `httpd`인 경우가 많습니다.

```shell
systemctl reload httpd
systemctl restart httpd
```

## `.htaccess` 차이

Apache는 `.htaccess`를 지원합니다.

```apache
RewriteEngine On
RewriteRule ^api/(.*)$ http://localhost:8080/$1 [P,L]
```

이 방식은 디렉터리별 설정을 애플리케이션 코드와 가까운 위치에서 관리할 수 있다는 장점이 있습니다.

하지만 운영 관점에서는 단점도 있습니다.

- 요청마다 디렉터리별 `.htaccess` 파일을 확인해야 할 수 있음
- 설정이 여러 위치에 흩어져 추적하기 어려움
- 중앙 운영 정책과 충돌할 수 있음

Nginx는 `.htaccess`를 지원하지 않습니다. 설정은 보통 `/etc/nginx/nginx.conf` 또는 `/etc/nginx/conf.d/*.conf` 같은 중앙 설정 파일에서 관리합니다.

운영 환경에서는 중앙 설정 방식이 더 예측 가능하고 배포 자동화에도 유리합니다.

## Kubernetes / Docker 환경

컨테이너 환경에서는 Nginx가 자주 사용됩니다.

```text
Internet
  -> Load Balancer
  -> Nginx Ingress Controller
  -> Service
  -> Pod
```

다만 Kubernetes에서는 Nginx만 정답은 아닙니다. 실무에서는 요구사항에 따라 다음 선택지도 많이 사용합니다.

- AWS ALB Ingress Controller
- Envoy
- Traefik
- HAProxy
- Istio Ingress Gateway

클라우드 환경에서는 이미 앞단에 ALB, CloudFront, API Gateway 같은 관리형 서비스가 들어가는 경우도 많기 때문에 Nginx를 반드시 직접 운영해야 하는 것은 아닙니다.

## 정리

- 레거시 PHP, CMS, `.htaccess` 중심이면 Apache가 자연스럽습니다.
- 현대적인 API 서버, SPA, 리버스 프록시, 고동시성 처리에는 Nginx가 더 많이 사용됩니다.
- Spring Boot 운영 환경에서는 보통 Nginx를 앞단에 두고 Spring Boot는 내부 포트로만 노출합니다.
- 성능 차이보다 중요한 것은 timeout, header, SSL, cache, upload limit, health check 설정입니다.
- 운영 환경에서는 웹 서버 단독 성능보다 전체 아키텍처의 병목을 함께 봐야 합니다.
