# 🚀 S3 Overview

- 객체(Object) 기반의 무한 확장 스토리지 서비스
- 파일을 서버에 저장하는 개념이 아니라, 객체(Object)를 Bucket 에 저장하는 구조

| 왜 S3 인가?

- 무한 확장: 용량 개념 없음
  - 디스크 용량 ❌
  - 파일 수 제한 ❌
  - 트래픽 제한 ❌ (실질적으로)
- 높은 내구성: 99.999999999% (11 9’s)
- 글로벌 사용: 수많은 서비스의 백엔드
- 서비스 통합: 거의 모든 AWS 서비스와 연동

- 👉 AWS 서비스 대부분이 내부적으로 S3를 사용
- ➡️ 확장 걱정 자체를 없애는 스토리지

## Amazon S3 주요 Use Cases (시험 단골)

| Use Case            | 설명                     |
| ------------------- | ------------------------ |
| Backup & Storage    | 기본 저장소              |
| Disaster Recovery   | Cross-Region Replication |
| Archive             | Glacier 계열             |
| Hybrid Cloud        | 온프레미스 + S3          |
| Application Hosting | 정적 자원                |
| Media Hosting       | 이미지/영상              |
| Data Lake           | 분석 데이터 저장         |
| Big Data Analytics  | Athena, EMR              |
| Software Delivery   | 설치 파일                |
| Static Website      | HTML/CSS/JS              |

👉 “파일 저장”이 보이면 S3부터 의심

## Amazon S3의 기본 구조

```shell
Bucket
 └── Object
      ├── Key (파일 이름)
      ├── Value (데이터)
      ├── Metadata
      └── Version ID (옵션)
```

**_핵심 개념_**

- Bucket = 최상위 컨테이너
- Object = 실제 파일
- 폴더는 개념적 구조 (Key prefix)

## Bucket의 핵심 속성 (시험 필수)

- ① Bucket 이름은 글로벌 유니크
  - 모든 리전 + 모든 계정 통틀어 유일
- ② Bucket은 Region 단위로 생성
  - S3는 글로벌 서비스 ❌
  - Bucket은 리전 소속 ⭕

```shell
ap-northeast-2
us-east-1
eu-west-1
```

👉 Bucket은 한 리전에만 존재

## Bucket Naming Convention (시험에서 그대로 나옴)

- 소문자만
- 언더스코어 불가
- 길이 3~63자
- IP 주소 형태 ❌
- 시작은 소문자 | 숫자
- 접두어 금지: xn--
- 점미어 금지: -s3alias
