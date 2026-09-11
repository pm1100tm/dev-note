# CloudFront Signed URL vs S3 Pre-Signed URL

둘 다 "잠깐만 쓸 수 있는 비밀 입장권"입니다.

하지만 어느 문으로 들어가느냐가 다릅니다.

| 구분        | CloudFront Signed URL                                        | S3 Pre-Signed URL                                 |
| ----------- | ------------------------------------------------------------ | ------------------------------------------------- |
| 들어가는 문 | CloudFront 문                                                | S3 문                                             |
| 목적        | 전 세계 사용자에게 빠르게, 캐시까지 써서 private 콘텐츠 제공 | S3 파일 하나를 잠깐 업로드/다운로드하게 허용      |
| 주 사용처   | 유료 영상, 강의, 이미지, 다운로드 콘텐츠 배포                | 사용자가 S3에 파일 업로드, 관리자용 임시 다운로드 |
| 속도        | CloudFront Edge Cache 사용 가능                              | S3 리전으로 직접 감                               |
| Origin      | S3, ALB, EC2 등 가능                                         | S3만 가능                                         |
| 서명 기준   | CloudFront key group/private key                             | IAM 권한                                          |

## 12살에게 설명하면

S3는 "창고"입니다.

CloudFront는 전 세계 여러 동네에 있는 "작은 배달 창구"입니다.

## S3 Pre-Signed URL

S3 Pre-Signed URL은 이런 느낌입니다.

> 창고에 있는 이 상자 하나를 10분 동안 꺼내 갈 수 있는 임시 열쇠

예를 들어 `report.pdf` 파일을 S3에 넣어두고, 친구에게 10분 동안만 다운로드하게 하고 싶습니다.

```text
사용자
  |
  | S3 Pre-Signed URL
  v
S3 Bucket
```

사용자는 CloudFront를 거치지 않고 S3에 직접 갑니다.

좋은 사용 예시는 이런 경우입니다.

- 사용자가 프로필 이미지를 S3에 직접 업로드
- 관리자가 특정 백업 파일을 5분 동안 다운로드
- 고객에게 영수증 PDF 하나를 임시로 제공
- 서버를 거치지 않고 브라우저에서 S3로 바로 파일 업로드

특히 업로드에는 S3 Pre-Signed URL이 자주 쓰입니다.

```text
1. 사용자가 프로필 사진 업로드 요청
2. 서버가 S3 Pre-Signed PUT URL 발급
3. 사용자가 그 URL로 S3에 직접 업로드
4. 서버는 큰 파일을 직접 받지 않아도 됨
```

## CloudFront Signed URL

CloudFront Signed URL은 이런 느낌입니다.

> 전 세계 배달 창구에서 이 영화 파일을 5분 동안 볼 수 있는 입장권

예를 들어 유료 강의 영상 `lesson1.mp4`를 프리미엄 회원에게만 보여주고 싶습니다.

```text
사용자
  |
  | CloudFront Signed URL
  v
CloudFront Edge Location
  |
  v
S3 / ALB / EC2 Origin
```

사용자는 S3로 직접 가지 않습니다.

CloudFront로 갑니다.

CloudFront가 "이 URL은 진짜 허가받은 URL인가?"를 검사한 뒤, 맞으면 콘텐츠를 줍니다.

좋은 사용 예시는 이런 경우입니다.

- 유료 강의 영상 제공
- 프리미엄 회원 전용 이미지/파일 제공
- 전 세계 사용자에게 빠르게 private 콘텐츠 제공
- S3 버킷은 private으로 잠그고 CloudFront를 통해서만 접근 허용
- 특정 IP 대역 또는 특정 시간 동안만 콘텐츠 접근 허용

CloudFront Signed URL은 캐시를 쓸 수 있습니다.

그래서 전 세계 사용자가 같은 영상을 볼 때 S3까지 매번 가지 않고 가까운 Edge Location에서 빠르게 받을 수 있습니다.

## 가장 중요한 차이

### S3 Pre-Signed URL은 S3 직접 임시 열쇠

```text
사용자 -> S3
```

S3 파일 하나에 대해 임시 권한을 줍니다.

서명한 IAM 사용자의 권한으로 S3 요청을 수행합니다.

예시:

```text
https://my-bucket.s3.amazonaws.com/profile.png?...signature...
```

### CloudFront Signed URL은 CloudFront 입장권

```text
사용자 -> CloudFront -> Origin
```

CloudFront가 URL 서명을 검사합니다.

Origin은 S3일 수도 있고, ALB나 EC2일 수도 있습니다.

예시:

```text
https://d123.cloudfront.net/videos/lesson1.mp4?Signature=...
```

## 언제 뭘 쓰나

| 상황                                              | 더 적합한 선택              |
| ------------------------------------------------- | --------------------------- |
| 사용자가 S3에 프로필 사진을 업로드해야 함         | S3 Pre-Signed URL           |
| 특정 S3 파일 하나를 5분 동안 다운로드하게 함      | S3 Pre-Signed URL           |
| 전 세계 프리미엄 회원에게 유료 영상을 빠르게 제공 | CloudFront Signed URL       |
| S3 버킷은 private으로 숨기고 CDN으로만 제공       | CloudFront Signed URL + OAC |
| 같은 사용자가 여러 강의 파일에 접근해야 함        | CloudFront Signed Cookies   |
| 캐싱으로 S3 부하를 줄이고 싶음                    | CloudFront Signed URL       |
| Origin이 S3가 아니라 ALB/EC2일 수도 있음          | CloudFront Signed URL       |

## 실무 예시 1: 프로필 사진 업로드

이 경우는 S3 Pre-Signed URL이 적합합니다.

```text
1. 사용자가 "사진 업로드" 클릭
2. 백엔드가 S3 Pre-Signed PUT URL 생성
3. 브라우저가 S3에 직접 사진 업로드
4. 업로드 완료 후 백엔드에 파일 경로 저장
```

이때 CloudFront Signed URL은 적합하지 않습니다.

목적이 "콘텐츠 배포"가 아니라 "S3에 업로드"이기 때문입니다.

## 실무 예시 2: 유료 강의 영상 보기

이 경우는 CloudFront Signed URL이 적합합니다.

```text
1. 사용자가 로그인
2. 백엔드가 결제 여부 확인
3. 백엔드가 CloudFront Signed URL 생성
4. 사용자가 CloudFront URL로 영상 시청
5. CloudFront가 서명을 검증하고 영상 제공
```

S3 버킷은 private으로 잠그고, CloudFront OAC만 접근하게 만들면 더 안전합니다.

## 기억법

S3 Pre-Signed URL:

> S3 창고에 직접 가는 임시 열쇠

CloudFront Signed URL:

> 전 세계 CDN 배달 창구에서 private 콘텐츠를 받는 입장권

## 시험 포인트

| 문제 표현                      | 정답 방향                                               |
| ------------------------------ | ------------------------------------------------------- |
| Temporary access to S3 object  | S3 Pre-Signed URL                                       |
| Premium users around the world | CloudFront Signed URL 또는 Signed Cookies               |
| Leverage caching               | CloudFront Signed URL                                   |
| Upload directly to S3          | S3 Pre-Signed URL                                       |
| One file                       | CloudFront Signed URL 또는 S3 Pre-Signed URL. 문맥 확인 |
| Many files / premium area      | CloudFront Signed Cookies                               |

## 한 줄 요약

S3 Pre-Signed URL은 S3 파일 하나에 직접 접근하는 임시 열쇠이고, CloudFront Signed URL은 CloudFront를
통해 private 콘텐츠를 빠르게 배포하는 입장권입니다.
