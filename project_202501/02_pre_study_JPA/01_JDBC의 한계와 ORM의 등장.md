# 🚀 JDBC의 한계와 ORM의 등장(ORM/JPA/Hibernate)

> 객체지향 프로그래밍과 관계형 데이터베이스 사이에는 근본적인 불일치가 존재한다. 이 간극을 매우는 것이 ORM 의 핵심 목표이다.

- [1.] 전통적 방식의 문제점
- [2.] ORM 이란
- [3.] JPA, Hibernate, Spring Data JPA 의 관계
- [4.] Spring Data JPA 는 어떻게 동작하는가

## [1.] 전통적 방식의 문제점

- 반복적인 보일러플레이트
  - connection 획득, Statement 생성, ResultSet 매핑, 자원 해제 코드가 모든 쿼리마다 반복
- SQL 의존성
  - 비지니스 로직이 SQL 문자열에 묻혀버림
  - DB 변경시 전체 코드 수정 필요
- 수동 매핑의 위험성
  - ResultSet 에서 객체로의 변환을 개발자가 직접 작성
  - 컬럼명 오타 시 런타임 에러 발생
- 연관관계 처리의 복잡성
  - N+1 문제를 개발자가 직접 인지하고 해결해야 함
  - 두 테이블간의 관계를 표현하려면 추가 쿼리와 복잡한 조인 로직 필요

## [2.] ORM 이란

- ORM은,
  - 객체지향 프로그래밍 언어와 관계형 데이터베이스 사이의 데이터를 변환하는 프로그래밍 기법
  - 이를 통해 개발자는 SQL 대신 객체를 사용하여 데이터베이스를 조작
  - 이를 통해 객체-관계 불일치 문제를 해결

```shell
Java 객체 Member
↔
ORM 자동 변환
↔
DB 테이블 member
```

### ORM 의 장단점

👏 장점

- 생산성 향상: 반복적인 CRUD 코드 90% 이상 감소
- 유지보수성: 스키마 변경 시 Entity만 수정하면 됨
- 객체지향 설계: 도메인 모델에 집중할 수 있음
- DB 독립성: 방언(Dialect)으로 DB 변경 용이
- 캐싱: 1차 캐시, 2차 캐시로 성능 최적화
- 지연 로딩: 필요한 시점에 데이터 로딩

👀 주의점 (단점이 아닌 주의점)

- 학습 곡선: 영속성 컨텍스트, 지연 로딩 등 개념 이해 필요
- 복잡한 쿼리: 통계, 리포트 등은 Native SQL 필요할 수 있음
- N+1 문제: 잘못 사용하면 성능 이슈 발생 (해결 방법 있음)
- 내부 동작 이해: 블랙박스로 사용하면 문제 해결 어려움
- 대량 데이터: 배치 처리 시 별도 최적화 필요

### ORM을 사용해야 하는 경우

적합한 경우

- 도메인 모델 중심의 애플리케이션
- CRUD 위주의 비지니스 로직
- 객체지향 설계를 중시하는 프로젝트
- 팀의 생산성이 중요한 경우
- DB 변경 가능성이 있는 경우

신중해야 하는 경우

- 복잡한 통계/분석 쿼리가 많은 경우
- 초당 수만 건 이상의 대용량 처리
- 레거시 DB 스키마가 매우 복잡한 경우
- SQL 튜닝이 핵심인 프로젝트

## [3] JPA, Hibernate, Spring Data JPA 의 관계

### JPA(Java Persistence API)

- JPA는 자바 진영의 ORM(Object-Relational Mapping)
- 👉 자바 객체를 관계형 데이터베이스 테이블에 어떻게 매핑하고, 어떻게 저장/조회/수정/삭제 에 대한
  규칙(인터페이스) 정의
- JPA = 규약(스펙), 구현체는 따로 있음
- 📌 Spring Boot에서 JPA를 쓴다고 하면 실제로는 보통 ➡ Hibernate ORM 가 동작한다.
- 핵심 개념
  - Entity
  - Entity Manager
  - Persistence Context(영속성 컨텍스트)

### JPA, Hibernate, Spring Data JPA 의 관계

#### Spring Data JPA

- Spring 프레임워크에서 제공하는 JPA 편의 기능
- Repository 인터페이스만 정의하면 구현체를 자동 생성
- 메서드 이름으로 쿼리를 자동 생성하는 강력한 기능 제공

