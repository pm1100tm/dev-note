# CloudFront vs S3 Cross Region Replication

CloudFront와 S3 Cross Region Replication은 모두 글로벌 성능이나 지역 분산과 관련되어 보일 수 있지만
목적이 다릅니다.

CloudFront는 CDN 캐싱 서비스이고, S3 CRR은 S3 객체를 다른 리전의 버킷으로 복제하는 기능입니다.

## 핵심 차이

| 구분          | CloudFront                                   | S3 Cross Region Replication           |
| ------------- | -------------------------------------------- | ------------------------------------- |
| 목적          | Edge Location에 콘텐츠 캐싱                  | S3 객체를 다른 리전 버킷으로 복제     |
| 범위          | Global Edge Network                          | 설정한 대상 리전                      |
| 업데이트 방식 | TTL 만료 또는 Invalidation 후 새 콘텐츠 반영 | 객체 변경을 near real-time으로 복제   |
| 읽기 성능     | 전 세계 사용자에게 낮은 latency 제공         | 특정 몇 개 리전에서 낮은 latency 제공 |
| 데이터 저장   | Edge cache                                   | 실제 S3 bucket에 복제본 저장          |
| 쓰기          | Origin에 쓰기                                | 복제 대상은 read-only 복제본 성격     |
| 적합한 콘텐츠 | 전 세계에 배포할 정적 콘텐츠                 | 몇 개 리전에서 최신 객체 접근 필요    |

## CloudFront

CloudFront는 전 세계 Edge Location에 콘텐츠를 캐싱합니다.

```text
User
  |
  v
Nearest CloudFront Edge Location
  |
  v
Cache Hit이면 즉시 응답
```

CloudFront는 아래 상황에 적합합니다.

- 정적 파일을 전 세계 사용자에게 빠르게 제공
- 이미지, JS, CSS, 동영상 캐싱
- Origin 부하 감소
- DDoS 보호, Shield, WAF 연동
- TTL 기반 캐싱 허용

주의할 점은 CloudFront가 Origin 변경을 즉시 아는 것은 아니라는 점입니다.
TTL이 만료되거나 Invalidation을 실행해야 새 콘텐츠를 가져옵니다.

## S3 Cross Region Replication

S3 CRR은 한 리전의 S3 버킷 객체를 다른 리전의 S3 버킷으로 복제합니다.

```text
S3 Bucket in us-east-1
  |
  | replication
  v
S3 Bucket in ap-northeast-2
```

S3 CRR은 아래 상황에 적합합니다.

- 특정 몇 개 리전에 객체 복제본이 필요
- 리전 수준 재해 복구 구성
- 규정 준수 목적으로 리전 간 복제 필요
- 객체 변경을 near real-time으로 다른 리전에 반영

복제를 원하는 리전마다 별도로 설정해야 합니다.

## 선택 기준

| 요구사항                                      | 선택                    |
| --------------------------------------------- | ----------------------- |
| 전 세계 사용자에게 정적 콘텐츠를 빠르게 제공  | CloudFront              |
| Edge Location에서 캐싱하고 Origin 부하를 줄임 | CloudFront              |
| TTL 기반 캐싱이 가능                          | CloudFront              |
| 캐시된 파일을 강제로 갱신                     | CloudFront Invalidation |
| 몇 개 리전의 S3 버킷에 객체 복제              | S3 CRR                  |
| 리전 장애 대비용 S3 복제본 필요               | S3 CRR                  |
| 객체 변경을 다른 리전에 near real-time 반영   | S3 CRR                  |

## 함께 쓰는 구조

CloudFront와 S3 CRR은 함께 사용할 수도 있습니다.

예를 들어 여러 리전의 S3 버킷을 만들고, CloudFront Origin Group으로 고가용성 구조를 구성할 수 있습니다.

```text
CloudFront
  |
  v
Origin Group
  |-- Primary S3 bucket
  |-- Secondary S3 bucket

S3 CRR로 Primary -> Secondary 복제
```

이 구조에서 CloudFront는 사용자에게 가까운 Edge Location 캐싱을 제공하고, S3 CRR은 리전 수준의 복제본을
유지합니다.

## 시험 함정

| 문제 표현                                        | 정답 방향  |
| ------------------------------------------------ | ---------- |
| "Static content must be available everywhere"    | CloudFront |
| "Files are cached for a TTL"                     | CloudFront |
| "Global edge network"                            | CloudFront |
| "Must be set up for each region"                 | S3 CRR     |
| "Files updated in near real-time across regions" | S3 CRR     |
| "Low latency in a few specific regions"          | S3 CRR     |
| "Reduce origin load through caching"             | CloudFront |

## 한 줄 요약

CloudFront는 전 세계 Edge Location에 캐싱해 읽기 성능을 높이는 CDN이고, S3 CRR은 S3 객체를 지정한
다른 리전 버킷으로 복제하는 기능입니다.
