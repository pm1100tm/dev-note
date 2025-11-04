# 🚀 DNS 관련 용어 정리

DNS - Domain Name System

사람이 읽을 수 있는 주소(example.com)를 실제 IP 주소(192.0.2.1)로 변환해주는 시스템입니다.
이 변환을 담당하는 것이 DNS 이며, 다양한 레코드(Record) 타입으로 구성됩니다.

## 🔹 1. A 레코드 (Address Record)

- 역할: 도메인 이름을 IPv4 주소(예: 123.45.67.89) 로 연결합니다.
- 설명
  - 브라우저가 example.com 에 접속할 때, 이 A 레코드가 가리키는 IP 주소로 요청을 보냄.
  - 즉, 웹서버나 로드벨런서의 IPv4 주소를 등록할 때 사용

```shell
example.com.   IN   A   123.45.67.89
```

- 주요 사용 예:
  - Cloud Run / EC2 / Nginx 서버와 연결
  - 기본 웹사이트 주소 (@) 또는 특정 서브도메인 (api.example.com)

## 🔹 2. AAAA 레코드 (Quad A Record)

- 역할: 도메인 이름을 IPv6 주소(예: 2001:db8::1) 로 연결합니다.
- 설명:
  - IPv6 기반 네트워크 환경을 지원하기 위한 레코드입니다.
  - IPv4(A 레코드)와 함께 등록해 두면, 클라이언트 환경에 맞게 자동으로 선택됩니다.

```shell
example.com.   IN   AAAA   2001:db8::1
```

- 주요 사용 예:
  - IPv6 지원 서버 (특히 CDN, CloudFront, Cloud DNS 등)

## 🔹 3. CNAME 레코드 (Canonical Name Record)

- 역할: 하나의 도메인 이름을 다른 도메인 이름에 별칭으로 연결합니다.
- 설명:
  - www.example.com 요청을 example.com으로 전달하도록 설정합니다.
  - 즉, 실제 IP를 지정하는 대신 다른 도메인 이름을 참조합니다.

```shell
www.example.com. IN CNAME example.com.
```

- 주의점:
  - 루트 도메인(example.com)에는 CNAME을 설정할 수 없습니다.
  - 항상 하위 도메인(www, blog, api)에만 사용합니다.
- 주요 사용 예:
  - Cloud Run, GitHub Pages, Netlify, Vercel, Firebase Hosting 연결 시
  - 서브도메인을 특정 호스팅 서비스로 연결할 때

## 🔹 4. MX 레코드 (Mail eXchange Record)

- 역할: 메일 서버 주소를 지정합니다. 이메일 전송 시, 어떤 서버가 메일을 수신할지 알려주는 역할입니다.
- 설명:
  - 숫자는 우선순위(priority) 를 의미하며, 숫자가 낮을수록 먼저 시도됩니다.
  - 메일 서버가 장애 시, 백업 서버로 순차적으로 전달됩니다.

```shell
example.com.   IN   MX   10   mail.example.com.
example.com.   IN   MX   20   backup-mail.example.com.
```

- 주요 사용 예:
  - Google Workspace, Naver Works, Microsoft 365, Zoho Mail 등과 연동 시
  - 기업 메일 서비스 세팅

## 🔹 5. TXT 레코드 (Text Record)

- 역할
  - 도메인과 관련된 문자열 정보를 저장합니다.
  - 인증용, 보안용, 또는 기타 설명용 데이터로 주로 사용됩니다.
- 설명:
  - 이메일 스푸핑 방지를 위한 SPF 레코드,
  - 또는 Google Search Console / AWS / GitHub / SSL 인증 등의 도메인 소유 검증에 자주 사용됩니다.

```shell
example.com.   IN   TXT   "v=spf1 include:_spf.google.com ~all"
```

- 주요 사용 예:
  - SPF, DKIM, DMARC 메일 인증
  - Google, AWS, GitHub 도메인 검증
  - API Key, 서비스 연결 인증

## 🔹 6. NS 레코드 (Name Server Record)

- 역할: 도메인 이름을 어떤 네임서버가 관리하는지 지정합니다.
- 설명:
  - 이 NS 레코드가 가리키는 네임서버에서 해당 도메인의 모든 DNS 레코드를 조회할 수 있습니다.
  - 도메인을 처음 등록하면 기본 NS가 자동 설정되지만, Cloud DNS로 이전 시 변경해야 합니다.

```shell
example.com.   IN   NS   ns1.google.com.
example.com.   IN   NS   ns2.google.com.
```

- 주요 사용 예:
  - Google Cloud DNS, Route53, Cloudflare 등으로 DNS 이전
  - 도메인 레지스트리에서 네임서버 변경 시 필요

---

## 📘 정리표

| 레코드 타입 | 역할                      | 주요 사용 예                      |
| ----------- | ------------------------- | --------------------------------- |
| A           | 도메인 -> IPv4 주소 연결  | 웹서버, 로드벨런서                |
| AAAA        | 도메인 -> IPv6 주소 연결  | IPv6 지원 서비스                  |
| CNAME       | 다른 도메인으로 별칭 연결 | www, blog, api 등 서브도메인      |
| MX          | 메일 서버 지정            | Gmail, Naver Works 등 메일 서비스 |
| TXT         | 텍스트 정보 저장          | SPF/DKIM/도메인 인증              |
| NS          | 네임서버 지정             | Cloud DNS / Cloudflare 등         |

---

## 💡 요약

DNS는 웹사이트 접속뿐 아니라 이메일, 보안 인증, 서비스 연동 등 인터넷의 모든 동작의 기반입니다.
Cloud DNS나 Route53 같은 관리형 DNS 서비스를 이용하면 이 레코드들을 손쉽게 설정하고,
안정적으로 운영할 수 있습니다.
