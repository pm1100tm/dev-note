# 다중테이블 - 행 집합을 다른 행 위에 추가

두 개 이상의 테이블에 저장된 데이터를 반환하고, 한 결과셋을 다른 결과셋 위에 포개려고 합니다.
이틀 테이블에 공통 키가 필요하지 않다고 합니다. 집합 연산 UNION ALL 을 사용합니다.

```sql
select
    ename as ename_and_dname,
    deptno
from
    emp
where
    deptno = 10
union all
(select '-----------------', null)
union all
select
    dname,
    deptno
from
    dept
where
    deptno = 10
;

-- KING,10
-- CLARK,10
-- MILLER,10
-- -----------------,
-- ACCOUNTING,10
```

union all 은 여러 행 소스의 행들을 하나의 결과셋으로 결합합니다. 모든 집합 연산과 마찬가지로 모든
select 목록의 항목은 숫자와 데이터 유형이 일치해야 합니다.

union all 은 중복 항목이 있으면 이를 포함합니다. 중복을 필터링하려면 union 연산자를 사용합니다.

```sql
-- union all
select deptno from dept
union all
select deptno from dept
;

-- 10
-- 20
-- 30
-- 40
-- 10
-- 20
-- 30
-- 40

-- union
select deptno from dept
union
select deptno from dept
;

-- 10
-- 30
-- 40
-- 20

-- union
select deptno from dept
union
select deptno from dept
order by
    deptno
;

-- 10
-- 20
-- 30
-- 40
```

> union all 대신 union 을 지정하면 중복을 제거하는 정렬 작업이 발생할 가능성이 높습니다.
> 대량의 결과셋으로 작업할 때는 이 점을 염두에 두어야 합니다. union 을 사용하는 것은 union all 에
> distinct 를 적용하는 것과 거의 같습니다.

```sql
select deptno from dept
union all
select deptno from dept
order by
    deptno
;
```
