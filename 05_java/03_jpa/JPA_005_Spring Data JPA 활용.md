# Spring Data JPA 활용

Spring Data JPA는 repository 구현의 반복을 줄여 주지만, 메서드 하나가 모든 조회와
수정을 해결해 주지는 않습니다. 실무에서는 조회 목적, 반환 데이터, 트랜잭션 경계에
맞춰 repository 메서드를 작게 설계해야 합니다.

이 문서에서는 `Member`를 기준으로 복잡한 서비스에서 자주 필요한 조회, 수정,
페이징, 감사 기능을 Spring Data JPA로 구성하는 기준을 설명합니다.

## 목차

- [Repository의 역할](#repository의-역할)
- [메서드 이름 쿼리의 범위](#메서드-이름-쿼리의-범위)
- [JPQL과 DTO 조회](#jpql과-dto-조회)
- [페이징과 정렬](#페이징과-정렬)
- [동적 조건에는 Specification](#동적-조건에는-specification)
- [수정 쿼리와 영속성 컨텍스트](#수정-쿼리와-영속성-컨텍스트)
- [save와 트랜잭션의 역할](#save와-트랜잭션의-역할)
- [Auditing으로 생성·수정 시각 관리하기](#auditing으로-생성수정-시각-관리하기)
- [실무 설계 기준](#실무-설계-기준)
- [참고 자료](#참고-자료)

## Repository의 역할

repository는 도메인 객체를 저장하고 조회하는 경계입니다. 서비스는 업무 규칙과
트랜잭션을 담당하고, repository는 데이터 접근 의도를 메서드로 표현합니다.

```java
public interface MemberRepository
        extends JpaRepository<Member, Long> {

    Optional<Member> findByEmail(String email);
}
```

`JpaRepository`는 `save()`, `findById()`, `findAll()`, `deleteById()` 같은
기본 기능을 제공합니다. 따라서 단순 CRUD 구현을 매번 작성할 필요가 없습니다.

그러나 repository를 모든 업무 로직을 넣는 장소로 만들면 안 됩니다. 예를 들어
회원 가입 가능 여부, 외부 API 호출, 권한 검사는 서비스가 담당해야 합니다.
repository 메서드는 "어떤 데이터를 어떤 조건으로 가져오는가"에 집중합니다.

## 메서드 이름 쿼리의 범위

간단한 조건은 메서드 이름으로 충분히 표현할 수 있습니다.

```java
List<Member> findByStatus(MemberStatus status);

boolean existsByEmail(String email);
```

`existsByEmail()`은 이메일 중복 여부만 필요할 때 엔티티 전체를 조회하지 않도록
의도를 드러냅니다. 다만 애플리케이션의 사전 확인만으로 중복을 막을 수는 없으므로,
`email` 컬럼에는 DB UNIQUE 제약 조건도 반드시 둬야 합니다.

### 잘못된 예: 메서드 이름에 모든 조건 넣기

```java
Page<Member> findByNameContainingAndStatusAndCreatedAtBetweenAndOrdersStatus(
    String name,
    MemberStatus status,
    LocalDateTime from,
    LocalDateTime to,
    OrderStatus orderStatus,
    Pageable pageable
);
```

이 메서드는 조건 하나만 늘어나도 이름과 인자가 빠르게 복잡해집니다. 조회 목적과
JOIN 여부를 파악하기 어렵고, 조건 조합마다 메서드가 늘어납니다.

### 올바른 예: 단순 조회와 복잡한 조회를 분리하기

```java
Optional<Member> findByEmail(String email);

Page<MemberSummary> findByStatus(MemberStatus status, Pageable pageable);
```

단순하고 자주 쓰는 조건은 메서드 이름으로 유지합니다. 검색 조건이 선택적으로
결합되거나 JOIN·집계가 필요하면 `@Query`, Specification, 별도 조회 repository를
선택합니다.

## JPQL과 DTO 조회

목록 화면에 회원 이름과 상태만 필요하다면 엔티티 전체를 조회하지 않고 DTO를
반환하는 편이 명확합니다. DTO는 API 응답에 필요한 데이터와 조회 범위를 함께
표현합니다.

```java
public record MemberSummary(
    Long id,
    String name,
    MemberStatus status
) {
}
```

```java
@Query("""
    select new com.example.member.MemberSummary(m.id, m.name, m.status)
    from Member m
    where m.status = :status
""")
Page<MemberSummary> findSummariesByStatus(
    @Param("status") MemberStatus status,
    Pageable pageable
);
```

JPQL 생성자 표현식의 클래스 이름은 전체 패키지 이름을 포함해야 합니다. 실제
프로젝트에서는 `com.example.member`를 DTO의 실제 패키지로 바꿉니다.

### 잘못된 예: 목록 조회에 엔티티를 그대로 반환하기

```java
Page<Member> findByStatus(MemberStatus status, Pageable pageable);
```

이 메서드 자체가 항상 잘못된 것은 아닙니다. 그러나 목록 화면이 일부 필드만 쓰는데
엔티티를 API 응답으로 그대로 반환하면 불필요한 컬럼과 연관관계가 노출될 수 있습니다.
지연 로딩으로 N+1이 발생하거나 JSON 순환 참조가 생길 위험도 있습니다.

### 올바른 예: 조회 모델을 반환하기

```java
Page<MemberSummary> findSummariesByStatus(
    MemberStatus status,
    Pageable pageable
);
```

조회 전용 DTO는 변경 감지 대상이 아니므로 수정 용도로 사용하지 않습니다. 명령
처리에는 영속 엔티티를 조회해 도메인 메서드로 상태를 바꾸고, 조회에는 DTO를
사용하는 방식이 규모가 큰 서비스에서 이해하기 쉽습니다.

## 페이징과 정렬

`Pageable`은 페이지 번호, 크기, 정렬 조건을 repository 메서드에 전달합니다.
`Page`는 내용과 전체 건수를 함께 반환하지만, 보통 전체 건수를 위한 count 쿼리도
실행합니다.

```java
Page<MemberSummary> page = memberRepository.findSummariesByStatus(
    MemberStatus.ACTIVE,
    PageRequest.of(0, 20, Sort.by("id").descending())
);
```

무한 스크롤처럼 다음 데이터 존재 여부만 필요하면 `Slice`를 고려합니다. `Slice`는
일반적으로 전체 건수 count 쿼리를 피할 수 있어 큰 테이블에서 비용을 줄일 수
있습니다.

```java
Slice<MemberSummary> findByStatus(
    MemberStatus status,
    Pageable pageable
);
```

컬렉션 fetch join과 `Pageable`을 함께 사용하면 결과 행이 늘어나 페이지가 부정확해질
수 있습니다. 회원 ID를 먼저 페이징한 후 상세 데이터를 별도 조회하는 방식으로
분리합니다. 복잡한 JPQL 또는 native query의 페이징은 count 쿼리가 올바른지 반드시
SQL 로그와 테스트로 확인합니다.

## 동적 조건에는 Specification

관리자 검색처럼 이름, 상태, 가입 기간이 선택적으로 들어오면 메서드 이름 쿼리를
조합하기 어렵습니다. Spring Data JPA의 `Specification`은 JPA Criteria API 기반의
조건을 재사용하고 조합하도록 돕습니다.

```java
public interface MemberRepository
        extends JpaRepository<Member, Long>, JpaSpecificationExecutor<Member> {
}
```

```java
public final class MemberSpecifications {

    private MemberSpecifications() {
    }

    public static Specification<Member> hasStatus(MemberStatus status) {
        return (root, query, builder) ->
            status == null ? null : builder.equal(root.get("status"), status);
    }
}
```

```java
Specification<Member> spec = MemberSpecifications.hasStatus(status);
Page<Member> page = memberRepository.findAll(spec, pageable);
```

Specification은 조건 조합에는 적합하지만, 복잡한 통계·그룹화·여러 DTO 조합까지
무리하게 담으면 읽기 어려워집니다. 이런 조회는 명시적인 JPQL, Querydsl 또는 별도의
조회 전용 repository로 분리하는 편이 낫습니다. Querydsl은 Spring Data JPA 표준
기능이 아니라 별도 라이브러리라는 점도 구분해야 합니다.

## 수정 쿼리와 영속성 컨텍스트

대량 상태 변경은 엔티티를 한 건씩 조회하지 않고 JPQL update를 사용할 수 있습니다.
이때 `@Modifying`을 붙이고 트랜잭션 안에서 실행합니다.

```java
@Modifying(clearAutomatically = true, flushAutomatically = true)
@Query("""
    update Member m
    set m.status = :status
    where m.id = :memberId
""")
int updateStatus(
    @Param("memberId") Long memberId,
    @Param("status") MemberStatus status
);
```

```java
@Transactional
public void deactivate(Long memberId) {
    memberRepository.updateStatus(memberId, MemberStatus.INACTIVE);
}
```

JPQL bulk update는 영속성 컨텍스트를 거치지 않고 DB에 바로 반영합니다. 이미 영속성
컨텍스트에 같은 `Member`가 있으면 값이 오래된 상태일 수 있습니다.

- `clearAutomatically = true`는 수정 뒤 영속성 컨텍스트를 비워 이 문제를 줄입니다.
- `flushAutomatically = true`는 실행 전 변경 내용을 flush합니다.

이 두 설정은 Spring Data JPA 기능이며, 해당 메서드 전후에 관리 중인 엔티티가 있는지 확인하고 사용합니다.

bulk update는 엔티티의 변경 감지와 생명주기 콜백을 개별 엔티티 수정과 똑같이
처리하지 않습니다. `@PreUpdate`, 감사 필드, 도메인 검증이 꼭 실행되어야 한다면
영속 엔티티를 조회해 도메인 메서드로 변경합니다.

## save와 트랜잭션의 역할

새 엔티티는 `save()`로 저장할 수 있습니다.

```java
@Transactional
public Long register(RegisterMemberCommand command) {
    Member member = new Member(command.name(), command.email());
    Member savedMember = memberRepository.save(member);
    return savedMember.getId();
}
```

이미 조회한 영속 엔티티는 트랜잭션 안에서 상태만 바꿔도 변경 감지로 UPDATE가 반영됩니다.

```java
@Transactional
public void changeName(Long memberId, String name) {
    Member member = memberRepository.getReferenceById(memberId);
    member.changeName(name);
}
```

### 잘못된 예: 변경마다 무조건 `save()` 호출하기

```java
@Transactional
public void changeName(Long memberId, String name) {
    Member member = memberRepository.findById(memberId).orElseThrow();
    member.changeName(name);
    memberRepository.save(member);
}
```

이 코드에서 `member`는 이미 영속 상태이므로 마지막 `save()`는 필요하지 않습니다.
Spring Data JPA의 기본 repository 구현은 엔티티가 새 객체인지 판단해 `persist` 또는
`merge`를 선택합니다. 구체적인 새 엔티티 판단과 `merge` 동작은 구현체와 설정을
확인해야 하며, 분리된 엔티티를 무심코 `save()`하는 방식은 의도하지 않은 상태 복사로
이어질 수 있습니다.

`getReferenceById()`는 프록시를 반환할 수 있으며, 실제 데이터 확인은 필드 접근 시점에
일어날 수 있습니다. 존재 여부를 바로 검증해야 한다면 `findById()`를 사용합니다.

## Auditing으로 생성·수정 시각 관리하기

여러 엔티티에 생성·수정 시각을 반복해서 넣는 대신 Spring Data JPA Auditing을 사용할
수 있습니다. 이 기능은 Spring Data JPA가 제공하며, JPA 표준 어노테이션은 아닙니다.

```java
@Configuration
@EnableJpaAuditing
public class JpaAuditConfig {
}
```

```java
@EntityListeners(AuditingEntityListener.class)
@Entity
public class Member {

    @CreatedDate
    @Column(nullable = false, updatable = false)
    private LocalDateTime createdAt;

    @LastModifiedDate
    @Column(nullable = false)
    private LocalDateTime updatedAt;
}
```

감사 필드는 서버 시간대와 DB 시간대 정책을 먼저 정한 뒤 사용합니다. DB 기본값이나
트리거도 함께 사용한다면 어느 쪽이 최종 값을 관리하는지 정해야 값이 엇갈리지
않습니다.

## 실무 설계 기준

- 단순 CRUD는 `JpaRepository`를 사용하고, 업무 규칙은 서비스에 둡니다.
- 간단한 조건만 메서드 이름 쿼리로 만들고, 긴 이름은 명시적 조회 방식으로 바꿉니다.
- 목록·검색 API는 엔티티보다 DTO projection을 우선 검토합니다.
- `Page`의 count 쿼리 비용을 확인하고, 필요하면 `Slice`나 키셋 방식도 검토합니다.
- 동적 조건은 Specification으로 조합하되, 복잡한 리포트 조회와 분리합니다.
- bulk update 뒤에는 영속성 컨텍스트의 오래된 데이터를 주의합니다.
- 외래 키, UNIQUE 제약, 인덱스는 repository 메서드가 아니라 DB 스키마로 보장합니다.
- 생성 SQL, 실행 계획, 실제 데이터 건수를 확인한 뒤 성능 개선을 적용합니다.

## 참고 자료

- [Spring Data JPA 참고 문서](https://docs.spring.io/spring-data/jpa/reference/)
- [Spring Data JPA Specifications](https://docs.spring.io/spring-data/jpa/reference/jpa/specifications.html)