```java
// 인터페이스만 정의하면 구현체 자동 생성!
public interface ProductRepository extends JpaRepository<Product, Long> {
    List<Product> findByNameContaining(String keyword);
    List<Product> findByPriceGreaterThan(BigDecimal price);
    Optional<Product> findBySku(String sku);
}
```

#### Hibernate

- JPA를 구현한 대표적인 구현체
- 가장 널리 사용되는 JPA 구현체
- EclipseLink, OpenJPA 등 다른 구현체도 있지만, Hibernate가 사실상 표준
- Hibernate 고유 기능
  - @BatchSize, @Fetch, @Formula 등

```java
Session session = sessionFactory.openSession();
session.createQuery("FROM Product p WHERE p.name LIKE :name", Product.class)
       .setParameter("name", "%" + keyword + "%")
       .list();
```

#### JPA

- 자바 ORM 기술의 표준 명세(인터페이스)
- 구현체가 아닌 스펙
- javax.persistence (Java EE) 또는 jakarta.persistence (Jakarta EE) 패키지에 정의

```java
// JPA 표준 API
EntityManager em = entityManagerFactory.createEntityManager();
em.persist(product);
em.find(Product.class, productId);
em.createQuery("SELECT p FROM Product p", Product.class).getResultList();

// JPA 표준 어노테이션
@Entity, @Table, @Id, @Column, @ManyToOne, @OneToMany ...
```

#### JDBC(Java Database Connectivity)

- 모든 Java DB 접근의 기반
- Hibernate도 내부적으로 JDBC를 사용하여 실제 데이터베이스와 통신
- 최종적으로 SQL이 실행되는 계층

```shell
- Spring Data JPA 는 내부적으로 Hibernate 사용
- Hibernate 를 구현한 것이 JPA(Java Persistence API)

JDBC(Java Database Connectivity)는
- 모든 Java DB 접근의 기반
- Hibernate도 내부적으로 JDBC를 사용하여 실제 데이터베이스와 통신
```

## 예시

### Spring Data JPA 예시

```java
// 인터페이스만 정의하면 구현체 자동 생성!
public interface ProductRepository extends JpaRepository<Product, Long> {
    List<Product> findByNameContaining(String keyword);
    List<Product> findByPriceGreaterThan(BigDecimal price);
    Optional<Product> findBySku(String sku);
}
```

### Hibernate 예시

```java
// Hibernate 고유 API (JPA 표준 아님)
Session session = sessionFactory.openSession();
session.createQuery("FROM Product p WHERE p.name LIKE :name", Product.class)
       .setParameter("name", "%" + keyword + "%")
       .list();

// Hibernate 고유 기능: @BatchSize, @Fetch, @Formula 등
```

### JPA (Java Persistence API)

```java
// JPA 표준 API
EntityManager em = entityManagerFactory.createEntityManager();
em.persist(product);
em.find(Product.class, productId);
em.createQuery("SELECT p FROM Product p", Product.class).getResultList();

// JPA 표준 어노테이션
@Entity, @Table, @Id, @Column, @ManyToOne, @OneToMany ...
```

## Spring Boot 에서의 설정

build.gradle 의존성

```java
dependencies {
    // Spring Data JPA (JPA + Hibernate 포함)
    implementation 'org.springframework.boot:spring-boot-starter-data-jpa'

    // 데이터베이스 드라이버
    runtimeOnly 'com.mysql:mysql-connector-j'      // MySQL
    runtimeOnly 'org.postgresql:postgresql'         // PostgreSQL
    runtimeOnly 'com.h2database:h2'                 // H2 (개발/테스트용)
}

// spring-boot-starter-data-jpa가 포함하는 것:
// - spring-data-jpa (Spring Data JPA)
// - hibernate-core (Hibernate)
// - jakarta.persistence-api (JPA 스펙)
// - spring-jdbc, spring-tx (트랜잭션)
// - HikariCP (커넥션 풀)
```

## [4.] Spring Data JPA 는 어떻게 동작하는가

### 프록시(Proxy)란?

- 프록시(Proxy)는 "대리인"이라는 뜻
- 실제 객체를 직접 호출하는 대신, 중간에서 대신 처리해주는 객체를 말함

일상 속 프록시

- 부동산 중개인 = 프록시
- 집주인(실제 객체)을 직접 만나지 않고, 중개인(프록시)이 대신 계약을 처리

프로그래밍 속 프록시

- MemberRepository 인터페이스를 호출하면, 실제로는 Spring 이 만든 프록시 객체가 요청을 받아 처리
