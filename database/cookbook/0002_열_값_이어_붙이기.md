# Where 절에서 별칭이 지정된 열 참조하기

결과셋에 별칭을 적용하고 where 절을 사용하여 일부 행을 제외하려고 할 때, where 절에서 별칭을 참조
하려고 하면 실패합니다. 원인은 where 절은 select 절을 시행하기 전에 판단되기 되므로, where 절을
평가하려고 할 때 별칭을 사용할 수 없습니다. 이것을 해결하기 위해서 원래의 쿼리를 from 절에 배치하는
인라인 뷰를 활용하여, 별칭이 지정된 열을 참조할 수 있습니다.

```sql
select
    *
from
(
  select
      sal as salary,
      comm as commision
  from emp
) as t1
where
    salary < 1000
;
```
