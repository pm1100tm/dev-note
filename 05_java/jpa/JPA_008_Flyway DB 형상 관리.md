# Flyway DB 형상 관리

데이터베이스 스키마는 애플리케이션 코드와 함께 배포되는 계약입니다. Flyway는
테이블, 인덱스, 제약 조건, 필요한 데이터 변환을 버전이 있는 마이그레이션으로
관리해 개발·테스트·운영 환경의 스키마를 같은 이력으로 맞추는 도구입니다.

이 문서는 Spring Boot 3과 JPA를 기준으로 설명합니다. Flyway의 의존성 구성과
DB별 지원 모듈은 Flyway 및 Spring Boot 버전에 따라 달라질 수 있으므로, 적용할
버전의 공식 호환성 문서를 확인해야 합니다.

## 목차

- [Flyway가 필요한 이유](#flyway가-필요한-이유)
- [마이그레이션의 동작과 파일 규칙](#마이그레이션의-동작과-파일-규칙)
- [Spring Boot와 JPA 설정](#spring-boot와-jpa-설정)
- [작은 스키마 변경 예시](#작은-스키마-변경-예시)
- [잘못된 변경과 올바른 변경](#잘못된-변경과-올바른-변경)
- [기존 DB를 도입할 때](#기존-db를-도입할-때)
- [무중단 배포를 위한 확장-수축 전략](#무중단-배포를-위한-확장-수축-전략)
- [운영 명령과 장애 대응](#운영-명령과-장애-대응)
- [실무 체크리스트](#실무-체크리스트)
- [참고 자료](#참고-자료)

## Flyway가 필요한 이유

개발 환경에서 `spring.jpa.hibernate.ddl-auto=update`는 엔티티 변경을 빠르게
확인하는 데 도움이 될 수 있습니다. 그러나 운영 스키마를 안전하게 바꾸는 배포
도구는 아닙니다. 데이터 이관 순서, 인덱스 생성 영향, 컬럼 이름 변경, 팀원 간
적용 이력을 코드 리뷰 가능한 형태로 남기기 어렵습니다.

### 잘못된 예: 운영에서 Hibernate DDL 자동 변경에 의존하기

```yaml
spring:
  jpa:
    hibernate:
      ddl-auto: update
```

`update`는 Hibernate 구현체의 스키마 생성 기능이며 JPA 표준 기능이 아닙니다.
복잡한 이름 변경·데이터 변환의 의도를 표현하지 못하고, DB와 Hibernate 버전에
따라 적용 결과가 달라질 수 있습니다. 운영 중인 테이블을 예측 없이 변경하면
배포 실패나 데이터 손실 위험이 있습니다.

### 올바른 방향: 스키마 변경도 코드처럼 이력으로 관리하기

```text
애플리케이션 코드 변경
        +
V12__add_order_status.sql
        +
코드 리뷰 · 테스트 · 배포 순서 검증
```

Flyway는 적용한 이력을 `flyway_schema_history` 테이블에 기록합니다. 버전
마이그레이션은 버전 순서대로 한 번 적용되고, 적용 당시의 checksum도 보관합니다.
다른 환경에서 같은 파일이 달라졌는지 검증할 수 있어 팀의 DB 상태를 재현하기
쉬워집니다.

## 마이그레이션의 동작과 파일 규칙

SQL 기반 버전 마이그레이션의 기본 이름 형식은 다음과 같습니다.

```text
V<version>__<description>.sql
```

```text
V1__create_products.sql
V2__create_orders.sql
V3__add_status_to_orders.sql
```

`V`는 versioned migration을 뜻하고, 두 개의 밑줄(`__`)은 버전과 설명의
구분자입니다. 각 버전은 고유해야 하며 Flyway는 버전 순서대로 적용합니다.
반복 실행할 뷰·프로시저 같은 변경에는 `R__refresh_order_summary.sql`처럼
repeatable migration을 사용할 수 있습니다. repeatable migration은 버전 순서가
아니라 checksum이 바뀌었는지에 따라 다시 적용됩니다.

한 번 영구 환경에 적용된 versioned migration은 수정하지 않습니다. 오타나 설계
오류가 발견되면 다음 버전 파일을 추가해 앞으로 진행합니다. 이미 적용된 파일을
바꾸면 checksum 검증 실패가 나고, 무엇보다 환경마다 실제 DB 변경 내용이 달라질
수 있습니다.

`flyway_schema_history`는 Flyway의 관리 테이블입니다. 애플리케이션 코드나
운영자가 직접 INSERT·UPDATE·DELETE해서 상태를 맞추면 실제 스키마와 이력이
어긋날 수 있으므로 수정하지 않습니다.

## Spring Boot와 JPA 설정

Spring Boot 애플리케이션에는 Flyway Core와 사용하는 DB에 필요한 Flyway
데이터베이스 모듈을 추가합니다. 최근 Flyway는 DB별 지원을 별도 모듈로 제공할 수
있습니다. 의존성 이름과 버전은 사용하는 Flyway 버전의 공식 문서를 따릅니다.

```java
dependencies {
    implementation("org.flywaydb:flyway-core")
    implementation("org.flywaydb:flyway-database-postgresql")
}
```

Spring Boot의 의존성 관리를 쓴다면 Flyway 버전을 임의로 고정하기 전에 Boot가
관리하는 버전과 호환되는지 확인합니다. PostgreSQL 예시는 다른 DB에 그대로
복사하지 않습니다.

기본 SQL 위치는 `src/main/resources/db/migration`입니다. 다음 설정은 운영에서
DDL 변경의 주체를 Flyway로 한정하고, Hibernate에는 엔티티 매핑 검증만 맡깁니다.

```yaml
spring:
  flyway:
    enabled: true
    locations: classpath:db/migration
    clean-disabled: true
  jpa:
    hibernate:
      ddl-auto: validate
```

`ddl-auto=validate`는 Hibernate가 엔티티 매핑과 실제 스키마의 기본 호환성을
확인하도록 돕지만, Flyway 마이그레이션의 테스트를 대체하지는 않습니다. 예를
들어 인덱스 설계, 데이터 변환 결과, 뷰 정의까지 업무 요구에 맞는지 확인하려면
빈 DB에서 migrate한 통합 테스트가 필요합니다.

`clean-disabled`의 기본값과 프로퍼티 지원 방식은 Flyway 버전에 따라 확인합니다.
운영 설정에서는 `clean`이 실행되지 않게 명시합니다. `clean`은 대상 schema의
객체를 삭제하므로 운영 DB에 실행해서는 안 됩니다.

## 작은 스키마 변경 예시

주문 상태가 필요한 경우, 엔티티 코드와 같은 변경에서 새 마이그레이션을
추가합니다. 아래 SQL은 PostgreSQL 예시입니다.

```sql
-- V3__add_status_to_orders.sql
alter table orders
    add column status varchar(20);

update orders
set status = 'CREATED'
where status is null;

alter table orders
    alter column status set not null;

alter table orders
    add constraint ck_orders_status
    check (status in ('CREATED', 'PAID', 'CANCELLED'));
```

```java
@Entity
@Table(name = "orders")
public class Order {

    @Enumerated(EnumType.STRING)
    @Column(nullable = false, length = 20)
    private OrderStatus status;
}
```

데이터를 먼저 채운 뒤 `NOT NULL` 제약을 추가해야 기존 행 때문에 실패하지
않습니다. `CHECK` 제약의 문법·지원 여부, `ALTER TABLE`의 잠금 영향은 DB마다
다릅니다. 대형 테이블에서는 update 한 번이 긴 잠금과 WAL·redo 부하를 만들 수
있으므로 배치 작업이나 별도 배포 단계가 필요한지 확인합니다.

## 잘못된 변경과 올바른 변경

### 잘못된 예: 적용된 파일을 수정하기

```sql
-- 이미 운영에 적용한 V3__add_status_to_orders.sql을 수정합니다.
alter table orders
    add column status varchar(50) not null;
```

기존 환경의 history checksum과 저장소의 파일 checksum이 달라져 `validate`가
실패합니다. 더 큰 문제는 어떤 환경에는 길이 20, 어떤 환경에는 길이 50인 컬럼이
남는다는 점입니다. checksum 오류를 없애려고 `repair`만 실행하면 이 차이를
정상 상태로 기록할 수 있으므로 해결책이 아닙니다.

### 올바른 예: 다음 버전으로 의도를 추가하기

```sql
-- V4__extend_order_status_length.sql
alter table orders
    alter column status type varchar(50);
```

새 마이그레이션은 모든 환경에 같은 순서로 적용됩니다. 변경의 이유와 배포 영향을
PR에 남기고, 복원해야 할 상황에서는 이전 파일을 수정하는 대신 별도의 forward
migration 또는 DB 백업·복구 절차를 준비합니다.

### 잘못된 예: 실패 기록을 SQL로 직접 지우기

```sql
delete from flyway_schema_history
where success = false;
```

실패한 SQL이 테이블이나 데이터를 일부 바꿨을 수 있습니다. history 행만 지우면
Flyway는 변경이 없었다고 판단해 더 위험한 재실행을 할 수 있습니다.

### 올바른 예: 실제 DB 상태를 먼저 확인하고 복구하기

```text
1. 실패한 migration과 DB의 부분 변경을 확인합니다.
2. 안전한 복구 또는 다음 forward migration 계획을 검토합니다.
3. 수정된 스크립트와 실제 DB 상태를 검증합니다.
4. 필요한 경우에만 Flyway repair를 검토합니다.
```

`repair`는 history의 실패 항목·checksum 등 메타데이터를 정리하는 명령이며,
스키마나 데이터를 되돌리는 명령이 아닙니다. 운영에서는 DBA와 배포 담당자의
검토, 백업·복구 절차, 영향 범위를 확인한 뒤 실행합니다.

## 기존 DB를 도입할 때

이미 운영 중인 DB에 Flyway를 처음 적용할 때, 과거 모든 변경을 추측해 파일로
재구성해서 바로 실행하면 안 됩니다. 현재 스키마를 기준점으로 삼고 이후 변경부터
관리하는 `baseline` 절차를 검토합니다.

`flyway baseline` 명령은 기존 DB에 history 테이블과 기준 버전을 기록합니다.
이는 `B<version>__...sql` 형태의 baseline migration 파일과 다른 개념입니다.
baseline migration 파일은 새 환경을 빠르게 만들기 위한 누적 스키마 스크립트이고,
기존에 migration 이력이 있는 환경에서는 일반적으로 무시됩니다.

기준 버전, 실제 운영 스키마 덤프, 다음 migration 번호를 합의한 뒤 복제한
스테이징 DB에서 먼저 검증합니다. `baseline-on-migrate`를 운영에 무심코 켜면
잘못된 대상 DB를 기준 처리할 수 있으므로 자동화 전 대상 검증을 강화해야 합니다.

## 무중단 배포를 위한 확장-수축 전략

앱을 순차 배포하면 이전 버전과 새 버전이 잠시 함께 실행됩니다. 컬럼 이름 변경과
삭제를 한 배포에 끝내면 이전 앱이 즉시 실패할 수 있습니다. 이를 피하려면
확장(Expand)과 수축(Contract)을 여러 배포로 나눕니다.

```text
1. 새 컬럼을 nullable로 추가합니다.
2. 새 앱이 기존·새 컬럼을 함께 읽고 새 컬럼에도 기록합니다.
3. 배치로 과거 데이터를 채우고 검증합니다.
4. 모든 앱이 새 컬럼을 사용한 뒤 NOT NULL·제약을 추가합니다.
5. 충분한 관찰 기간 뒤 이전 컬럼과 호환 코드를 제거합니다.
```

대용량 backfill, 인덱스 생성, 제약 검증은 온라인 DDL 지원 여부와 락 범위를 DB
공식 문서로 확인합니다. Flyway가 SQL을 실행할 수 있다는 사실은 그 SQL이
무중단이라는 보장이 아닙니다. DB가 DDL 트랜잭션을 지원하는 범위도 다르므로,
실패 시에는 복구용 migration·백업·배포 중단 기준을 미리 정합니다.

## 운영 명령과 장애 대응

| 명령       | 목적                                                | 운영 주의점                                   |
| ---------- | --------------------------------------------------- | --------------------------------------------- |
| `migrate`  | 아직 적용되지 않은 변경을 적용합니다.               | 배포 전 백업·락·실행 시간을 확인합니다.       |
| `validate` | 파일과 적용 이력의 이름·버전·checksum을 확인합니다. | CI와 배포 전 단계에서 실패시킵니다.           |
| `info`     | 적용·대기 상태를 조회합니다.                        | 읽기 작업이지만 대상 URL을 반드시 확인합니다. |
| `repair`   | history 메타데이터를 정리합니다.                    | 실제 DB 상태를 고친 뒤에만 검토합니다.        |
| `baseline` | 기존 DB의 관리 기준점을 기록합니다.                 | 처음 도입할 올바른 DB에서만 실행합니다.       |
| `clean`    | schema 객체를 삭제합니다.                           | 운영에서는 비활성화하고 실행하지 않습니다.    |

CI에서는 빈 데이터베이스에 `migrate`를 실행하고, 애플리케이션 기동 또는
`validate`로 엔티티 매핑을 확인합니다. 변경이 기존 데이터와 함께 동작하는지는
운영과 유사한 데이터 규모의 스테이징 환경에서 별도 검증합니다. DB 접속 정보는
환경 변수·비밀 관리 도구로 주입하고 migration 파일이나 CI 로그에 넣지 않습니다.

## 실무 체크리스트

- 하나의 migration에는 검토 가능한 하나의 논리 변경을 넣습니다.
- 적용된 versioned migration은 수정하지 않고 다음 버전으로 진행합니다.
- SQL 문법, 트랜잭션 DDL, 온라인 인덱스 지원은 사용하는 DB 기준으로 검증합니다.
- 운영에서는 Flyway가 스키마 변경의 유일한 경로가 되도록 `ddl-auto`를
  `validate` 또는 `none`으로 제한합니다.
- 대용량 데이터 이관은 실행 시간, 잠금, 재실행 가능성, 백업·복구를 검토합니다.
- 무중단 배포에서는 이전 앱과 새 앱의 스키마 호환 기간을 설계합니다.
- checksum 불일치와 실패 migration은 history를 직접 고치지 말고 실제 DB 상태와
  배포 이력을 먼저 조사합니다.

## 참고 자료

- [Spring Boot: SQL Database Migration][spring-boot-flyway]

[spring-boot-flyway]: https://docs.spring.io/spring-boot/reference/data/sql.html
