# 🚀 쿼리 결과를 열이 아닌 행으로 취득하는 방법(Vertically)

쿼리 실행 결과는 기본적으로 각 컬럼에 대해서 행으로 표시되는데, 하나의 결과 행에 대해서는 새로로 표시하면
좀 더 가독성이 좋을 때가 있습니다.

그럴 때 아래의 함수를 조합하여 쿼리 결과를 새로로 볼 수 있습니다.

## jsonb_each_text, to_jsonb 함수 조합

```sql
select
    key, value
from
    jsonb_each_text(
        to_jsonb(
            (select t from <your_table_name> as t where t.table_pk = 'PK_111')
        )
    )
;
```

결과 값은 아래와 같은 형태로 표시됩니다.

| Key        | Value               |
| ---------- | ------------------- |
| pk         | PK_00000000         |
| desc       | 설명설명설명...     |
| created_at | 2022-03-02T18:46:43 |
| updated_at | 2022-03-02T18:46:43 |

> 참고로 PostgreSQL 을 사용하였습니다.
