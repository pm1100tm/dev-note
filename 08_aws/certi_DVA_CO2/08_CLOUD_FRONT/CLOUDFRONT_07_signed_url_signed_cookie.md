# CloudFront Signed URL vs Signed Cookies

CloudFront Signed URL과 Signed Cookies는 제한된 사용자에게 private content를 제공할 때 사용합니다.

대표 시나리오는 다음과 같습니다.

- 유료 영상, 음악, 강의 콘텐츠 제공
- 프리미엄 사용자에게만 파일 다운로드 허용
- 특정 시간 동안만 private file 접근 허용
- 특정 IP 대역에서만 콘텐츠 접근 허용

## Signed URL

Signed URL은 개별 파일에 대한 접근 권한을 URL에 담아 제공하는 방식입니다.

```text
https://d123.cloudfront.net/videos/movie.mp4?Expires=...&Signature=...&Key-Pair-Id=...
```

| 특징        | 설명                                                 |
| ----------- | ---------------------------------------------------- |
| 접근 범위   | 개별 파일                                            |
| 발급 단위   | 파일마다 Signed URL 생성                             |
| 적합한 경우 | 특정 파일 하나를 짧은 시간 동안 제공                 |
| 예시        | 유료 다운로드 링크, 짧은 시간만 재생 가능한 영상 URL |

## Signed Cookies

Signed Cookies는 여러 파일에 대한 접근 권한을 Cookie로 제공하는 방식입니다.

| 특징        | 설명                                         |
| ----------- | -------------------------------------------- |
| 접근 범위   | 여러 파일 또는 경로                          |
| 발급 단위   | Cookie 한 번 발급                            |
| 적합한 경우 | 여러 protected object에 접근해야 하는 서비스 |
| 예시        | 로그인한 프리미엄 회원이 여러 강의 영상 접근 |

```text
1. 사용자가 로그인
2. 애플리케이션이 권한 확인
3. 애플리케이션이 Signed Cookies 발급
4. 사용자가 CloudFront protected content 접근
```

## Signed URL vs Signed Cookies

| 구분          | Signed URL              | Signed Cookies                   |
| ------------- | ----------------------- | -------------------------------- |
| 접근 대상     | 개별 파일               | 여러 파일 또는 경로              |
| 사용자 경험   | URL마다 서명 필요       | Cookie 기반으로 여러 요청 처리   |
| 적합한 콘텐츠 | 단일 파일 다운로드      | 회원 전용 콘텐츠 영역            |
| 예시          | `movie.mp4` 하나 제공   | `/premium/*` 전체 접근           |
| 시험 키워드   | one signed URL per file | one signed cookie for many files |

## 정책에 포함할 수 있는 조건

Signed URL과 Signed Cookies에는 접근 정책을 붙일 수 있습니다.

| 조건           | 설명                         |
| -------------- | ---------------------------- |
| Expiration     | URL 또는 Cookie 만료 시간    |
| IP Range       | 특정 IP 범위에서만 접근 허용 |
| Path           | 특정 경로에 대한 접근 허용   |
| Trusted Signer | 서명할 수 있는 주체          |

공유 콘텐츠는 보통 짧은 만료 시간을 사용합니다.

```text
영화, 음악 공유 링크 -> 몇 분 단위
```

사용자 개인 콘텐츠는 더 긴 만료 시간을 사용할 수 있습니다.

```text
사용자 개인 파일 -> 더 긴 기간
```

## Trusted Key Group

CloudFront Signed URL/Cookies를 사용하려면 signer가 필요합니다.

권장 방식은 Trusted Key Group입니다.

```text
1. public/private key pair 생성
2. public key를 CloudFront에 업로드
3. public key를 Trusted Key Group에 연결
4. Distribution의 Cache Behavior에 Trusted Key Group 연결
5. 애플리케이션은 private key로 URL 또는 Cookie 서명
6. CloudFront는 public key로 signature 검증
```

| Signer                                 | 설명                                            | 권장 여부 |
| -------------------------------------- | ----------------------------------------------- | --------- |
| Trusted Key Group                      | API로 키 관리/회전 가능, IAM으로 권한 제어 가능 | 권장      |
| CloudFront Key Pair가 있는 AWS Account | root account로 키를 관리해야 함                 | 비권장    |

시험에서는 "root account로 key pair를 관리해야 하는 방식"이 나오면 비권장 방식으로 보면 됩니다.

## Signed URL 처리 흐름

```text
Client
  |
  v
Application
  |
  | 인증/인가 후 Signed URL 생성
  v
Client
  |
  | Signed URL로 요청
  v
CloudFront
  |
  | signature 검증
  v
Origin
```

Origin이 S3라면 OAC를 함께 사용해 S3 bucket을 private으로 유지할 수 있습니다.

## CloudFront Signed URL vs S3 Pre-Signed URL

| 구분        | CloudFront Signed URL                      | S3 Pre-Signed URL                          |
| ----------- | ------------------------------------------ | ------------------------------------------ |
| 접근 대상   | CloudFront path                            | S3 object                                  |
| Origin 제한 | Origin 종류와 무관하게 사용 가능           | S3 전용                                    |
| 서명 주체   | Trusted Key Group 또는 CloudFront Key Pair | IAM principal 자격 증명                    |
| 권한 의미   | CloudFront가 signature 검증 후 콘텐츠 제공 | 서명한 IAM principal 권한으로 S3 요청 수행 |
| 조건        | IP, path, date, expiration 등              | 제한된 lifetime 중심                       |
| 캐싱        | CloudFront 캐싱 활용 가능                  | CloudFront 캐싱과 직접 관련 없음           |
| 글로벌 배포 | 적합                                       | S3 endpoint 접근 중심                      |

## 시험 함정

| 문제 표현                                       | 정답 방향                                 |
| ----------------------------------------------- | ----------------------------------------- |
| "Premium users around the world"                | CloudFront Signed URL 또는 Signed Cookies |
| "Access to individual files"                    | Signed URL                                |
| "Access to multiple files"                      | Signed Cookies                            |
| "Use caching while protecting private content"  | CloudFront Signed URL/Cookies             |
| "Temporary access to a specific S3 object"      | S3 Pre-Signed URL                         |
| "Recommended signer for CloudFront signed URLs" | Trusted Key Group                         |
| "Root account manages key pair"                 | CloudFront Key Pair 방식, 비권장          |

## 한 줄 요약

Signed URL은 파일 하나에 대한 제한 접근이고, Signed Cookies는 여러 파일에 대한 제한 접근입니다.
CloudFront에서 private content를 글로벌 캐싱과 함께 보호하려면 S3 Pre-Signed URL보다 CloudFront Signed
URL/Cookies가 더 적합합니다.
