# 🚀 lateral 문법

LATERAL 은 PostgreSQL 의 강력한 기능 중 하나로,
**“왼쪽 테이블의 각 행(row)을 기준으로 서브쿼리를 실행”**할 수 있게 해주는 문법입니다.

## 🔍 1. LATERAL 이란?

LATERAL 키워드를 사용하면 왼쪽 테이블의 컬럼을 서브쿼리 안에서 참조할 수 있습니다.

즉,

LATERAL = “왼쪽 테이블의 각 행마다 이 서브쿼리를 실행해라”

✅ 예시

```sql
SELECT *
FROM shopper.review_desc AS a
LEFT JOIN LATERAL (
    SELECT ARRAY_AGG(i.img_url ORDER BY i.img_seq) AS imgs
    FROM shopper.review_img AS i
    WHERE i.rvw_id = a.rvw_id  -- ← 왼쪽 테이블의 컬럼 사용 가능!
) AS imgs ON true;
```

👉 이 쿼리는 a.rvw_id (리뷰 ID)마다
해당 리뷰의 이미지를 배열 형태로 합쳐서(ARRAY_AGG) 결과에 붙입니다.

## 🧠 2. ON TRUE 의 의미

ON TRUE 는 단순히 “항상 조인하라”는 뜻입니다.

보통 LEFT JOIN 은 ON 절에서 조인 조건을 지정하지만,
이미 서브쿼리 내부에 WHERE i.rvw_id = a.rvw_id 조건이 있기 때문에
ON TRUE 로 대체할 수 있습니다.

```sql
LEFT JOIN LATERAL (...) AS imgs ON true
```

➡ 즉,

“조건 없이 항상 이 LATERAL 서브쿼리를 실행하고 결과를 붙여라.”

## 💎 3. 왜 LATERAL 이 좋은가?

✅ 행 단위 연산에 강하다

각 행별로 서브쿼리를 실행하므로, 1:N 관계를 간단하게 1개의 행으로 압축할 수 있습니다.

예를 들어:
• 리뷰 1개 → 여러 이미지 (1:N 관계)
• LATERAL을 사용하면 리뷰 1개 행에 [img1, img2, img3] 배열을 바로 붙일 수 있음

```sql
LEFT JOIN LATERAL (
   SELECT ARRAY_AGG(i.img_url ORDER BY i.img_seq) AS imgs
   FROM shopper.review_img i
   WHERE i.rvw_id = a.rvw_id
) AS imgs ON true;
```

## ⚖️ 4. WITH (CTE) 와의 차이점

| 구분                       | LATERAL                                      | WITH (CTE)                                    |
| -------------------------- | -------------------------------------------- | --------------------------------------------- |
| 실행 단위                  | **행(row)** 단위로 실행                      | **쿼리 전체** 단위로 한 번만 실행             |
| 왼쪽 테이블 참조 가능 여부 | ✅ 가능 (`a.rvw_id`)                         | ❌ 불가능                                     |
| 주요 용도                  | 행별 서브쿼리, 동적 조인                     | 공통 서브쿼리 재사용, 복잡한 쿼리 가독성 향상 |
| 성능 특성                  | 각 행마다 실행될 수 있어 비용이 높을 수 있음 | 한 번만 실행되어 상대적으로 비용이 낮음       |
| 문법 위치                  | `FROM ... JOIN LATERAL (...)`                | `WITH ... AS (...)`                           |

## ⚙️ 5. ON FALSE 는 뭐지?

| 조건                 | 의미                                    | 결과                                    |
| -------------------- | --------------------------------------- | --------------------------------------- |
| `ON true`            | 항상 조인 성공 (모든 행에 LATERAL 실행) | LATERAL 서브쿼리가 실행되고 결과가 붙음 |
| `ON false`           | 항상 조인 실패 (조인 무효화)            | 항상 `NULL` 값이 붙음                   |
| `ON a.col = sub.col` | 조건부 조인                             | 조건이 맞는 행에만 LATERAL 실행         |

예시 비교
✅ ON true

```sql
LEFT JOIN LATERAL (
    SELECT ARRAY_AGG(i.url) AS imgs
    FROM product_image i
    WHERE i.product_id = a.id
) AS imgs ON true;
```

⚠️ ON false

```sql
LEFT JOIN LATERAL (
    SELECT ARRAY_AGG(i.url) AS imgs
    FROM product_image i
    WHERE i.product_id = a.id
) AS imgs ON false;
```

## 🧩 6. ON false 의 사용 사례

일반적으로 ON false 는 잘 쓰이지 않습니다.
하지만 조건부 LATERAL 실행 제어에 활용될 수 있습니다.

```sql
LEFT JOIN LATERAL (
    SELECT ...
    FROM ...
    WHERE ...
) AS sub ON (a.status = 'ACTIVE');
```

- a.status = 'ACTIVE' → 실행 (ON true)
- a.status != 'ACTIVE' → 생략 (ON false)

즉, ON false 는

“이 조인을 하지 말라”는 제어문 역할을 할 수 있습니다.
