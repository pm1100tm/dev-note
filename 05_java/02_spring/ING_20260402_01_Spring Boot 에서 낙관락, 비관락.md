# 🚀 Spring Boot 에서 낙관락(Optimistic Lock), 비관락(Pessimistic Lock)

데이터베이스를 사용하는 서비스에서 가장 중요한 것 중 하나는
👉 **동시성 제어(Concurrency Control)**입니다.

여러 사용자가 동시에 같은 데이터를 수정할 때,
👉 데이터가 꼬이는 문제를 반드시 방지해야 합니다.

이때 사용하는 대표적인 방법이 바로:

- 낙관적 락 (Optimistic Lock)
- 비관적 락 (Pessimistic Lock)

입니다.

<br>

## 📌 1. 낙관적 락 (Optimistic Lock)

🔍 개념

“충돌이 거의 발생하지 않을 것이라고 가정하고, 문제가 생기면 그때 처리하는 방식입니다.”

### ✅ 구현 방법 (Spring / JPA)

```java
@Entity
public class Product {

    @Id
    private Long id;

    private int stock;

    @Version
    private Long version;
}
```

### 🔍 동작 방식

- 1. 데이터를 조회할 때 version 값도 함께 조회
- 2. 업데이트 시 version 조건을 포함

```sql
UPDATE product
SET stock = ?, version = version + 1
WHERE id = ? AND version = ?
```

👉 version이 다르면 업데이트 실패

❗ 예외 발생

```java
OptimisticLockException
```

<br>

## 📌 2. 비관적 락 (Pessimistic Lock)

🔍 개념

“충돌이 발생할 것이라고 가정하고, 미리 락을 걸어버리는 방식입니다.”

### ✅ 구현 방법

```java
@Lock(LockModeType.PESSIMISTIC_WRITE)
@Query("SELECT p FROM Product p WHERE p.id = :id")
Product findByIdForUpdate(Long id);
```

#### 🔍 실제 SQL

```sql
SELECT * FROM product WHERE id = ? FOR UPDATE;
```

👉 다른 트랜잭션은 대기 상태로 들어감

## 📌 낙관락 vs 비관락 비교

| 구분        | 낙관락         | 비관락          |
| ----------- | -------------- | --------------- |
| 락 방식     | 충돌 시 검증   | 미리 락         |
| 성능        | 좋음           | 상대적으로 느림 |
| 충돌 처리   | 실패 후 재시도 | 대기            |
| DB 부하     | 낮음           | 높음            |
| 데드락 위험 | 없음           | 있음            |

<br>

## 📌 실무에서 자주 쓰이는 방식

### ✅ 결론

👉 대부분 낙관락을 사용합니다

### 🔍 이유

1. 트래픽 환경

- 대부분 서비스는 충돌이 자주 발생하지 않음
- 따라서 미리 락을 거는 것은 비효율적

2. 성능

- 낙관락은 락이 없기 때문에 빠름
- DB 커넥션 점유 없음

3. 확장성

- MSA / 분산 환경에서도 유리

<br>

---

## ❗ 언제 비관락을 사용할까?

👉 아래 상황에서는 비관락이 필요합니다

```markdown
- 재고 감소 (선착순 구매)
- 좌석 예약 시스템
- 중복 결제 방지
- 금융 거래
```

---

## 📌 더 좋은 코드 (실무 Best Practice)

### ✅ 낙관락 + 재시도 전략

```java
@Transactional
public void decreaseStock(Long productId, int quantity) {
    for (int i = 0; i < 3; i++) {
        try {
            Product product = repository.findById(productId)
                .orElseThrow();

            product.decrease(quantity);
            return;
        } catch (OptimisticLockException e) {
            // retry
        }
    }
    throw new RuntimeException("재시도 실패");
}
```

#### 🔍 왜 중요한가?

- 낙관락은 “실패”가 정상 흐름입니다
- 반드시 재시도 로직 필요

<br>

### ✅ 비관락 사용 예

```java
@Transactional
public void decreaseStock(Long productId, int quantity) {
    Product product = repository.findByIdForUpdate(productId);
    product.decrease(quantity);
}
```

<br>

---

## 📌 실무 기준 선택 가이드

| 상황             | 추천   |
| ---------------- | ------ |
| 일반 CRUD        | 낙관락 |
| 트래픽 많음      | 낙관락 |
| 충돌 거의 없음   | 낙관락 |
| 재고/결제        | 비관락 |
| 정합성 100% 필요 | 비관락 |

## 📌 핵심 정리

- 낙관락은 성능 중심 전략입니다
- 비관락은 안정성 중심 전략입니다
- 실무에서는 낙관락 + 재시도가 기본입니다
