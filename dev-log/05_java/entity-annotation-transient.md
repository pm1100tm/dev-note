# JPA Entity Annotation

## @Transient

엔티티 클래스에서 특정 필드를 DB와 매핑하지 않도록 하며, 저장/조회 시 무시된다.

- INSERT, UPDATE, SELECT 쿼리에 포함되지 않는다.
- 컬럼으로 생성되지 않는다.
- 런타임 동안 엔티티 객체의 임시 데이터를 저장하거나 계산된 값을 저장하는데 유용하다.
- DB와 관련이 없는 비지니스 로직이나 DTO 변환에 사용되는 값들을 저장할 때 활용된다.

```java
@Entity
public class Order {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    private int quantity;

    private double pricePerUnit;

    @Transient
    private double totalPrice; // 데이터베이스와 매핑되지 않음

    public double getTotalPrice() {
        return this.quantity * this.pricePerUnit;
    }
}
```

### 영속성 컨텍스트와의 관계

@Transient 필드는 JPA가 관리하지 않으므로 영속성 컨텍스트에서 변경 상태를 추적하지 않는다.

### DTO 변환 시 사용

DB 매핑이 필요 없는 값을 포함해야 할 경우 유용하게 사용할 수 있다.

### 조회 성능 최적화

데이터베이스에서 필요하지 않은 컬럼은 @Transient로 처리하여 불필요한 데이터 조회를 방지한다.
