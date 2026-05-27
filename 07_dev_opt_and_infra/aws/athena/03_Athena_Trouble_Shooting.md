# AWS Athena Trouble Shooting

## 🙋 S3에 여러 log 파일을 날짜별로 업로드 하였는데, 쿼리에 데이터가 없다고 나온다.

- Athena 파티션이 등록되지 않음
  - S3에 데이터가 있어도 Athena가 해당 파티션을 인식하지 못하면 쿼리 결과에 포함되지 않음
- Glue Crawler가 설정되어 있다면, Crawler가 아직 day=21(예시) 파티션을 크롤링하지 않았을 수도 있음

### 확인 방법

Athena에서 파티션 목록 확인:

```sql
SHOW PARTITIONS <database>.search_log;

```

또는 MSCK REPAIR TABLE로 누락 파티션 자동 등록:

```sql
MSCK REPAIR TABLE <database>.search_log;
```
