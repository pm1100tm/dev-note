# Amazon CloudFront 보안 기능

| 보안 기능             | 설명                                                           |
| --------------------- | -------------------------------------------------------------- |
| DDoS 보호             | 전 세계 Edge Network를 통해 대규모 트래픽을 흡수               |
| AWS Shield 연동       | 기본적으로 AWS Shield Standard 보호 적용                       |
| AWS WAF 연동          | SQL Injection, XSS, IP 차단 등 웹 공격 방어                    |
| HTTPS 지원            | ACM 인증서를 사용해 TLS 적용 가능                              |
| Origin Access Control | S3 Origin을 직접 공개하지 않고 CloudFront를 통해서만 접근 가능 |
| Geo Restriction       | 특정 국가에서의 접근 허용 또는 차단 가능                       |
| Signed URL            | 특정 파일에 대해 제한된 접근 권한 제공                         |
| Signed Cookies        | 여러 파일에 대해 제한된 접근 권한 제공                         |

<br>

## 1. AWS Shield

CloudFront는 AWS Shield Standard와 통합되어 기본 DDoS 보호를 제공합니다.
AWS 공식 CloudFront 페이지에서도 Shield Standard를 통한 DDoS 방어를 설명합니다.

```
DDoS 공격
  ↓
CloudFront + Shield
  ↓
Origin 보호
```

AWS Shield에는 두 가지 종류가 있습니다.

- AWS Shield Standard: 기본 제공 DDoS 보호
- AWS Shield Advanced: 유료 고급 DDoS 보호, 비용 보호 및 상세 대응 지원

CloudFront를 사용하면 Origin Server가 직접 인터넷 트래픽을 받는 구조보다 DDoS에 더 강한 구조를
만들 수 있습니다.

<br>

## 2. AWS WAF

AWS WAF는 Web Application Firewall입니다.
CloudFront 앞단에 WAF를 붙이면 악성 요청을 Origin까지 보내기 전에 차단할 수 있습니다.

예를 들어 다음과 같은 규칙을 만들 수 있습니다.

| WAF 규칙           | 설명                                        |
| ------------------ | ------------------------------------------- |
| IP 차단            | 특정 IP 주소에서 오는 요청 차단             |
| Rate-based Rule    | 일정 시간 동안 과도한 요청을 보내는 IP 차단 |
| SQL Injection 차단 | SQL Injection 패턴 탐지                     |
| XSS 차단           | Cross-Site Scripting 공격 탐지              |
| 특정 국가 차단     | 특정 국가에서 오는 요청 제한                |

```
사용자
  ↓
CloudFront
  ↓
AWS WAF 검사
  ↓
정상 요청만 Origin 전달
```

AWS 문서에서도 WAF를 사용해 CloudFront Distribution과 Origin 서버를 보호할 수 있다고 설명합니다.

정확히는 WAF가 CloudFront 배포에 연결되어 CloudFront 레벨에서 요청을 검사한다고 보면 됩니다.

### WAF로 막을 수 있는 것들입니다.

| 방어 대상     | 설명                    |
| ------------- | ----------------------- |
| SQL Injection | SQL 삽입 공격 차단      |
| XSS           | 스크립트 삽입 공격 차단 |
| IP 차단       | 특정 IP 또는 국가 차단  |
| Rate Limit    | 과도한 요청 제한        |
| Bot 차단      | 비정상 봇 트래픽 제한   |

<br>

## OAC / OAI

- S3를 Origin으로 사용할 때 중요한 개념입니다.
- 예전에는 **OAI(Origin Access Identity)**를 많이 사용했고, 현재는 OAC(Origin Access Control)
  사용이 권장됩니다.

목적은 간단합니다.

```
사용자 → S3 직접 접근 차단
사용자 → CloudFront → S3 접근 허용
```

즉, S3 버킷을 Public으로 열지 않고 CloudFront를 통해서만 접근하게 만드는 구조입니다.

OAC는 S3 Origin을 보호하는 기능입니다.
사용자는 S3 URL로 직접 접근하지 못하고, CloudFront Distribution을 통해서만 객체를 조회합니다.

```
User
  |
  v
CloudFront
  |
  v
OAC
  |
  v
Private S3 Bucket
```

## Geo Restriction

Geo Restriction은 사용자의 국가를 기준으로 CloudFront Distribution 접근을 허용하거나 차단하는 기능입니다.

| 방식 | 설명 |
| --- | --- |
| Allowlist | 지정한 국가의 사용자만 접근 허용 |
| Blocklist | 지정한 국가의 사용자 접근 차단 |

사용자 국가는 3rd party Geo-IP database를 기준으로 판단합니다.

