# Spring Data JPA와 JPQL의 차이

Spring Data JPA와 JPQL은 같은 종류의 기술이 아닙니다. Spring Data JPA는 repository
구현을 간소화하는 Spring 모듈이고, JPQL은 엔티티를 대상으로 작성하는 JPA 쿼리 언어입니다. Spring
Data JPA repository 안에서 JPQL을 함께 사용할 수 있습니다.

## 목차

- [먼저 구분하기](#먼저-구분하기)
- [예제 엔티티](#예제-엔티티)
- [순수 JPA에서 JPQL 실행하기](#순수-jpa에서-jpql-실행하기)
- [Spring Data JPA의 쿼리 메서드](#spring-data-jpa의-쿼리-메서드)
- [Spring Data JPA에서 JPQL 사용하기](#spring-data-jpa에서-jpql-사용하기)
- [JPQL과 SQL의 차이](#jpql과-sql의-차이)
- [실무 선택 기준](#실무-선택-기준)

## 먼저 구분하기

| 구분        | Spring Data JPA                             | JPQL                                        |
| ----------- | ------------------------------------------- | ------------------------------------------- |
| 정체        | JPA 데이터 접근을 편하게 만드는 Spring 모듈 | JPA 표준의 객체 지향 쿼리 언어              |
| 주 역할     | repository 구현, 기본 CRUD, 페이징          | 조건, JOIN, 집계가 있는 조회·수정 쿼리 표현 |
| 대표 사용법 | `JpaRepository`, 쿼리 메서드                | `EntityManager.createQuery()`, `@Query`     |
| 대상 이름   | repository와 엔티티                         | 엔티티명과 Java 필드명                      |
| 관계        | JPQL을 실행할 수 있습니다.                  | Spring Data JPA에서 사용할 수 있습니다.     |

따라서 "Spring Data JPA를 쓸지 JPQL을 쓸지"처럼 둘 중 하나를 고르는 문제는 아닙니다. 간단한 조회에는
Spring Data JPA 쿼리 메서드를 쓰고, 쿼리 의도가 더 복잡해지면 같은 repository 안에 JPQL을
작성하는 흐름이 일반적입니다.

## 예제 엔티티

`Product`는 실제 테이블 이름과 다를 수 있습니다. JPQL에서는 테이블명 `products`가 아닌 엔티티명
`Product`와 필드명 `price`를 사용합니다.

```java
@Entity
@Table(name = "products")
public class Product {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @Column(nullable = false)
    private String name;

    @Column(nullable = false)
    private int price;
}
```

## 순수 JPA에서 JPQL 실행하기

순수 JPA는 `EntityManager`로 영속성 컨텍스트를 다루고 JPQL을 실행합니다.

```java
@Repository
@RequiredArgsConstructor
public class ProductJpaRepository {

    private final EntityManager entityManager;

    public List<Product> findExpensiveProducts(int minPrice) {
        return entityManager.createQuery("""
            select p
            from Product p
            where p.price >= :minPrice
            order by p.price desc
        """, Product.class)
            .setParameter("minPrice", minPrice)
            .getResultList();
    }
}
```

다음 부분이 JPQL입니다.

```sql
select p
from Product p
where p.price >= :minPrice
order by p.price desc
```

이 방식은 JPA의 동작을 직접 제어해야 하거나 Spring Data JPA를 사용하지 않는 프로젝트에 적합합니다. 반면
단순 CRUD까지 직접 구현하면 반복 코드가 많아집니다.

## Spring Data JPA의 쿼리 메서드

Spring Data JPA는 메서드 이름을 해석해 간단한 조회 쿼리를 만듭니다.

```java
public interface ProductRepository extends JpaRepository<Product, Long> {

    List<Product> findByPriceGreaterThanEqualOrderByPriceDesc(int minPrice);
}
```

이 메서드는 개념적으로 앞의 JPQL과 같은 조건을 표현합니다. ID 조회, 이메일 중복 검사, 상태별 목록처럼 조건이
짧고 자주 쓰이는 경우에 적합합니다.

```java
Optional<Product> findById(Long id);

boolean existsByName(String name);

List<Product> findByPriceBetween(int minPrice, int maxPrice);
```

조건이 늘어난다고 쿼리 메서드 이름을 계속 늘리면, 조회 목적을 파악하기 어려워집니다.

```java
Page<Product> findByNameContainingAndPriceGreaterThanEqualAndCreatedAtBetween(
    String name,
    int minPrice,
    LocalDateTime from,
    LocalDateTime to,
    Pageable pageable
);
```

이 정도부터는 JPQL, QueryDSL, 별도 조회 repository를 검토합니다.

## Spring Data JPA에서 JPQL 사용하기

Spring Data JPA의 `@Query` 안에 JPQL을 넣으면, repository의 편의 기능과 명시적인 쿼리를
함께 사용할 수 있습니다.

```java
public interface ProductRepository extends JpaRepository<Product, Long> {

    @Query("""
        select p
        from Product p
        where p.price >= :minPrice
        order by p.price desc
    """)
    List<Product> findExpensiveProducts(@Param("minPrice") int minPrice);
}
```

상품과 등록 회원을 이번 조회에서 함께 사용해야 한다면 fetch join도 JPQL로 표현할 수 있습니다.

```java
@Query("""
    select p
    from Product p
    join fetch p.member
    where p.id = :productId
""")
Optional<Product> findWithMemberById(@Param("productId") Long productId);
```

fetch join은 상세 화면처럼 `Product`와 `Member`가 모두 필요한 조회에 적합합니다. 모든 기본 조회에
적용하면 필요하지 않은 연관 데이터까지 읽게 됩니다. 컬렉션 fetch join과 페이징의 조합도 주의해야 합니다.

## JPQL과 SQL의 차이

JPQL은 Java 모델을 기준으로, SQL은 실제 DB 스키마를 기준으로 작성합니다.

```sql
select p
from Product p
where p.price >= :minPrice
```

```sql
select p.id, p.name, p.price
from products p
where p.price >= ?
```

JPQL은 `Product` 엔티티와 `price` 필드를 사용합니다. Hibernate 같은 JPA 구현체가 매핑 정보와
DB dialect를 사용해 SQL로 변환합니다. 실제 SQL의 테이블명, 컬럼명, 별칭은 `@Table`,
`@Column`, 네이밍 전략과 DB에 따라 달라집니다.

윈도우 함수, DB 전용 JSON 함수처럼 JPQL로 표현하기 어려운 기능이 필요할 때는 native query를 사용할
수 있습니다. 이 경우에는 테이블·컬럼명을 쓰며 DB 의존성, 매핑 결과, 페이징 count 쿼리를 함께 검증해야 합니다.

## 실무 선택 기준

- 기본 CRUD와 짧은 단일 조건은 `JpaRepository`와 쿼리 메서드를 사용합니다.
- 고정된 JOIN, fetch join, DTO 생성자 표현식, 집계는 `@Query`의 JPQL을 검토합니다.
- 선택 조건의 조합이 많으면 QueryDSL 또는 Specification을 사용합니다.
- DB 전용 기능이 꼭 필요할 때만 native query를 사용하고, 생성된 SQL을 확인합니다.
- 목록 API에는 엔티티 전체보다 필요한 필드만 담은 DTO 조회를 우선 검토합니다.

Spring Data JPA를 사용하더라도 JPQL과 실제 실행 SQL을 이해해야 N+1, 불필요한 JOIN, 잘못된
count 쿼리 같은 문제를 분석할 수 있습니다.

## 참고 자료

- [Spring Data JPA Reference](https://docs.spring.io/spring-data/jpa/reference/)
- [Jakarta Persistence Specification](https://jakarta.ee/specifications/persistence/)
