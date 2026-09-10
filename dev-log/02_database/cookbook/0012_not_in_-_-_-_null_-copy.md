# NOT IN 을 where 절에 사용할 때 null 값 주의하기

```sql
-- 기존 부서 테이블
select deptno from dept; -- 10, 20, 30, 40

-- 새로운 부서 테이블
create table new_dept(deptno integer);
insert into new_dept values (10), (50), (null);
select deptno from new_dept; -- 10, 50, null
```

테이블과 row 값이 위와 같을 때 deptno 에서 new_dept 테이블의 부서 정보를 제외하고 가져오려면
역시 아래과 같은 쿼리가 가능합니다.

```sql
select deptno from dept
except
select deptno from new_dept;
-- 40
-- 30
-- 20
```

그러나 where 절에서 not in 연산자를 사용한다면, 아무런 행도 반환하지 않습니다. 이유는 간단하게도
new_dept 테이블에 null 값이 존재하기 때문입니다.

```sql
select
    deptno
from
    dept
where
    deptno not in (select deptno from new_dept)
;
```

SQL에서 TRUE or NULL은 TRUE 이지만, FALSE or NULL 은 NULL 입니다.

not in 및 null 문제를 방지하려면 not exists 와 함께 서브쿼리를 사용하는 것이 좋습니다.

```sql
select
    deptno
from
    dept as d
where
    not exists (
        select 1 from new_dept as nd where nd.deptno = d.deptno
    )
;
```
