# 🚀 HTTPD 는 무엇이고, EC2 에 설치하는 경우 어떤 일이 발생하는가

## 🤔 HTTPD 란?

- HTTP Daemon 의 약자 -> 보통 Apache HTTP Server 를 의미합니다.
- Linux 에서 yum install httpd 하면 설치되는게 바로 이 Apache 웹서버입니다.

### 하는 일

- HTTP 요청을 받아서 정적 파일(html, css, js, 이미지 등)을 응답
- 리버스 프록시(reverse proxy) 역할
  - 들어온 요청을 다른 포트(예: 8080)에서 실행 중인 앱(Springboot, Node.js) 등에 전달
- SSL/TLS(HTTPS) 처리 -> 인증서 적용 및 HTTPS 핸들링

## EC2에 HTTPD를 설치하면 어떤 일이 생기나?

- [1.] 80번 포트를 LISTEN
  - 보안 그룹에서 80 열려 있으면, 브라우저에서 http://<EC2-IP> 접속 시 httpd가 응답합니다.
  - Spring Boot가 8080에서만 돌아가고 있어도, httpd가 대신 요청을 받아 응답하거나 프록시할
    수 있습니다.
- [2.] 정적 웹서버로 동작 가능
  - 예: /var/www/html/index.html 같은 정적 페이지를 브라우저에 서빙 가능.
  - React/Vue 같은 프론트엔드 빌드 파일을 올려도 됩니다.
- [3.] 리버스 프록시로 Spring Boot와 연결 가능
  - httpd 설정에 다음과 같이 넣으면:

```shell
ProxyPass "/" "http://localhost:8080/"
ProxyPassReverse "/" "http://localhost:8080/"
```

- [4.] SSL(TLS) 처리
  - mod_ssl 모듈을 이용해 HTTPS 인증서를 설치하면, httpd가 443포트에서 HTTPS 요청을 받고
    뒤의 Spring Boot 앱에는 HTTP로 전달합니다.
  - Spring Boot 자체 SSL 설정 대신 httpd에서 관리하는 게 일반적입니다.

## 3. 그럼 설치하면 뭐가 달라지는가?

- 지금 상태:
  - 8080만 열려 있음 → http://<EC2-IP>:8080 으로만 접근 가능
  - http://<EC2-IP> (80포트) 접근하면 아무도 응답 안 함 → 연결 실패
- httpd 설치 후 (80 리스닝 시작):
  - http://<EC2-IP> 접속 시 httpd가 응답
  - 설정에 따라:
    - 정적 페이지 보여주기 가능
    - Spring Boot 8080으로 프록시 가능
    - SSL 인증서 적용해서 HTTPS로 서비스 가능

---

✅ 정리

- HTTPD = Apache 웹서버
- EC2에 설치하면 → 80/443 포트를 열고 클라이언트 요청을 처리할 수 있게 됨
- 주로 쓰이는 용도:
  - [1.] 정적 리소스 서빙
  - [2.] 리버스 프록시(80/443 → 8080)
  - [3.] SSL 인증서 관리
