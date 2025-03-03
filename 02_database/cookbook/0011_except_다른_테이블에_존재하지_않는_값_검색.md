# 다른 테이블에 존재하지 않는 값 검색하기

예를 들어 EMP 테이블에 없는 DEPT 테이블의 부서 정보를 찾으려고 할 때, 차집합 수행 연산이 있다면
유용합니다.

```sql
select deptno from dept; -- 10, 20, 30, 40

select distinct deptno from emp order by deptno ; -- 10, 20, 30

-- postgresql, db2, sql server
select deptno from dept
except
select deptno from emp
; -- 40

-- mysql
select
    deptno
from
    dept
where
    deptno not in (
        select deptno from emp
    )
;
```

## !important

except 를 비롯한 집합 연산자의 사용에는 제한이 있습니다. 비교할 데이터 유형 및 값 개수는 두
select 목록에서 일치해야 합니다. 또한 except 는 중복 항목을 반환하지 않으며, not in 을 사용하는
서브쿼리와 달리 null은 문제가 되지 않습니다.
