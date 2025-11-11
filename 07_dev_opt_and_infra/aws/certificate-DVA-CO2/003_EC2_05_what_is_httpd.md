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

---

### 실습

- 환경: Ubuntu 24 LTS

#### 1. 설치

Ubuntu에서는 httpd 대신 패키지명이 apache2입니다. (yum을 쓰는 CentOS/RHEL 계열은 httpd)

```shell
sudo apt update
sudo apt install apache2 -y
```

#### 2. 설치 후 생성되는 주요 파일/디렉토리

설치가 완료되면 기본적으로 아래 구조가 생깁니다:

- 실행/서비스 관련
  - /usr/sbin/apache2 → Apache 실행 바이너리
  - /lib/systemd/system/apache2.service → systemd 서비스 유닛 파일 (부팅 시 자동 실행 가능)
- 설정 파일
  - /etc/apache2/apache2.conf → 메인 설정 파일
  - /etc/apache2/ports.conf → 어떤 포트를 열지 정의 (기본: 80, 443)
  - /etc/apache2/sites-available/000-default.conf → 기본 가상호스트 설정 (DocumentRoot 경로 등)
  - /etc/apache2/sites-enabled/ → 실제 활성화된 사이트 설정 심볼릭 링크
- 웹 루트 디렉토리
  - /var/www/html/ → 기본 문서 디렉토리 (여기 index.html 있으면 브라우저에서 보여짐)
- 로그 파일
  - /var/log/apache2/access.log → 요청 로그
  - /var/log/apache2/error.log → 에러 로그

#### 3. 서비스 동작

```shell
sudo systemctl status apache2
```

출력에 active (running)이 보이면 성공적으로 동작 중입니다. 기본적으로:

- 80번 포트 (HTTP) 에서 LISTEN
- 443번 포트 (HTTPS) 는 SSL 모듈 활성화해야 동작
- /etc/apache2/ports.conf 파일 확인

```shell
sudo lsof -i -P -n | grep LISTEN | grep apache2
```

#### 4. 외부 접속 동작 방식

- 클라이언트 브라우저 → EC2 퍼블릭 IP (80포트) 요청
- 보안 그룹에서 80번 포트가 열려 있으면 → EC2로 트래픽 전달
- EC2 내 apache2 프로세스가 80포트를 LISTEN 중이므로 요청을 수신
- /var/www/html/index.html 같은 파일을 찾아 클라이언트로 응답
  - • 없으면 기본적으로 Apache2 Ubuntu Default Page 라는 안내 페이지가 뜸
