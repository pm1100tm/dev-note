# 📚 SKU

✅ SKU란? (Stock Keeping Unit)

- → 재고 관리 단위
- → 시스템에서 상품을 구분하기 위한 최소 식별 코드

## 🏷️ IT 시스템에서 SKU의 역할

| 관점     | 의미                                 |
| -------- | ------------------------------------ |
| 비즈니스 | 상품 / 옵션 / 패키지를 구분하는 코드 |
| 시스템   | 상품을 식별하는 논리적 키            |
| 데이터   | DB, ERP, OMS, WMS에서 상품 연결 기준 |
| 개발     | 상품 관련 API, 주문, 재고의 기준값   |

👉 중요

- SKU ≠ DB PK
- SKU는 도메인 식별자
- PK는 기술 식별자

## 🛒 실무 예시 (이커머스)

- 상품: 나이키 티셔츠
- 옵션: 색상 = BLACK, 사이즈 = L

```shell
SKU = NIKE-TSHIRT-BLACK-L
```

| 필드            | 값                  |
| --------------- | ------------------- |
| product_id (PK) | 128391              |
| sku             | NIKE-TSHIRT-BLACK-L |
| color           | BLACK               |
| size            | L                   |

## 🧠 DDD 관점에서 SKU

✔️ SKU는 무엇인가?

- Value Object 로 보는 게 가장 자연스럽다
- 불변(immutable)
- 의미를 가진 식별 값

```java
public record Sku(String value) {
    public Sku {
        if (value == null || value.isBlank()) {
            throw new IllegalArgumentException("SKU must not be empty");
        }
    }
}
```

❌ 잘못된 설계

- SKU를 단순 String으로 여기저기 전달
- SKU 규칙을 Service나 Controller에서 검증

✅ 올바른 설계

- SKU 생성/검증 책임을 도메인에 둠
- SKU 변경은 도메인 이벤트를 동반

| 오해                    | 사실                   |
| ----------------------- | ---------------------- |
| SKU는 UUID다            | ❌ 사람이 만든 코드    |
| SKU는 절대 변경 안 된다 | ❌ 변경될 수 있음      |
| SKU = 상품              | ❌ 상품의 한 옵션 단위 |
| SKU는 기술 용어다       | ❌ 비즈니스 용어       |
