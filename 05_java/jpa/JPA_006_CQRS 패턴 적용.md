# CQRS 패턴 적용

CQRS는 명령(Command)과 조회(Query)의 책임을 분리해 각각의 목적에 맞게
설계하는 패턴입니다. JPA를 쓰는 애플리케이션에서는 명령에 도메인 규칙과
엔티티를 사용하고, 조회에는 화면에 필요한 DTO를 직접 조회하는 방식으로
작게 시작할 수 있습니다.

CQRS가 읽기 데이터베이스를 별도로 두어야 한다는 뜻은 아닙니다. 처음에는
같은 데이터베이스와 같은 트랜잭션 경계를 사용해도 충분하며, 읽기 부하나
모델의 차이가 실제로 커질 때만 별도 저장소와 동기화 방식을 검토합니다.

## 목차

- [CQRS의 개념과 필요한 이유](#cqrs의-개념과-필요한-이유)
- [명령 모델: 상태와 규칙을 함께 변경하기](#명령-모델-상태와-규칙을-함께-변경하기)
- [조회 모델: DTO를 필요한 모양으로 조회하기](#조회-모델-dto를-필요한-모양으로-조회하기)
- [읽기 전용 트랜잭션의 의미](#읽기-전용-트랜잭션의-의미)
- [같은 데이터베이스에서 시작하는 구조](#같은-데이터베이스에서-시작하는-구조)
- [별도 읽기 모델로 확장할 때](#별도-읽기-모델로-확장할-때)
- [실무 적용 기준](#실무-적용-기준)
- [참고 자료](#참고-자료)

## CQRS의 개념과 필요한 이유

Command는 시스템 상태를 바꾸는 요청이고, Query는 상태를 바꾸지 않고
데이터를 반환하는 요청입니다. CQRS(Command Query Responsibility
Segregation)는 이 두 책임에 다른 모델과 코드를 사용할 수 있게 합니다.

| 구분      | Command                    | Query                             |
| --------- | -------------------------- | --------------------------------- |
| 목적      | 주문 생성, 취소, 재고 차감 | 주문 목록, 관리자 통계, 상세 화면 |
| 중심 모델 | 엔티티와 도메인 규칙       | DTO와 조회 성능                   |
| 반환      | 식별자 또는 처리 결과      | 화면·API에 맞는 읽기 모델         |
| 트랜잭션  | 쓰기 트랜잭션              | 일반적으로 읽기 전용 트랜잭션     |

하나의 엔티티로 읽기와 쓰기를 모두 해결하려 하면 서로 다른 요구가 충돌합니다.
주문 취소는 재고 복구와 상태 전이 규칙이 중요하지만, 주문 목록은 회원명,
상품명, 금액처럼 여러 테이블의 일부 값만 빠르게 보여 주는 일이 중요합니다.

### 잘못된 예: 목록 응답에 엔티티를 그대로 사용하기

```java
@GetMapping("/orders")
public List<Order> orders() {
    return orderRepository.findAll();
}
```

이 코드는 API가 엔티티 구조에 묶입니다. 지연 로딩 관계를 JSON 변환 중에
접근하면 N+1 쿼리나 예외가 생길 수 있고, 내부 필드가 의도치 않게 노출될 수
있습니다. 목록에 필요 없는 컬럼까지 조회할 가능성도 큽니다.

### 올바른 방향: 쓰기와 읽기의 관심사를 분리하기

```text
POST /orders  -> OrderCommandService -> Order, OrderItem -> DB
GET  /orders  -> OrderQueryService   -> OrderSummary    -> DB
```

명령 서비스는 `Order`가 지켜야 할 규칙을 실행합니다. 조회 서비스는
`OrderSummary`처럼 반환할 모양을 명시하고 필요한 컬럼만 조회합니다. 두
서비스가 처음부터 서로 다른 데이터베이스를 사용해야 하는 것은 아닙니다.

## 명령 모델: 상태와 규칙을 함께 변경하기

명령은 단순한 setter 호출이 아니라 "무엇을 바꾸며, 어떤 조건을 지켜야 하는가"를
표현해야 합니다. 예를 들어 배송 완료 주문은 취소할 수 없고, 취소하면 주문 항목의
재고를 복구해야 할 수 있습니다.

```java
@Entity
public class Order {

    @Id
    @GeneratedValue
    private Long id;

    @Enumerated(EnumType.STRING)
    private OrderStatus status;

    protected Order() {
    }

    public void cancel() {
        if (status == OrderStatus.DELIVERED) {
            throw new IllegalStateException("배송 완료 주문은 취소할 수 없습니다.");
        }
        if (status == OrderStatus.CANCELLED) {
            return;
        }
        status = OrderStatus.CANCELLED;
    }
}
```

### 잘못된 예: Controller에서 엔티티 상태를 직접 바꾸기

```java
@PatchMapping("/orders/{id}/cancel")
public void cancel(@PathVariable Long id) {
    Order order = orderRepository.findById(id).orElseThrow();
    order.setStatus(OrderStatus.CANCELLED);
    orderRepository.save(order);
}
```

상태 전이 규칙이 Controller, 서비스, 배치마다 흩어집니다. 배송 완료 여부나
재고 복구를 빠뜨린 호출 경로가 생기며, 영속 상태 엔티티에 불필요한 `save()`를
호출하게 됩니다. `save()`는 업무 규칙을 검증하거나 즉시 커밋하는 명령이 아닙니다.

### 올바른 예: 명령 서비스가 트랜잭션과 흐름을 맡기

```java
@Service
@RequiredArgsConstructor
public class OrderCommandService {

    private final OrderRepository orderRepository;

    @Transactional
    public void cancel(Long orderId) {
        Order order = orderRepository.findById(orderId)
            .orElseThrow(() -> new IllegalArgumentException("주문이 없습니다."));

        order.cancel();
    }
}
```

트랜잭션 안에서 조회한 `Order`는 영속 상태입니다. `cancel()`이 상태를 바꾸면
JPA 변경 감지가 flush 시점에 SQL을 만듭니다. 서비스는 트랜잭션과 여러 객체의
작업 순서를 맡고, 엔티티는 자신이 지켜야 하는 규칙을 맡으므로 책임이 명확합니다.

위의 `@Transactional`은 Spring 기능입니다. JPA 표준은 `EntityManager`와
영속성 컨텍스트를 정의하지만, Spring의 선언적 트랜잭션 프록시를 정의하지는
않습니다.

## 조회 모델: DTO를 필요한 모양으로 조회하기

조회는 변경 감지할 엔티티 그래프가 아니라 응답에 필요한 데이터 모양이 중심입니다.
Spring Data JPA에서는 인터페이스 기반 Projection, 클래스·record DTO, JPQL
생성자 표현식 등을 선택할 수 있습니다.

```java
public record OrderSummary(
    Long orderId,
    String memberName,
    OrderStatus status,
    long totalAmount
) {
}
```

```java
public interface OrderQueryRepository {

    @Query("""
        select new com.example.order.OrderSummary(
            o.id, m.name, o.status, sum(i.price * i.quantity)
        )
        from Order o
        join o.member m
        join o.items i
        group by o.id, m.name, o.status
        order by o.id desc
        """)
    List<OrderSummary> findSummaries();
}
```

JPQL의 DTO 생성자 표현식은 JPA 표준 문법입니다. 클래스 이름은 전체 패키지
이름을 써야 하며, 선택한 값 순서와 생성자 또는 record의 구성 요소 순서가
일치해야 합니다. Spring Data의 Projection 편의 기능과는 구분해야 합니다.

### 잘못된 예: Query 서비스에서 엔티티를 수정하기

```java
@Transactional(readOnly = true)
public OrderSummary getOrder(Long orderId) {
    Order order = orderRepository.findById(orderId).orElseThrow();
    order.markViewed();
    return OrderSummary.from(order);
}
```

조회 메서드에 상태 변경이 섞이면 호출자가 URL이나 메서드 이름만 보고 부수 효과를
예측할 수 없습니다. 또한 `readOnly = true`는 모든 데이터베이스와 JPA 구현체에서
쓰기 SQL을 차단하는 보안 장치가 아닙니다. 설정에 따라 변경이 flush될 가능성도
있으므로 조회 메서드의 변경을 허용해서는 안 됩니다.

### 올바른 예: Query 서비스와 DTO 반환을 분리하기

```java
@Service
@RequiredArgsConstructor
public class OrderQueryService {

    private final OrderQueryRepository orderQueryRepository;

    @Transactional(readOnly = true)
    public List<OrderSummary> findOrders() {
        return orderQueryRepository.findSummaries();
    }
}
```

DTO는 영속 엔티티가 아니므로 변경 감지와 지연 로딩의 대상이 아닙니다. 반환
데이터가 명시적이고, 목록 API에 맞는 조인·집계·인덱스도 조회 요구에 맞춰
검토하기 쉬워집니다.

## 읽기 전용 트랜잭션의 의미

`@Transactional(readOnly = true)`는 Spring 트랜잭션 정의의 힌트입니다. 실제
효과는 `PlatformTransactionManager`, JDBC 드라이버, 데이터베이스, JPA 구현체의
조합에 따라 달라집니다.

- Spring은 JDBC Connection에 읽기 전용 힌트를 전달할 수 있습니다.
- Hibernate를 쓰는 일반적인 Spring JPA 구성에서는 flush 동작과 세션의 기본
  읽기 전용 상태를 조정해 스냅샷 관리 비용을 줄일 수 있습니다.
- 데이터베이스가 읽기 전용 트랜잭션을 엄격히 강제하는지는 DB와 설정에 따라
  다릅니다. 권한 검증이나 쓰기 차단을 이것만으로 보장하면 안 됩니다.

따라서 모든 조회 서비스에 관례적으로 붙이기보다, 실제 읽기 경계에는 붙이고
쓰기 또는 감사 기록이 필요한 메서드는 분리합니다. Hibernate 최적화 효과를
기대한다면 사용하는 Spring Framework와 Hibernate 버전에서 SQL 및 flush 동작을
테스트로 확인해야 합니다.

## 같은 데이터베이스에서 시작하는 구조

대부분의 업무 시스템은 다음처럼 같은 DB에서 CQRS를 시작하는 것이 적절합니다.
코드 책임만 분리해도 엔티티 노출, N+1, 복잡한 조회가 쓰기 규칙을 침범하는 문제를
크게 줄일 수 있습니다.

```text
api
├── OrderCommandController -> OrderCommandService
│                              -> OrderRepository
├── OrderQueryController   -> OrderQueryService
│                              -> OrderQueryRepository
└── domain
    ├── Order, OrderItem, Member
    └── OrderRepository
```

`OrderRepository`는 명령이 필요한 aggregate 조회에 집중하고,
`OrderQueryRepository`는 DTO 조회에 집중합니다. 둘은 같은 테이블을 읽어도
됩니다. Querydsl을 사용한다면 그것은 JPA 표준이 아니라 별도 라이브러리이므로,
타입 안전한 동적 조회가 실제로 필요한 곳에만 도입합니다.

조회 DTO를 API 요청 DTO나 명령 객체로 재사용하지 않습니다. 조회용 필드가 추가된
DTO가 명령 API의 입력 계약까지 바꾸는 일을 막을 수 있습니다.

## 별도 읽기 모델로 확장할 때

읽기 트래픽이 압도적으로 크거나, 검색·통계 화면이 운영 트랜잭션의 테이블 구조와
매우 다르면 읽기 모델을 별도 저장소에 둘 수 있습니다. 예를 들어 주문 생성
트랜잭션이 완료된 뒤 이벤트를 발행하고, 검색용 문서 또는 집계 테이블을 갱신합니다.

이 단계에서는 반드시 일관성 모델을 API 계약으로 드러내야 합니다. 쓰기 직후의
`GET`이 읽기 저장소에서 이전 값을 반환할 수 있으며, 이벤트 중복 전달과 처리
실패도 고려해야 합니다. outbox, 멱등 소비자, 재처리·모니터링 없이 단순히 비동기
복제를 추가하면 데이터 불일치가 운영 장애가 될 수 있습니다.

강한 일관성이 필요한 주문 직후 응답이나 재고 검증은 쓰기 DB를 읽습니다. 약간의
지연을 허용하는 검색 목록·통계만 별도 읽기 모델로 옮기는 것이 안전한 출발점입니다.

## 실무 적용 기준

- 단순 CRUD와 조회 요구가 비슷하면 CQRS 패키지를 과도하게 쪼개지 않습니다.
- 상태를 바꾸는 유스케이스에는 명시적인 명령 서비스와 짧은 트랜잭션을 둡니다.
- 목록·검색·통계는 DTO를 직접 조회하고 생성 SQL과 실행 계획을 확인합니다.
- 엔티티를 Controller 응답으로 노출하지 않고, 조회 DTO와 명령 입력을 분리합니다.
- 별도 읽기 저장소는 성능 측정과 일관성 요구가 확인된 뒤 도입합니다.
- 명령과 조회를 분리해도 재고, 중복, 권한 같은 정합성 규칙은 DB 제약 조건과
  트랜잭션·동시성 제어로 별도 보장해야 합니다.

## 참고 자료

- [Spring Data JPA: Projections][spring-data-projections]
- [Jakarta Persistence 3.2][jakarta-persistence]

[spring-data-projections]: https://docs.spring.io/spring-data/jpa/reference/repositories/projections.html
[jakarta-persistence]: https://jakarta.ee/specifications/persistence/3.2/
