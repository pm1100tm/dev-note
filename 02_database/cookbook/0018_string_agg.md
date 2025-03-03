# STRING_AGG 함수

1:N 관계의 테이블을 쿼리하면, 1 쪽의 데이터가 중복되서 출력될 수 있습니다. 예를 들어, emp_bonus
테이블은 보너스를 받은 날짜와 보너스의 타입을 기록하는 테이블인데, 같은 사원이 여러 개의 보너스 레코드를
소유할 수 있게 됩니다. 즉, 우수한 사원은 보너스를 여러 번 받을 수 있다는 가정입니다.

그러면 여기서 사원 번호를 유니크하게 출력하되, 보너스를 받은 날짜를 콤마(,)로 구분하여 하나의 문자열로
출력하려면 어떻게 해야 할까요?

- 1. 사원번호를 기준으로 데이터를 그룹화합니다.
- 2. GROUP_CONCAT 함수(MySQL 지원), 또는 LISTAGG(ORACLE 지원) 함수를 사용합니다.
- 3. PostgreSQL 에서는 STRING_AGG 함수를 사용할 수 있습니다.

```sql
select * from emp_bonus order by empno;

-- EB_000000004,2005-02-15,1,7782
-- EB_000000003,2005-02-15,3,7839
-- EB_000000001,2005-03-17,1,7934
-- EB_000000002,2005-02-15,2,7934
```

위의 데이터를 보면 7934 사원번호는 2번의 보너스를 받을 것을 확인할 수 있습니다. 그럼 이제
STRING_AGG 함수를 사용하여, 원하던대로 데이터를 출력해보겠습니다.

```sql
select
    e.ename,
    eb.empno,
    string_agg(
        to_char(eb.received, 'YYYY-MM-DD'),
        ', ' order by received
    ) as bonus_dates
from
    emp_bonus as eb
inner join emp as e
    on e.empno = eb.empno
group by
    e.ename,
    eb.empno
order by
    empno
;

-- CLARK,7782,2005-02-15
-- KING,7839,2005-02-15
-- MILLER,7934,"2005-02-15, 2005-03-17"
```

이렇게 하면 원하던 목적을 이룰 수 있습니다. 그런데 여기서 문제가 생길 수 있겠다 싶은 부분이 있습니다.
그룹 처리된 열의 received 는 스트링 타입인데, 그룹 처리되지 않은 received 값들은 date 타입으로
나올 수 있지 않을까 하는 의심입니다. 그래서 결과 값의 타입을 같이 출력해보았습니다.

```sql
select
    x.ename || ' | ' || pg_typeof(x.ename),
    x.empno || ' | ' || pg_typeof(x.ename),
    x.bonus_dates || ' | ' || pg_typeof(x.bonus_dates)
from
(
    select
        e.ename as ename,
        eb.empno as empno,
        string_agg(
            to_char(eb.received, 'YYYY-MM-DD'),
            ', ' order by received
        ) as bonus_dates
    from
        emp_bonus as eb
    inner join emp as e
        on e.empno = eb.empno
    group by
        e.ename,
        eb.empno
    order by
        empno
) as x
;

-- CLARK | character varying,7782 | character varying,2005-02-15 | text
-- KING | character varying,7839 | character varying,2005-02-15 | text
-- MILLER | character varying,7934 | character varying,"2005-02-15, 2005-03-17 | text"
```

보너스를 한 번 받은 레코드의 received 컬럼의 값도 text 로 변환되어 출력되었습니다.
사실 STRING_AGG 함수를 잘 알고 있다면, 데이터 리턴 타입을 의심하지 않아도 되었습니다.

## STRING_AGG 함수에 대해서 in PostgreSQL

여러 행의 문자열 값을 하나의 문자열로 결합할 때 사용됩니다. 이 함수는 집계 함수로서 그룹화된 데이터를
문자열로 연결할 수 있습니다. 여러 값들을 특정 구분자로 연결하여 단일 문자열로 만듭니다. 주로 데이터를
집계하여 리포트나 통계 정보를 생성할 때 유용합니다.

```sql
STRING_AGG(expression, delimiter [ORDER BY expression])
```

- expression: 결합할 문자열 값입니다.
- delimiter: 문자열 값을 구분할 구분자입니다. 예를 들어, ', '는 값들 사이에 콤마와 공백을 추가합니다.
- ORDER BY expression (선택적): 결합할 값들을 정렬할 기준입니다. 이 옵션을 사용하면 값들을
  지정된 순서로 정렬한 후 결합합니다.

### 주의사항

STRING_AGG 함수는 매우 큰 문자열을 처리할 때 성능에 영향을 미칠 수 있으므로, 대량의 데이터를 처리
할 때는 주의해야 합니다. PostgreSQL의 기본 문자열 길이 제한은 1GB까지 지원하므로, 일반적인 사용에
서는 큰 문제는 없습니다.
