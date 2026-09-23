# @Modifying 완벽 정리

`@Modifying`은 Spring Data JPA의 `@Query`가 SELECT가 아닌 변경 쿼리임을 알리는
어노테이션입니다.

JPQL 또는 native SQL의 UPDATE·DELETE·INSERT·DDL을 실행할 때 사용합니다.

특히 벌크 연산은 영속성 컨텍스트를 거치지 않으므로, 성능 이점과 상태 불일치 위험을 함께 이해해야 합니다.

## 목차

- [@Modifying이 필요한 이유](#modifying이-필요한-이유)
- [기본 사용법](#기본-사용법)
- [반환값으로 영향 행 수 확인하기](#반환값으로-영향-행-수-확인하기)
- [트랜잭션 경계](#트랜잭션-경계)
- [영속성 컨텍스트와 벌크 연산](#영속성-컨텍스트와-벌크-연산)
- [flushAutomatically와 clearAutomatically](#flushautomatically와-clearautomatically)
- [파생 삭제와 벌크 삭제의 차이](#파생-삭제와-벌크-삭제의-차이)
- [native SQL과 DDL](#native-sql과-ddl)
- [실무 선택 기준](#실무-선택-기준)
- [참고 자료](#참고-자료)

## @Modifying이 필요한 이유

Spring Data JPA는 `@Query` 메서드를 기본적으로 조회 쿼리로 취급합니다. UPDATE나 DELETE를
`@Query`로 선언했다면 `@Modifying`을 붙여 해당 쿼리를 `executeUpdate()` 방식으로 실행하게
해야 합니다.

```java
public interface ProductRepository extends JpaRepository<Product, Long> {

    @Modifying
    @Query("""
        update Product p
        set p.status = :status
        where p.id = :productId
    """)
    int changeStatus(
        @Param("productId") Long productId,
        @Param("status") ProductStatus status
    );
}
```

`@Modifying`은 JPA 표준 어노테이션이 아니라 Spring Data JPA 기능입니다.

다음처럼 메서드 이름으로 만든 파생 삭제 쿼리나, 직접 `EntityManager`를 사용하는 custom
repository 메서드에는 붙이지 않습니다.

```java
void deleteByStatus(ProductStatus status);
```

## 기본 사용법

변경 쿼리는 서비스 계층의 트랜잭션 안에서 실행합니다. 재고 상태를 변경하는 예시는 다음과 같습니다.

```java
@Service
@RequiredArgsConstructor
public class ProductService {

    private final ProductRepository productRepository;

    @Transactional
    public void discontinue(Long productId) {
        int updated = productRepository.changeStatus(
            productId,
            ProductStatus.DISCONTINUED
        );

        if (updated != 1) {
            throw new IllegalArgumentException("대상 상품이 없거나 수정할 수 없습니다.");
        }
    }
}
```

한 행의 단순 상태 변경에는 엔티티를 조회하고 도메인 메서드로 바꾸는 방식이 더 적합할 수 있습니다. 벌크 UPDATE는
많은 행을 같은 규칙으로 바꾸거나, 조건 자체를 DB에서 원자적으로 평가해야 할 때 유용합니다.

```java
@Modifying
@Query("""
    update Product p
    set p.stock = p.stock - :quantity
    where p.id = :productId
      and p.stock >= :quantity
""")
int decreaseStockIfEnough(
    @Param("productId") Long productId,
    @Param("quantity") int quantity
);
```

이 쿼리는 재고 부족 여부와 차감을 하나의 SQL로 처리합니다. 반환 행 수가 0이면 상품이 없거나 재고가 부족한
경우이므로, 호출자가 이를 구분할 추가 조회가 필요한지 업무 규칙에 따라 결정합니다.

## 반환값으로 영향 행 수 확인하기

`@Modifying` 메서드는 보통 `int` 또는 `void`를 반환합니다. `int`는 DB에서 실제로 영향을 받은
행 수이므로, 낙관적 검증이나 조건부 변경 결과를 확인할 때 사용합니다.

```java
@Modifying
@Query("""
    update Member m
    set m.status = :status
    where m.id = :memberId
      and m.status = :expectedStatus
""")
int changeStatusIfCurrent(
    @Param("memberId") Long memberId,
    @Param("expectedStatus") MemberStatus expectedStatus,
    @Param("status") MemberStatus status
);
```

반환값 1은 조건에 맞는 회원 한 명이 변경됐다는 뜻입니다. 0은 회원이 없거나 현재 상태가 기대값과 다르다는 뜻입니다.
영향 행 수는 DB와 SQL 종류에 따라 해석이 달라질 수 있으므로, 상태 변경의 성공 기준으로 사용할 때는 사용하는
DB에서 확인합니다.

## 트랜잭션 경계

JPQL 벌크 UPDATE·DELETE는 트랜잭션 안에서 실행해야 합니다. 일반적으로 서비스 메서드에
`@Transactional`을 선언해 업무 단위의 모든 변경을 묶습니다.

```java
@Transactional
public void deactivateSeller(Long memberId) {
    memberRepository.changeStatus(memberId, MemberStatus.INACTIVE);
    productRepository.changeStatusByMemberId(
        memberId,
        ProductStatus.HIDDEN
    );
}
```

repository 메서드에 `@Transactional`을 붙일 수도 있지만, 여러 repository 호출의 원자성을
표현하기 어렵습니다. 서비스가 트랜잭션 경계를 갖는 편이 일반적으로 이해하기 쉽습니다.

## 영속성 컨텍스트와 벌크 연산

벌크 연산은 엔티티를 조회해 변경 감지(Dirty Checking)로 UPDATE하는 방식과 다릅니다. DB에 직접
SQL을 실행하므로, 현재 영속성 컨텍스트가 관리하던 엔티티 상태는 자동으로 바뀌지 않습니다.

```java
@Transactional
public void example(Long productId) {
    Product product = productRepository.findById(productId)
        .orElseThrow();

    productRepository.changeStatus(productId, ProductStatus.HIDDEN);

    // DB 값은 HIDDEN이지만, product.getStatus()는 이전 값일 수 있습니다.
    ProductStatus status = product.getStatus();
}
```

이 불일치를 방치하면 이후 로직이 오래된 상태를 판단하거나, flush 시 의도하지 않은 값을 다시 반영할 수 있습니다.
벌크 연산 전후에 같은 엔티티를 계속 사용해야 하는지 먼저 검토합니다.

또한 벌크 UPDATE·DELETE는 개별 엔티티의 변경 감지 흐름과 다릅니다. 엔티티 리스너, `@PreUpdate`,
`@PreRemove`, Auditing, `@Version` 처리에 의존하는 업무라면 실제로 필요한 동작이 실행되는지
확인해야 합니다.

이런 규칙이 중요하고 대상 수가 작다면 엔티티를 조회해 도메인 메서드로 변경하는 편이 안전합니다.

## flushAutomatically와 clearAutomatically

`@Modifying`의 두 속성은 영속성 컨텍스트와 벌크 SQL의 순서를 제어합니다. 기본값은 둘 다
`false`입니다.

```java
@Modifying(flushAutomatically = true, clearAutomatically = true)
@Query("""
    update Product p
    set p.status = :status
    where p.member.id = :memberId
""")
int changeStatusByMemberId(
    @Param("memberId") Long memberId,
    @Param("status") ProductStatus status
);
```

| 속성                 | 실행 시점     | 필요한 경우                                            | 주의할 점                                              |
| -------------------- | ------------- | ------------------------------------------------------ | ------------------------------------------------------ |
| `flushAutomatically` | 벌크 SQL 직전 | 아직 flush되지 않은 변경과 벌크 SQL의 순서가 중요할 때 | 예상보다 많은 INSERT·UPDATE가 먼저 실행될 수 있습니다. |
| `clearAutomatically` | 벌크 SQL 직후 | 같은 트랜잭션에서 영향을 받은 엔티티를 다시 읽을 때    | 관리 중이던 모든 엔티티가 detach됩니다.                |

`clearAutomatically = true`는 오래된 엔티티를 막는 데 도움이 되지만, flush되지 않은 변경도
함께 잃을 수 있습니다. 그래서 변경이 남아 있을 수 있는 흐름에서는 `flushAutomatically = true`를
함께 검토합니다.

두 속성을 습관적으로 항상 켜기보다, 벌크 연산 전후에 관리 중인 엔티티가 있는지 기준으로 결정합니다.

## 파생 삭제와 벌크 삭제의 차이

아래 두 메서드는 결과적으로 같은 조건의 데이터를 삭제할 수 있지만 실행 방식이 다릅니다.

```java
void deleteByStatus(ProductStatus status);

@Modifying
@Query("delete from Product p where p.status = :status")
int deleteInBulkByStatus(@Param("status") ProductStatus status);
```

파생 삭제는 먼저 엔티티를 조회한 다음 엔티티별 삭제를 수행합니다. 따라서 JPA 생명주기 콜백이 실행될 수 있지만, 대상
엔티티를 메모리에 올리므로 대량 삭제에는 부담이 될 수 있습니다.

벌크 삭제는 하나의 DELETE SQL로 처리하므로 대량 데이터에 유리합니다. 대신 엔티티 단위의 `@PreRemove`와
cascade 동작을 기대하면 안 됩니다. 외래 키 제약, DB cascade, 삭제 이력 보관 정책까지 함께 설계해야
합니다.

## native SQL과 DDL

DB 전용 문법이 필요하면 native SQL에도 `@Modifying`을 사용할 수 있습니다.

```java
@Modifying
@Query(
    value = "update products set status = :status where member_id = :memberId",
    nativeQuery = true
)
int changeStatusByMemberIdNative(
    @Param("memberId") Long memberId,
    @Param("status") String status
);
```

native SQL은 엔티티명과 필드명이 아니라 실제 테이블명과 컬럼명을 사용합니다. DB 방언, enum 저장 방식,
예약어, 결과 매핑이 달라질 수 있어 JPQL로 충분한 경우에는 JPQL을 우선 검토합니다.

`@Modifying`은 DDL에도 사용할 수 있지만, 운영 스키마 변경은 repository 메서드로 실행하지 않습니다.

DDL은 트랜잭션 지원과 잠금 범위가 DB마다 다르며, Flyway 같은 형상 관리 도구로 버전·검증·복구 절차와 함께
관리합니다.

## 실무 선택 기준

- 한 건 변경에서 도메인 검증, 이벤트, 감사 필드가 중요하면 엔티티를 조회해 변경합니다.
- 같은 조건으로 많은 행을 바꾸거나 재고처럼 조건부 변경이 필요하면 벌크 연산을 검토합니다.
- `@Modifying`은 `@Query` 기반의 변경 쿼리에만 사용합니다.
- 서비스 계층에서 `@Transactional`로 업무 트랜잭션 경계를 설정합니다.
- `int` 반환값을 확인해 조건부 변경이 실제로 성공했는지 판단합니다.
- 벌크 연산 뒤 같은 엔티티를 사용할 수 있다면 flush·clear 전략을 명시합니다.
- 대량 DELETE 전에 FK 제약, DB cascade, 데이터 보관·복구 요구 사항을 확인합니다.

## 참고 자료

- [Spring Data JPA Modifying Queries](https://docs.spring.io/spring-data/jpa/reference/jpa/query-methods.html)
- [@Modifying API](https://docs.spring.io/spring-data/jpa/docs/current/api/org/springframework/data/jpa/repository/Modifying.html)
