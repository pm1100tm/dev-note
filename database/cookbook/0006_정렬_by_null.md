# 정렬 null

필드가 null 을 허용할 때, null 을 마지막에 정렬할지를 지정하는 방법이 필요합니다.

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
