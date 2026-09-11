# CloudFront Price Class

CloudFront Edge Location은 전 세계에 분산되어 있고, 지역별 데이터 전송 비용은 다릅니다.

Price Class는 비용을 줄이기 위해 사용할 Edge Location 범위를 제한하는 설정입니다.

## Price Class 종류

| Price Class | 범위 | 특징 |
| --- | --- | --- |
| Price Class All | 모든 CloudFront edge location 사용 | 최고 성능, 가장 넓은 글로벌 커버리지 |
| Price Class 200 | 대부분의 region 사용, 가장 비싼 region 일부 제외 | 성능과 비용의 절충 |
| Price Class 100 | 가장 저렴한 region만 사용 | 비용 절감, 일부 지역 성능 저하 가능 |

## 성능과 비용 관점

| 목표 | 선택 |
| --- | --- |
| 전 세계 사용자에게 가장 낮은 latency 제공 | Price Class All |
| 비용을 줄이되 넓은 지역 커버리지 유지 | Price Class 200 |
| 비용 절감이 가장 중요 | Price Class 100 |

Price Class를 낮추면 일부 사용자는 가장 가까운 Edge Location을 사용하지 못할 수 있습니다.
따라서 latency가 증가할 수 있습니다.

## 시험 포인트

Price Class는 Origin 종류나 캐시 정책을 바꾸는 기능이 아닙니다.
사용 가능한 Edge Location 범위를 줄여 비용을 제어하는 기능입니다.

```text
Price Class All -> best performance
Price Class 200 -> excludes most expensive regions
Price Class 100 -> least expensive regions only
```

## 예시

서비스 사용자가 대부분 미국과 유럽에 있고, 일부 고비용 지역의 성능이 중요하지 않다면 Price Class 100
또는 200을 고려할 수 있습니다.

반대로 글로벌 사용자가 넓게 분산되어 있고 latency가 중요하다면 Price Class All이 적합합니다.

## 시험 함정

| 문제 표현 | 정답 방향 |
| --- | --- |
| "Reduce CloudFront cost by reducing edge locations" | Price Class |
| "Best performance globally" | Price Class All |
| "Use only least expensive regions" | Price Class 100 |
| "Most regions but exclude most expensive regions" | Price Class 200 |
| "Change TTL or cache key" | Price Class가 아님. Cache Policy |

## 한 줄 요약

CloudFront Price Class는 사용할 Edge Location 범위를 조절해 성능과 비용을 trade-off하는 설정입니다.
