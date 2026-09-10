# PostgreSQL LATERAL

`LATERAL`은 `FROM` 절의 왼쪽에 있는 각 행을 오른쪽 서브쿼리에서 참조할 수 있게
합니다. 1:N 관계의 여러 행을 행별로 집계하거나, 각 행마다 상위 N개를 조회할 때
유용합니다.

## 기본 예제

```sql
SELECT a.rvw_id, imgs.imgs
FROM shop.review_desc AS a
LEFT JOIN LATERAL (
    SELECT ARRAY_AGG(i.img_url ORDER BY i.img_seq) AS imgs
    FROM shop.review_img AS i
    WHERE i.rvw_id = a.rvw_id
) AS imgs ON true;
```

서브쿼리 안에서 `a.rvw_id`를 참조할 수 있기 때문에 리뷰마다 이미지 배열을 만들 수
있습니다. `LEFT JOIN`을 사용하면 이미지가 없는 리뷰도 결과에 남고, 배열 값은 NULL이
될 수 있습니다.

## `ON true`의 의미

서브쿼리의 필터는 내부 `WHERE`에서 이미 처리하므로, 외부 조인 조건은 `ON true`로
작성하는 경우가 많습니다. 이는 왼쪽 행마다 서브쿼리의 결과를 붙인다는 의미이지,
서브쿼리가 실제로 비용 없이 실행된다는 뜻은 아닙니다.

## 행별 상위 N개 조회

```sql
SELECT p.id, latest.created_at, latest.price
FROM product AS p
LEFT JOIN LATERAL (
    SELECT h.created_at, h.price
    FROM price_history AS h
    WHERE h.product_id = p.id
    ORDER BY h.created_at DESC
    LIMIT 1
) AS latest ON true;
```

`price_history(product_id, created_at DESC)`와 같은 인덱스를 함께 검토하면 제품별
최신 행을 효율적으로 찾을 수 있습니다. 실제 효과는 데이터 분포와 실행 계획으로
확인해야 합니다.

## LATERAL과 CTE 비교

| 구분 | LATERAL | CTE(`WITH`) |
| --- | --- | --- |
| 참조 범위 | 왼쪽 행의 컬럼을 참조 | 쿼리에서 정의한 이름을 참조 |
| 주요 용도 | 행별·상관 서브쿼리 | 복잡한 쿼리 분리·재사용 |
| 실행 특성 | 행마다 반복될 수 있음 | 계획과 사용 방식에 따라 달라짐 |

CTE가 항상 한 번만 실행되거나 LATERAL이 항상 느린 것은 아닙니다. PostgreSQL 버전,
`MATERIALIZED` 여부, 선택도와 인덱스에 따라 달라지므로 `EXPLAIN (ANALYZE, BUFFERS)`로
검증합니다.

## 조인 조건 주의

`ON false`는 외부 조인의 결과를 항상 매칭되지 않게 만들어 오른쪽 컬럼을 NULL로
반환합니다. 이를 “서브쿼리 실행을 생략하는 제어문”으로 이해하면 안 됩니다. 조건부로
결과를 붙이려면 `ON (a.status = 'ACTIVE')`처럼 명시적인 조인 조건을 사용합니다.
