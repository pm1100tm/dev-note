# coalesce - null 값을 유효한 값으로 반환하기

null 값을 찾는 방법은 is null 을 사용하는 방법입니다. 여기서 null 값을 다른 유효한 값으로 대채하려면
coalesce 내장 함수를 사용합니다. coalesce 는 모든 DBMS에 사용할 수 있습니다.

```sql
select empno, coalesce(mgr, 0) from emp;;
```
