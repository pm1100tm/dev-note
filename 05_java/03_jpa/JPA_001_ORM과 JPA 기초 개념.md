# ORM과 JPA 기초 개념

JPA를 사용하면 SQL과 객체 변환 코드를 줄이고, 객체 중심으로 데이터 접근 코드를 작성할 수 있습니다. 그러나
JPA는 SQL을 없애는 기술이 아닙니다. 객체와 관계형 데이터베이스 사이의 차이를 관리하고 SQL 실행을 대신하는
ORM 표준입니다.

## 목차

- [ORM이 필요한 이유](#orm이-필요한-이유)
- [JPA, Hibernate, Spring Data JPA의 관계](#jpa-hibernate-spring-data-jpa의-관계)
- [첫 번째 Entity 정의](#첫-번째-entity-정의)
- [영속성 컨텍스트](#영속성-컨텍스트)
- [Entity의 네 가지 생명주기](#entity의-네-가지-생명주기)
- [변경 감지](#변경-감지)
- [JPA를 사용해도 SQL을 알아야 합니다](#jpa를-사용해도-sql을-알아야-합니다)
- [ORM 사용의 판단 근거](#orm-사용의-판단-근거)

## ORM이 필요한 이유

Java 객체와 관계형 데이터베이스는 데이터를 표현하는 방식이 다릅니다.

| 객체 모델                              | 관계형 데이터베이스                     |
| -------------------------------------- | --------------------------------------- |
| 객체 참조로 관계를 표현합니다.         | 외래 키와 JOIN으로 관계를 표현합니다.   |
| 상속과 다형성을 지원합니다.            | 테이블에는 객체 상속 개념이 없습니다.   |
| 객체 동일성과 값 동등성을 구분합니다.  | 기본 키로 행을 식별합니다.              |
| 컬렉션과 값 객체를 사용할 수 있습니다. | 컬렉션은 보통 별도 테이블로 표현합니다. |

이 차이를 객체-관계 불일치(Object-Relational Impedance Mismatch)라고 합니다. ORM(Object-Relational
Mapping)은 객체와 테이블의 대응 관계를 정의하고, 객체의 상태를 SQL과 데이터베이스 행으로 변환합니다.

### JDBC만 사용할 때 반복되는 작업

JDBC는 Java에서 데이터베이스에 접근하는 기반 기술입니다. 직접 사용하면 연결 획득, SQL 작성, 파라미터 설정,
결과 변환, 자원 해제를 쿼리마다 처리해야 합니다.

```java
String sql = "SELECT id, name FROM member WHERE id = ?";

try (Connection connection = dataSource.getConnection();
     PreparedStatement statement = connection.prepareStatement(sql)) {
    statement.setLong(1, memberId);

    try (ResultSet resultSet = statement.executeQuery()) {
        if (resultSet.next()) {
            Member member = new Member(
                resultSet.getLong("id"),
                resultSet.getString("name")
            );
        }
    }
}
```

이 코드는 JDBC 사용법으로는 잘못되지 않았습니다. 다만 엔티티와 컬럼이 늘어날수록 비슷한 코드가 반복되고,
컬럼명 오타나 변환 누락은 컴파일 시점에 발견하기 어렵습니다. 연관된 데이터를 객체 그래프로 만들려면 JOIN과
추가 변환 코드도 직접 관리해야 합니다.

JPA를 사용하면 매핑 정보를 바탕으로 SQL 생성과 조회 결과 변환을 위임할 수 있습니다.

```java
Member member = entityManager.find(Member.class, memberId);
```

JPA도 내부에서는 JDBC를 사용합니다. JDBC 자체를 대체한다기보다, 반복되는 데이터 접근 작업 위에 객체 중심의
추상화를 제공합니다. 복잡한 통계 조회나 대량 처리처럼 SQL 특성이 중요한 작업은 JPQL, 네이티브 SQL 또는 다른
데이터 접근 기술이 더 적합할 수 있습니다.

## JPA, Hibernate, Spring Data JPA의 관계

세 기술은 경쟁 관계가 아니라 서로 다른 계층의 역할을 담당합니다.

| 구분            | 역할                                     | 대표 API                   |
| --------------- | ---------------------------------------- | -------------------------- |
| JPA             | Java ORM의 표준 명세입니다.              | `EntityManager`, `@Entity` |
| Hibernate       | JPA 명세를 구현한 ORM 구현체입니다.      | `Session`, `@BatchSize`    |
| Spring Data JPA | JPA 사용을 단순화하는 Spring 모듈입니다. | `JpaRepository`            |

Spring Boot 3 계열에서는 JPA 표준 API가 `jakarta.persistence` 패키지에 있습니다. 일반적인 Spring Data
JPA 프로젝트도 최종적으로 `EntityManager`를 사용하며, 기본 구현체로 Hibernate를 사용하는 경우가 많습니다.

```java
public interface MemberRepository extends JpaRepository<Member, Long> {
    Optional<Member> findByName(String name);
}
```

Spring Data JPA는 Repository 구현체와 기본 CRUD 코드를 생성해 줍니다. 하지만 엔티티 상태, 트랜잭션,
flush 같은 JPA 규칙까지 없애 주는 것은 아닙니다.

## 첫 번째 Entity 정의

Entity는 데이터베이스 테이블과 매핑되며, 영속성 컨텍스트가 생명주기와 변경을 관리하는 객체입니다. 단순히
데이터를 전달하는 DTO와 역할이 다릅니다.

```java
import jakarta.persistence.Entity;
import jakarta.persistence.GeneratedValue;
import jakarta.persistence.GenerationType;
import jakarta.persistence.Id;

@Entity
public class Member {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    private String name;

    protected Member() {
    }

    public Member(String name) {
        this.name = name;
    }

    public void changeName(String name) {
        this.name = name;
    }
}
```

- `@Entity`는 이 클래스를 JPA가 관리할 엔티티로 등록합니다.
- `@Id`는 엔티티를 식별하는 기본 키를 지정합니다.
- `@GeneratedValue`는 기본 키 생성을 영속성 제공자에게 위임합니다.
- 기본 생성자는 JPA 구현체가 엔티티를 생성할 때 필요합니다.

`IDENTITY` 전략은 데이터베이스의 자동 증가 컬럼을 이용합니다. 데이터베이스에서 키를 받아야 하므로
Hibernate가 `persist()`시점에 `INSERT`를 실행할 수 있습니다. 키 생성 전략별 동작은 기본
Entity Mapping에서 자세히 다룹니다.

## 영속성 컨텍스트

영속성 컨텍스트(Persistence Context)는 엔티티 인스턴스를 관리하는 논리적 공간입니다.
애플리케이션은 `EntityManager`를 통해 접근합니다.

영속성 컨텍스트는 다음 기능의 기반이 됩니다.

- 같은 컨텍스트 안에서 기본 키가 같은 엔티티의 동일성을 보장합니다.
- 조회한 엔티티를 1차 캐시에 보관합니다.
- 엔티티의 변경을 감지해 SQL로 동기화합니다.
- 쓰기 작업을 flush 시점까지 모아 둘 수 있습니다.

1차 캐시는 애플리케이션 전체에서 공유하는 캐시가 아닙니다. 해당 영속성 컨텍스트 안에서만 유효합니다.

### persist 호출과 SQL 실행은 같은 의미가 아닙니다

```java
EntityTransaction transaction = entityManager.getTransaction();
transaction.begin();

Member member = new Member("민수"); // 비영속
entityManager.persist(member);     // 영속

transaction.commit();              // flush 후 트랜잭션 커밋
```

`persist()`는 새 엔티티를 영속 상태로 만드는 JPA 생명주기 연산입니다. 표준상 `INSERT`는 flush 시점이나
그 이전에 실행될 수 있습니다. 따라서 다음처럼 `persist()`가 항상 SQL 실행을 커밋까지 미룬다고 가정하면 잘못입니다.

```java
entityManager.persist(member);
// 잘못된 가정: 이 줄에서는 어떤 INSERT도 절대 실행되지 않습니다.
```

실제 SQL 시점은 식별자 생성 전략, flush 모드, 실행할 쿼리와 구현체 최적화에 따라 달라집니다.
특히 `IDENTITY` 전략은 키를 얻기 위해 INSERT가 일찍 실행될 수 있습니다.

`flush()`는 영속성 컨텍스트의 변경 내용을 데이터베이스 SQL로 동기화합니다.
트랜잭션을 확정하는 `commit()`과는 다르므로, flush후에도 트랜잭션을 rollback하면 변경을 취소할 수 있습니다.

## Entity의 네 가지 생명주기

엔티티 상태는 특정 영속성 컨텍스트와의 관계를 기준으로 구분합니다.

| 상태             | 의미                                         | 대표 전환             |
| ---------------- | -------------------------------------------- | --------------------- |
| 비영속(New)      | 새로 만들었지만 아직 관리되지 않습니다.      | `new Member()`        |
| 영속(Managed)    | 영속성 컨텍스트가 관리합니다.                | `persist()`, `find()` |
| 준영속(Detached) | 관리되던 엔티티가 컨텍스트에서 분리됐습니다. | `detach()`, `clear()` |
| 삭제(Removed)    | 삭제 대상으로 표시된 관리 상태입니다.        | `remove()`            |

```java
Member member = new Member("민수"); // 비영속

entityManager.persist(member);      // 영속
entityManager.detach(member);       // 준영속

Member managedMember = entityManager.find(Member.class, 1L);
entityManager.remove(managedMember); // 삭제
```

`remove()`를 호출했다고 데이터베이스 행이 반드시 즉시 삭제되는 것은 아닙니다. 삭제 SQL은 flush 과정에서
데이터베이스에 전달됩니다.

## 변경 감지

변경 감지(Dirty Checking)는 영속 상태 엔티티의 값을 처음 관리할 때의 상태와 비교하고, 달라진 내용을
flush 과정에서 `UPDATE` SQL로 만드는 기능입니다.

### 잘못된 예: 트랜잭션 밖에서 변경하기

```java
public void changeMemberName(Long memberId, String newName) {
    Member member = memberRepository.findById(memberId)
        .orElseThrow();

    member.changeName(newName);
}
```

Spring Data JPA의 조회가 끝난 뒤 엔티티가 이미 준영속 상태가 되었다면 변경 감지가 동작하지 않습니다.
메모리의 객체 값만 바뀌고 데이터베이스에는 반영되지 않을 수 있습니다.

### 올바른 예: 변경 작업을 트랜잭션 안에서 수행하기

```java
@Transactional
public void changeMemberName(Long memberId, String newName) {
    Member member = memberRepository.findById(memberId)
        .orElseThrow();

    member.changeName(newName);
}
```

이 예제에서는 조회한 `Member`가 트랜잭션에 참여한 영속성 컨텍스트에서 관리됩니다. 메서드가 정상 종료되면
일반적으로 커밋 전에 flush가 수행되고, 변경된 이름을 반영하는 `UPDATE`가 실행됩니다.
이미 영속 상태이므로 다시 `save()`할 필요가 없습니다.

변경 감지가 동작하려면 다음 조건을 확인해야 합니다.

- 엔티티가 현재 영속성 컨텍스트에 연결된 영속 상태여야 합니다.
- 변경 내용이 flush되어야 합니다.
- 데이터베이스 반영을 확정하려면 트랜잭션이 정상적으로 커밋되어야 합니다.
- 읽기 전용 트랜잭션이나 수동 flush 설정은 동작에 영향을 줄 수 있습니다.

Spring Data JPA의 `save()`는 새 엔티티에는 일반적으로 `persist()`를, 기존 엔티티에는 `merge()`를 사용합니다.
`save()`가 모든 변경을 데이터베이스에 즉시 확정하는 명령은 아닙니다. 반대로 영속 상태의 엔티티는 `save()`를
다시 호출하지 않아도 변경 감지의 대상이 됩니다.

## JPA를 사용해도 SQL을 알아야 합니다

JPA가 생성하는 SQL은 애플리케이션 성능과 데이터 정합성에 직접 영향을 줍니다. 다음 항목은 반드시 확인해야 합니다.

- 예상한 `SELECT`, `INSERT`, `UPDATE`, `DELETE`가 실행되는지 확인합니다.
- 조회 횟수와 JOIN 형태를 확인해 N+1 문제를 찾습니다.
- 조회 조건과 정렬에 맞는 인덱스가 있는지 확인합니다.
- 한 번에 수정하거나 조회하는 행 수가 적절한지 확인합니다.
- 트랜잭션 범위가 너무 넓거나 짧지 않은지 확인합니다.

JPA는 SQL 작성을 줄여 주지만 데이터베이스 비용까지 자동으로 최적화하지는 않습니다. ORM을 블랙박스로 사용하지 않고
생성 SQL과 실행 계획을 함께 확인해야 합니다.

## ORM 사용의 판단 근거

### 사용해야 하는 경우

- 도메인 모델 중심의 애플리케이션
- CRUD 위주의 비즈니스 로직
- 객체지향 설계를 중시하는 프로젝트
- 팀의 생산성이 중요한 경우
- DB 변경 가능성이 있는 경우

### 사용을 신중하게 고려해야 하는 경우

- 복잡한 통계/분석 쿼리가 많은 경우
- 초당 수만 건 이상의 대용량 처리
- 레거시 DB 스키마가 매우 복잡한 경우
- SQL 튜닝이 핵심인 프로젝트
