# AWS Athena Trouble Shooting

## Trouble Shooting

---

원인을 찾았습니다.

WhereClauseBuilder.buildDateRangeWhere()에서 날짜 범위를 월 단위로 분할할 때 day BETWEEN을 사용하고 있습니다.

2026년 5월 20일~26일 검색 시 생성되는 WHERE 절:

(year = 2026 AND month = 5 AND day BETWEEN 20 AND 26)

이 쿼리 자체는 정상입니다. 문제는 S3 파티션 키의 타입에 있을 가능성이 높습니다.

S3 경로가 year=2026/month=05/day=21/로 되어 있는데, Athena 테이블의 파티션 컬럼이 string 타입으로 정의되어 있다면:

- day BETWEEN 20 AND 26 → 숫자 비교
- 파티션 값 '21' → 문자열

이 경우 Athena가 암묵적 형변환을 하면서 일부 파티션을 누락할 수 있습니다.

하지만 더 가능성 높은 원인은 Athena 파티션이 아직 day=21에 대해 등록되지 않은 것입니다. S3에 데이터가 있어도 Athena가 해당 파티션을 인식하지 못하면 쿼리 결과에 포함되지 않습니다.

확인 방법:

1. Athena에서 파티션 목록 확인:

SHOW PARTITIONS <database>.search_log;

2. 또는 MSCK REPAIR TABLE로 누락 파티션 자동 등록:

MSCK REPAIR TABLE <database>.search_log;

3. Glue Crawler가 설정되어 있다면, Crawler가 아직 day=21 파티션을 크롤링하지 않았을 수 있습니다.

────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────

요약: S3에 day=21/ 데이터가 존재하지만 백엔드 결과에 안 나오는 이유는, Athena 테이블에 해당 파티션이 등록되지 않았기 때문일 가능성이 가장 높습니다. MSCK REPAIR TABLE을 실행하거나, 파티션을 수동으로
추가(ALTER TABLE ADD PARTITION)하면 해결됩니다.