대표 사용 사례는 저작권, 라이선스, 지역별 서비스 제한입니다.

```
특정 국가에서만 영상 콘텐츠 제공
특정 국가에서는 다운로드 차단
```

## Signed URL / Signed Cookies

유료 콘텐츠, 프리미엄 사용자 콘텐츠, 비공개 파일처럼 접근 권한이 필요한 콘텐츠는 Signed URL 또는
Signed Cookies로 보호할 수 있습니다.

Signed URL / Signed Cookies에는 정책을 붙일 수 있습니다.

- URL 만료 시간
- 접근 가능한 IP 범위
- 접근 가능한 경로
- 신뢰할 수 있는 signer

### Signed URL

Signed URL은 개별 파일에 대한 접근 권한을 부여합니다.

```
https://d123.cloudfront.net/movie.mp4?Signature=...
```

| 적합한 경우 | 이유 |
| --- | --- |
| 특정 파일 하나에 접근 허용 | 파일마다 URL을 발급 |
| 짧은 시간 동안 공유 콘텐츠 제공 | 만료 시간을 몇 분 단위로 짧게 설정 가능 |

### Signed Cookies

Signed Cookies는 여러 파일에 대한 접근 권한을 한 번에 부여합니다.

| 적합한 경우 | 이유 |
| --- | --- |
| 회원 전용 강의 여러 개 접근 | 파일마다 Signed URL을 만들 필요 없음 |
| 로그인 후 프리미엄 콘텐츠 영역 접근 | Cookie 기반으로 여러 경로 접근 제어 |

## Signed URL 생성 방식

CloudFront Signed URL은 signer가 필요합니다.

| Signer | 설명 |
| --- | --- |
| Trusted Key Group | 권장 방식. API로 키 생성/회전 가능, IAM으로 관리 가능 |
| CloudFront Key Pair가 있는 AWS Account | root account로 키를 관리해야 하므로 권장되지 않음 |

권장 흐름은 다음과 같습니다.

```
1. 애플리케이션이 사용자를 인증/인가
2. 애플리케이션이 private key로 Signed URL 생성
3. CloudFront에 public key를 등록한 Trusted Key Group 연결
4. 사용자가 Signed URL로 CloudFront에 요청
5. CloudFront가 signature를 검증한 뒤 Origin 콘텐츠 제공
```

## CloudFront Signed URL vs S3 Pre-Signed URL

둘 다 제한된 접근을 제공하지만 목적과 동작 범위가 다릅니다.

| 구분 | CloudFront Signed URL | S3 Pre-Signed URL |
| --- | --- | --- |
| 접근 대상 | CloudFront 경로. Origin 종류와 무관하게 사용 가능 | S3 객체 |
| 서명 주체 | Trusted Key Group 또는 CloudFront Key Pair | IAM Principal의 자격 증명 |
| 제어 조건 | IP, path, date, expiration 등 | 제한된 lifetime 중심 |
| 캐싱 | CloudFront 캐싱 활용 가능 | CloudFront 캐싱 목적이 아님 |
| 사용 예 | 글로벌 유료 콘텐츠 배포 | 특정 S3 객체 임시 업로드/다운로드 |

시험에서 "전 세계 사용자에게 유료 콘텐츠를 배포하고 캐싱도 활용"한다면 CloudFront Signed URL/Cookies가
더 적절합니다.

| 상황                                          | 정답 후보                              |
| --------------------------------------------- | -------------------------------------- |
| 전 세계 사용자에게 정적 콘텐츠를 빠르게 제공  | CloudFront                             |
| S3 정적 파일을 빠르게 배포                    | S3 + CloudFront                        |
| Origin 서버 부하를 줄이고 싶음                | CloudFront Caching                     |
| DDoS 방어가 필요                              | CloudFront + AWS Shield                |
| SQL Injection, XSS 방어 필요                  | CloudFront + AWS WAF                   |
| S3를 Public으로 열지 않고 CloudFront로만 접근 | CloudFront OAC (Origin Access Control) |
| 캐시된 파일을 즉시 갱신                       | CloudFront Invalidation                |
| 특정 국가 접근 차단                           | Geo Restriction                        |
| 유료 파일 하나에 제한된 접근 제공             | CloudFront Signed URL                  |
| 여러 프리미엄 파일에 제한된 접근 제공         | CloudFront Signed Cookies              |
| 사용자 위치에 따라 Origin 다르게 처리         | CloudFront Functions / Lambda@Edge     |
| HTTPS 커스텀 도메인 연결                      | CloudFront + ACM 인증서                |
| 루트 도메인을 CloudFront에 연결               | Route 53 Alias                         |
