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

## 사용 시 주의점

`EXCEPT`를 비롯한 집합 연산자는 두 SELECT의 컬럼 수와 호환 가능한 데이터 타입이
일치해야 합니다. `EXCEPT`는 기본적으로 중복을 제거하며, 중복을 유지하려면
`EXCEPT ALL`을 사용합니다. `NOT IN`과 달리 비교 대상에 `NULL`이 있어도 전체 결과가
사라지는 문제가 없습니다.
