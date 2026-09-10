# NULL 값의 정렬 순서 지정하기

필드가 `NULL`을 허용할 때에는 `NULL`을 앞이나 뒤 어느 쪽에 배치할지 명시하는 편이
안전합니다. `NULLS FIRST`와 `NULLS LAST`는 PostgreSQL과 Oracle에서 지원됩니다.

```sql
select empno, comm from emp order by comm asc nulls last;
-- 7499,300.00
-- 7521,500.00
-- 7782,<null>

select empno, comm from emp order by comm desc nulls last;
-- 7521,500.00
-- 7499,300.00
-- 7782,<null>

select
    empno,
    case
        when comm is null then 0
        else 1
    end as is_null,
    comm
from
    emp
order by
    is_null desc , comm asc
;

-- 7499,1,300.00
-- 7521,1,500.00
-- 7782,0,<null>
-- 7566,0,<null>
```
