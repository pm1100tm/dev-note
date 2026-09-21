# 기본 Entity Mapping

기본 Entity Mapping은 Java 객체를 어느 테이블과 컬럼에 저장할지 JPA에
알려 주는 규칙입니다. 쉽게 말해 `Member` 클래스의 각 필드에 데이터베이스의
저장 위치를 붙이는 작업입니다.

매핑이 정확해야 JPA가 원하는 테이블에 올바른 SQL을 실행합니다. 이후에 배울
연관관계도 기본 매핑이 먼저 올바르게 되어 있어야 안전하게 연결할 수 있습니다.

## 목차

- [Entity와 테이블 연결](#entity와-테이블-연결)
- [기본 키 매핑](#기본-키-매핑)
- [컬럼 매핑](#컬럼-매핑)
- [Enum과 날짜·시간 매핑](#enum과-날짜시간-매핑)
- [매핑에서 제외하거나 묶기](#매핑에서-제외하거나-묶기)
- [하나의 Member 엔티티로 정리하기](#하나의-member-엔티티로-정리하기)

## Entity와 테이블 연결

`@Entity`는 이 객체를 데이터베이스에 저장하고 조회할 대상이라고 JPA에 알립니다.
`@Table`은 이 객체가 어떤 테이블과 연결되는지 지정할 때 사용합니다.

```java
import jakarta.persistence.Entity;
import jakarta.persistence.Table;

@Entity
@Table(name = "members")
public class Member {
    // 필드는 다음 절에서 정의합니다.
}
```

`@Table`을 생략하면 JPA가 클래스 이름에서 테이블 이름을 만듭니다. 예를 들어
Spring Boot와 Hibernate의 일반적인 설정에서는 `MemberProfile`이
`member_profile`처럼 변환될 수 있습니다. 이 이름 변환 규칙을 네이밍 전략이라고
합니다.

새 프로젝트에서는 규칙에 맡겨도 됩니다. 이미 운영 중인 테이블을 연결한다면
추측에 맡기지 말고 `@Table(name = "...")`으로 실제 이름을 적는 편이 안전합니다.

### 잘못된 예: 기존 테이블 이름을 추측에 맡기기

```java
@Entity
public class Member {
    // 실제 테이블 이름은 tbl_member입니다.
}
```

JPA가 `member`라는 테이블을 찾는데 실제 이름은 `tbl_member`라면 조회와 저장이
실패합니다. 운영 데이터베이스에서 테이블 자동 생성·변경 설정까지 켜면, 의도하지
않은 스키마 변경으로 이어질 수 있습니다.

### 올바른 예: 기존 테이블과 컬럼 이름을 명시하기

```java
@Entity
@Table(name = "tbl_member")
public class Member {

    @Column(name = "member_name", nullable = false, length = 50)
    private String name;
}
```

`@Entity`에는 인자가 없는 기본 생성자가 필요합니다. JPA가 조회 결과를 바탕으로
객체를 만들 때 이 생성자를 사용하기 때문입니다. 애플리케이션 코드에서 함부로
호출하지 못하게 보통 protected로 선언합니다.

```java
protected Member() {
}
```

## 기본 키 매핑

모든 엔티티는 `@Id`로 기본 키를 지정해야 합니다. 기본 키는 회원 번호처럼 한 행을
구별하는 값입니다. JPA도 이 값을 기준으로 "이 객체가 어느 데이터인지" 판단합니다.

```java
import jakarta.persistence.GeneratedValue;
import jakarta.persistence.GenerationType;
import jakarta.persistence.Id;

@Id
@GeneratedValue(strategy = GenerationType.IDENTITY)
private Long id;
```

`@GeneratedValue`는 기본 키를 누가 어떻게 만들지 정합니다. 사용하는 데이터베이스와
저장 성능 요구 사항에 따라 선택이 달라집니다.

| 전략       | 기본 동작                                | 주로 고려할 데이터베이스 |
| ---------- | ---------------------------------------- | ------------------------ |
| `IDENTITY` | DB의 자동 증가 기능으로 번호를 만듭니다. | MySQL                    |
| `SEQUENCE` | DB의 번호 발급기인 시퀀스를 사용합니다.  | PostgreSQL, Oracle, H2   |
| `TABLE`    | 번호를 저장한 별도 테이블을 사용합니다.  | 시퀀스가 없는 환경       |
| `AUTO`     | JPA 구현체가 DB에 맞는 전략을 고릅니다.  | 전략 선택을 맡길 때      |
| `UUID`     | 애플리케이션에서 고유한 UUID를 만듭니다. | 분산 시스템              |

- MySQL에서는 보통 `IDENTITY`와 `AUTO_INCREMENT`를 사용합니다.
- PostgreSQL에서는 보통 `SEQUENCE`를 사용합니다.
- MariaDB는 DB 차원에서 시퀀스를 지원하지만, JPA `SEQUENCE` 전략 적용 전에는
  MariaDB 버전과 Hibernate Dialect, 생성 SQL을 확인해야 합니다.
- 기존 테이블의 키 생성 방식이 정해져 있다면 그 방식을 따라야 합니다.

`IDENTITY`는 데이터베이스가 번호를 만든 뒤에야 그 값을 알 수 있습니다. 그래서
Hibernate는 `persist()` 때 INSERT SQL을 일찍 실행할 수 있습니다. 이는 Hibernate와
데이터베이스 조합에 따른 동작이므로, 실제 SQL 로그로 확인해야 합니다.

다음은 PostgreSQL처럼 시퀀스를 지원하는 데이터베이스에서 사용하는 예제입니다.
이 예시는 애플리케이션이 ID를 여러 개씩 미리 확보하도록 선택한 경우입니다.

```java
import jakarta.persistence.SequenceGenerator;

@Id
@GeneratedValue(strategy = GenerationType.SEQUENCE, generator = "member_seq")
@SequenceGenerator(
    name = "member_seq",
    sequenceName = "member_seq",
    allocationSize = 50
)
private Long id;
```

`allocationSize = 50`은 한 번 시퀀스를 조회할 때 ID 50개 범위를 확보하여 DB 왕복을
줄입니다. JPA 표준의 기본값도 50입니다. 따라서 어노테이션에 이 값을 직접 보지
못했더라도, `SEQUENCE` 전략에서는 이미 적용되어 있을 수 있습니다.

하지만 `50`은 모든 프로젝트에 복사할 값이 아닙니다. Hibernate가 외부에서 만든
시퀀스를 사용할 때는 `allocationSize`와 DB 시퀀스의 `INCREMENT BY`를 맞춰야
합니다. 위 예시를 선택했다면 DB 스키마도 다음처럼 관리합니다.

```sql
CREATE SEQUENCE member_seq START WITH 1 INCREMENT BY 50;
```

### `allocationSize`의 실무 선택

`allocationSize`는 `SEQUENCE` 전략에서만 사용합니다. MySQL처럼 보통 `IDENTITY`를
사용하는 프로젝트에서는 보지 못하는 것이 자연스럽습니다.

- 기존 시퀀스가 `INCREMENT BY 1`이면, 기존 설정을 바꾸기 어렵다면
  `allocationSize = 1`로 맞춥니다.
- 새 서비스에서 스키마를 함께 관리한다면, `50`을 시작값으로 명시하고 시퀀스
  증가 폭도 50으로 맞출 수 있습니다.
- INSERT가 적으면 번호 선확보의 이점이 작으므로 단순한 설정을 우선합니다.
- 대량 INSERT 또는 쓰기 부하가 크면 부하 측정 후 적절한 크기로 늘려 DB 왕복을
  줄입니다.

즉, `50`은 Hibernate에서 흔히 사용하는 정상적인 성능 최적화 값이지만 필수 규칙은
아닙니다. 특히 기존 DB에서는 시퀀스 정의, Hibernate 버전과 설정, 실제 생성 SQL을
먼저 확인해야 합니다.

ID 범위를 미리 확보하면 애플리케이션 재시작이나 트랜잭션 롤백 뒤에 번호가 건너뛸
수 있습니다. 기본 키는 순번을 보여 주는 값이 아니라 데이터를 구분하는 값입니다.
연속 번호가 업무상 필요하다면 별도의 업무 번호를 설계해야 합니다.

## 컬럼 매핑

`String`, `Long`, `LocalDateTime` 같은 기본 타입 필드는 어노테이션이 없어도
컬럼으로 연결됩니다. `@Column`은 컬럼 이름과 저장 규칙을 더 구체적으로 적을 때
사용합니다.

```java
import java.math.BigDecimal;
import jakarta.persistence.Column;

@Column(name = "member_name", nullable = false, length = 50)
private String name;

@Column(nullable = false, precision = 12, scale = 2)
private BigDecimal credit;
```

- `name`은 실제 컬럼 이름입니다. 기존 스키마에서는 명시합니다.
- `nullable`은 빈 값(NULL) 허용 여부입니다. API 입력값 검증을 대신하지는
  않습니다.
- `length`는 문자열 최대 길이입니다. DB가 지원하는 타입도 확인합니다.
- `precision`, `scale`은 숫자의 전체·소수 자릿수입니다. 금액에는
  `BigDecimal`을 사용합니다.
- `updatable`은 JPA UPDATE SQL 포함 여부입니다. DB 자체의 변경을 막지는
  않습니다.
- `columnDefinition`은 DB 전용 컬럼 정의를 직접 적습니다. 다른 DB로 옮기기
  어려워집니다.

DDL은 `CREATE TABLE`처럼 테이블 구조를 만드는 SQL입니다. `nullable`, `unique`,
`length`는 JPA가 테이블을 자동 생성할 때 주로 반영됩니다. 이미 운영 중인 테이블을
수정하는 설정은 피하고, `ddl-auto=validate` 또는 `none`으로 구조만 검증합니다.

`nullable = false`만으로 요청값을 친절하게 검사할 수는 없습니다. API 입력값은
Bean Validation으로 검사하고, 최종 데이터 보호는 데이터베이스 제약조건으로 합니다.

### 잘못된 예: 금액에 `double` 사용하기

```java
private double credit;
```

`double`은 컴퓨터가 소수를 저장하는 방식 때문에 0.1 같은 값을 정확히 담지 못할
수 있습니다. 금액을 더하거나 비교할 때 아주 작은 오차가 생길 수 있습니다.

### 올바른 예: 금액에 `BigDecimal` 사용하기

```java
@Column(nullable = false, precision = 12, scale = 2)
private BigDecimal credit;

public void addCredit(BigDecimal amount) {
    this.credit = this.credit.add(amount);
}
```

`BigDecimal`은 `new BigDecimal("0.1")`처럼 문자열로 만들거나, 원 단위처럼
가장 작은 금액 단위를 정수로 저장하는 방식을 사용합니다.

## Enum과 날짜·시간 매핑

Enum은 회원 상태처럼 정해진 값 중 하나만 선택하게 할 때 사용합니다. 기본값인
`ORDINAL`로 저장하면 `ACTIVE`는 0, `INACTIVE`는 1처럼 선언 순서의 숫자가
저장됩니다.

### 잘못된 예: `ORDINAL`에 의존하기

```java
@Enumerated(EnumType.ORDINAL)
private MemberStatus status;

enum MemberStatus {
    ACTIVE,
    INACTIVE
}
```

나중에 `PENDING`을 맨 앞에 추가하면 데이터베이스의 `0`은 기존 `ACTIVE`가
아니라 `PENDING`으로 읽힙니다.

### 올바른 예: Enum 이름을 문자열로 저장하기

```java
import jakarta.persistence.EnumType;
import jakarta.persistence.Enumerated;

@Enumerated(EnumType.STRING)
@Column(nullable = false, length = 20)
private MemberStatus status;

enum MemberStatus {
    ACTIVE,
    INACTIVE
}
```

`EnumType.STRING`은 순서 변경 문제를 피하지만 이름 변경까지 자동으로 처리하지는
않습니다. DB에 `ACTIVE`가 저장된 뒤 Enum 이름을 바꾸려면 데이터도 함께 바꿔야
합니다. 외부 시스템이 정한 코드값을 써야 한다면 `AttributeConverter`를 검토합니다.

새 코드에서는 Java 8 이상의 날짜·시간 타입을 사용합니다. `LocalDate`는 날짜만,
`LocalDateTime`은 날짜와 시간, `Instant`는 UTC 기준의 특정 시점을 표현합니다.

```java
import java.time.LocalDate;
import java.time.LocalDateTime;

private LocalDate birthDate;

@Column(nullable = false, updatable = false)
private LocalDateTime createdAt;
```

`LocalDateTime`에는 "서울 시간인지, 뉴욕 시간인지" 정보가 없습니다. 여러 국가의
시간을 다룬다면 UTC 기준으로 저장하고 `Instant`를 사용하는 방식을 먼저 검토합니다.
실제 SQL 타입과 시간대 처리 방식은 데이터베이스와 Hibernate 설정을 확인합니다.

## 매핑에서 제외하거나 묶기

`@Transient`는 필드를 데이터베이스에 저장하지 말라고 JPA에 알립니다. 화면에만
보여 줄 값이나 계산 결과처럼 다시 만들어 낼 수 있는 값에 사용합니다.

```java
import jakarta.persistence.Transient;

@Transient
private String displayName;
```

`@Embedded`와 `@Embeddable`은 관련 있는 값을 작은 객체로 묶는 기능입니다.
예를 들어 Email 객체를 만들더라도 이메일만 저장한다면 별도 `email` 테이블은
생기지 않고 `members` 테이블의 `email` 컬럼에 저장됩니다.

```java
import jakarta.persistence.Embeddable;
import jakarta.persistence.Embedded;

@Embeddable
public class Email {

    @Column(name = "email", nullable = false, length = 100)
    private String value;

    protected Email() {
    }

    public Email(String value) {
        this.value = value;
    }
}

@Entity
@Table(name = "members")
public class Member {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @Embedded
    private Email email;
}
```

값 타입은 한 객체를 여러 엔티티가 함께 수정하지 않도록 불변으로 설계하는 편이
안전합니다. 즉, 값을 바꿔야 하면 기존 Email 객체의 필드를 수정하기보다 새 Email
객체를 만들어 교체합니다.

## 하나의 Member 엔티티로 정리하기

```java
@Entity
@Table(name = "members")
public class Member {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @Column(name = "member_name", nullable = false, length = 50)
    private String name;

    @Enumerated(EnumType.STRING)
    @Column(nullable = false, length = 20)
    private MemberStatus status;

    @Column(nullable = false, precision = 12, scale = 2)
    private BigDecimal credit;

    @Embedded
    private Email email;

    @Column(nullable = false, updatable = false)
    private LocalDateTime createdAt;

    protected Member() {
    }

    public void changeName(String name) {
        this.name = name;
    }
}
```

엔티티 매핑은 어노테이션을 붙이는 작업이지만, 결과적으로 데이터베이스 구조를
정하는 일입니다. 기본 키 방식과 컬럼 타입을 바꾸면 API, 기존 데이터, 배포 절차에
영향을 줄 수 있습니다.

개발 환경에서는 DDL 자동 생성을 써서 빠르게 확인할 수 있습니다. 운영 환경의
스키마 변경은 Flyway 같은 마이그레이션 도구와 검토 절차로 관리합니다.
