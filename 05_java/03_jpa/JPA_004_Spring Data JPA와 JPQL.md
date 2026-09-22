# Spring Data JPA와 JPQL

Spring Data JPA와 JPQL은 서로 대체하는 기술이 아니라 역할이 다릅니다.
Spring Data JPA는 JPA를 편리하게 사용하도록 도와주는 도구이고, JPQL은 엔티티를
조회하는 객체 지향 쿼리 언어입니다.

## 목차

- [Spring Data JPA란 무엇인가요](#spring-data-jpa란-무엇인가요)
- [JPQL이란 무엇인가요](#jpql이란-무엇인가요)
- [두 기술은 어떻게 함께 사용하나요](#두-기술은-어떻게-함께-사용하나요)
- [Spring Data JPA와 JPQL 비교](#spring-data-jpa와-jpql-비교)
- [JPQL과 SQL의 차이](#jpql과-sql의-차이)
- [실무에서 선택하는 기준](#실무에서-선택하는-기준)

## Spring Data JPA란 무엇인가요

Spring Data JPA는 JPA를 기반으로 repository를 쉽게 만들도록 도와주는 Spring
프로젝트입니다. 기본 CRUD와 조회 메서드 구현을 직접 작성하지 않아도 됩니다.

```java
public interface MemberRepository
        extends JpaRepository<Member, Long> {
}
```

`JpaRepository`를 상속하면 `save()`, `findById()`, `findAll()`, `deleteById()`
같은 기본 메서드를 사용할 수 있습니다. 실제 SQL을 작성하지 않아도 JPA 구현체인
Hibernate가 데이터베이스에 맞는 SQL을 생성합니다.

간단한 조건은 메서드 이름으로 표현할 수 있습니다.

```java
List<Member> findByName(String name);

List<Member> findByAgeGreaterThan(int age);
```

Spring Data JPA는 메서드 이름을 해석해 조회 쿼리를 만들지만, 메서드 이름이 너무
길어지면 읽기 어렵습니다. 조건이 복잡해질 때는 JPQL이나 DTO 조회를 고려합니다.

## JPQL이란 무엇인가요

JPQL은 테이블과 컬럼이 아니라 엔티티와 엔티티 필드를 대상으로 작성하는 쿼리
언어입니다. SQL과 비슷해 보이지만 조회 대상이 데이터베이스 테이블이 아니라
영속성 컨텍스트가 관리하는 엔티티라는 점이 다릅니다.

```java
@Query("""
    select m
    from Member m
    where m.name = :name
""")
List<Member> findMembers(@Param("name") String name);
```

여기서 `Member`는 테이블 이름이 아니라 엔티티 이름입니다. `m.name`도 데이터베이스
컬럼명이 아니라 `Member` Java 클래스의 필드입니다. JPQL은 JPA 구현체가 데이터베이스
SQL로 변환한 뒤 실행합니다.

JPQL은 연관관계를 이용한 조회에도 사용합니다.

```java
@Query("""
    select distinct m
    from Member m
    left join fetch m.orders
    where m.id = :memberId
""")
Optional<Member> findMemberWithOrders(
    @Param("memberId") Long memberId
);
```

`left join fetch`는 회원과 주문을 이번 조회에서 함께 가져오도록 요청합니다.
이처럼 JPQL은 단순 조건을 넘어 fetch join, 집계, 여러 조건의 조합을 표현할 때
유용합니다.

## 두 기술은 어떻게 함께 사용하나요

Spring Data JPA repository의 `@Query` 안에 JPQL을 작성하는 방식이 실무에서 자주
사용됩니다.

```java
public interface MemberRepository
        extends JpaRepository<Member, Long> {

    @Query("""
        select m
        from Member m
        where m.name = :name
    """)
    List<Member> findMembers(@Param("name") String name);
}
```

호출하는 서비스 코드는 repository 메서드만 사용합니다.

```java
List<Member> members = memberRepository.findMembers("홍길동");
```

즉, Spring Data JPA는 repository와 실행 흐름을 제공하고, JPQL은 복잡한 조회
조건을 표현합니다. 두 기술은 함께 사용할 수 있지만 같은 개념은 아닙니다.

## Spring Data JPA와 JPQL 비교

| 구분   | Spring Data JPA                            | JPQL                                       |
| ------ | ------------------------------------------ | ------------------------------------------ |
| 정체   | Spring 데이터 접근 도구입니다.             | 엔티티 조회 언어입니다.                    |
| 역할   | repository와 기본 CRUD를 제공합니다.       | 복잡한 조회 조건을 작성합니다.             |
| 사용법 | `JpaRepository`, 메서드 이름을 사용합니다. | `@Query` 안에 작성합니다.                  |
| 대상   | JPA repository를 통한 엔티티입니다.        | 엔티티와 엔티티 필드입니다.                |
| 관계   | JPQL을 실행할 수 있습니다.                 | Spring Data JPA 안에서 사용할 수 있습니다. |

## JPQL과 SQL의 차이

JPQL은 엔티티와 필드를 사용하고, SQL은 테이블과 컬럼을 사용합니다.

```java
@Query("select m from Member m where m.name = :name")
List<Member> findMembers(@Param("name") String name);
```

```sql
select *
from members
where member_name = ?;
```

JPQL의 `Member`와 `name`은 Java 모델에 맞춘 이름이고, SQL의 `members`와
`member_name`은 데이터베이스 스키마에 맞춘 이름입니다. 실제 SQL은 엔티티 매핑,
네이밍 전략, 데이터베이스 종류에 따라 달라질 수 있습니다.

데이터베이스 기능을 직접 사용해야 하거나 JPQL로 표현하기 어려운 쿼리는 native
query를 선택할 수 있습니다.

```java
@Query(
    value = "select * from members where member_name = :name",
    nativeQuery = true
)
List<Member> findByNameNative(@Param("name") String name);
```

native query는 데이터베이스 문법과 스키마에 강하게 결합됩니다. 따라서 일반적인
엔티티 조회는 JPQL을 우선 사용하고, 데이터베이스 전용 기능이나 성능상 필요한 경우에
한해 native query를 선택하는 편이 유지보수에 유리합니다.

## 실무에서 선택하는 기준

- 기본 CRUD와 단순 조건은 `JpaRepository`와 메서드 이름을 사용합니다.
- 메서드 이름이 지나치게 길어지거나 조건이 복잡하면 `@Query`와 JPQL을 사용합니다.
- fetch join, 집계, 여러 엔티티 조합이 필요하면 JPQL을 먼저 검토합니다.
- 조회 결과가 일부 필드뿐이면 엔티티 대신 DTO 조회를 고려합니다.
- 실행 SQL과 조회 횟수를 확인해 N+1이나 불필요한 컬럼 조회를 점검합니다.
- 데이터베이스 전용 기능이 필요할 때만 native query를 제한적으로 사용합니다.

Spring Data JPA를 사용한다고 해서 JPQL이나 SQL을 몰라도 되는 것은 아닙니다. 조회가
복잡해질수록 JPQL의 의미와 실제 생성 SQL을 함께 이해해야 안정적으로 최적화할 수
있습니다.
