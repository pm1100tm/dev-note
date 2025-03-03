# 열 값 이어 붙이기

여러 열의 값을 하나의 열로 반환하려고 하는 방법은 아래와 같습니다.

```sql
select
    ename || ' WORK AS A ' || job as msg
from
    emp
;
```

- mysql 은 concat 함수를 사용합니다.
