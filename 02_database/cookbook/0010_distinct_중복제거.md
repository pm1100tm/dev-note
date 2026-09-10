# DISTINCT 와 DISTINCT ON

## DISTINCT

DISTINCT 와 DISTINCT ON 은 둘 다 중복된 값을 제거할 때 사용됩니다. 그러나, DISTINCT는 선택한
모든 컬럼을 기준으로 중복된 행을 제거합니다. 즉, 결과에서 중복된 값이 완전히 제거됩니다.

즉, 여러 컬럼을 선택한 경우, 모든 컬럼 값이 동일한 행만 중복으로 간주됩니다.

```sql
SELECT DISTINCT deptno, ename FROM emp;
```

위의 쿼리는 deptno 과 ename 의 조합이 중복되는 행을 제거한 후 결과를 반환합니다. 즉, 두 컬럼의 값이
모두 동일한 행만 제거됩니다.

## DISTINCT ON

DISTINCT ON 은 특정 컬럼에 대해서만 중복을 제거합니다. 특정 컬럼에 대해 중복된 값 중에서 첫 번째
값을 반환하고, 나머지는 무시합니다. 이 때, 어떤 값을 반환할지 결정하기 위해서 order by 절을 함께
사용해야 합니다.

```sql
SELECT
    DISTINCT ON (deptno) deptno, ename, sal
FROM
    emp
ORDER BY
    deptno, sal DESC
;
```

이 쿼리는 각 deptno 에 대해 가장 높은 sal 값을 가진 행만 반환합니다. 예를 들어, deptno = 10 인
여러 행이 있을 때, 그 중에서 sal 값이 가장 큰 하나의 행만 반환됩니다.

## 사용 시 주의점

- `DISTINCT`는 정렬이나 해시 집계 비용을 추가할 수 있으므로 중복 제거가 정말 필요한지
  확인합니다.
- 같은 원칙으로 중복이 허용되는 경우에는 `UNION`보다 `UNION ALL`이 적합합니다.
- `DISTINCT ON`을 사용할 때 `ORDER BY`의 가장 왼쪽 표현식은 `DISTINCT ON` 표현식과
  일치해야 합니다.
