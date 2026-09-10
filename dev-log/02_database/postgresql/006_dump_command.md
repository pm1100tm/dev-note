# 🚀 pg_dump 관련 명령어 정리

## SQL 로 dump

```shell
# 덤프 (*filename 확장자는 .sql 로 통일함)
pg_dump -U <user> -d <database_name> -t <schema> > <filename>

# 복원
psql -U <user> -d <database_name> -f <filename>
```

## Custom 형식 (압축 지원, pg_restore로 복원)

```shell
# 덤프 (*filname 확장자는 .bk 로 통일함)
pg_dump -U <user> -d <database_name> -n <schema> -Fc -f <filename>
pg_restore -U <user> -d <database_name> -n <schema> <filename>

# 복원
psql -U <user> -d <database_name> -f <filename>
```

---

### sample

```shell
# -----------------------------------------------------------------------
# SQL 로 dump
# -----------------------------------------------------------------------
# 덤프하기
pg_dump -U root -d postgres -t shop.product_info > product_info_bk_20250928.sql

# 복원하기(복원은 위에서 덤프한 파일을 그대로 흘려 보내기만 하면 됨)
psql -U root -d postgres -f product_info_bk_20250928.sql



# -----------------------------------------------------------------------
# Custom 형식 (압축 지원, pg_restore로 복원)
# -----------------------------------------------------------------------
# 덤프하기
pg_dump -U root -d postgres -n shop -Fc -f shop_schema.backup

# 복원하기
pg_restore -U root -d postgres -n shop shop_schema.backup
```
