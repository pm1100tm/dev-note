# Java와 Spring Boot

Java 언어의 기초, JVM, Spring Boot, JPA, 트랜잭션과 운영에 필요한 주제를 정리한다.
문서는 JDK 17 이상과 Spring Boot 3 계열을 기준으로 읽되, 실제 프로젝트의 버전과
공식 문서를 함께 확인한다.

## Java 기초

- [객체지향 프로그래밍의 특징](01_java/001_객체지향프로그래밍의특징.md)
- [변수와 메서드가 JVM에 저장되는 위치](01_java/002_변수와메소드가JVM에저장되는위치.md)
- [인터페이스](01_java/003_인터페이스.md)
- [인터페이스의 메서드](01_java/004_인터페이스에서의_메서드.md)
- [추상 클래스](01_java/005_추상클래스.md)
- [JVM](01_java/006_JVM.md)
- [String.toString과 String.valueOf](<01_java/etc_001_JAVA_Java String.toString() vs String.valueOf() 차이 완벽 정리.md>)
- [Long.valueOf와 Long.parseLong](<01_java/etc_002_JAVA_Long.valueOf() vs Long.parseLong() 차이 완벽 정리.md>)
- [SLF4J로 Spring 로그 작성하기](<01_java/etc_003_JAVA_Spring 로그 제대로 쓰기 — SLF4J의 올바른 사용법.md>)
- [Enum 비교에 `==` 사용하기](<01_java/etc_004_JAVA_Enum 비교에 == 연산자를 사용해야 하는 이유.md>)

## Spring과 데이터 접근

- [Java 환경 설정](001_Java_환경설정.md)
- [Spring Boot 프로젝트 만들기](002_SpringBoot_프로젝트_만들기.md)
- [H2 Database 실습](003_H2DB.md)
- [Spring 프로젝트 소개](02_spring/001_스프링_프로젝트_소개.md)
- [JDBC와 DataSource](02_spring/002_JDBC_DataSource.md)
- [트랜잭션](02_spring/003_트랜잭션.md)
- [선언적·명시적 트랜잭션](02_spring/004_트랜잭션_선언적_명시적.md)
- [Spring Boot 프로젝트 구동 과정](02_spring/005_Springboot_startup_process.md)
- [실행 JAR와 plain JAR](02_spring/006_SNAP_jar_plain_jar_차이점.md)
- [트랜잭션 아웃박스 패턴](etc_001_트랜잭션_아웃박스_패턴.md)

## JPA와 동시성

- [ORM과 JPA 기초 개념](<03_jpa/JPA_001_ORM과 JPA 기초 개념.md>)
- [기본 Entity Mapping](<03_jpa/JPA_002_기본 Entity Mapping.md>)
- [연관관계 매핑 기초](<03_jpa/JPA_003_연관관계 매핑 기초.md>)
- [Spring Data JPA와 JPQL](<03_jpa/JPA_004_Spring Data JPA와 JPQL.md>)
- [연관관계 고급 & N+1 문제](<03_jpa/JPA_004_연관관계 고급 & N+1 문제.md>)
- [Spring Data JPA 활용](<03_jpa/JPA_005_Spring Data JPA 활용.md>)
- [CQRS 패턴 적용](<03_jpa/JPA_006_CQRS 패턴 적용.md>)
- [트랜잭션과 동시성 제어](<03_jpa/JPA_007_트랜잭션과 동시성 제어.md>)
- [Flyway DB 형상 관리](<03_jpa/JPA_008_Flyway DB 형상 관리.md>)
- [JPA 완전 정복](03_jpa/JPA_009_완정정복.md)
- [Spring Data JPA와 JPQL의 차이](03_jpa/JPA_Q_001_DataJPA_JPQL.md)
- [@Modifying 완벽 정리](03_jpa/JPA_Q_002_Modifying.md)
- [낙관적 락의 처리 흐름](03_jpa/JPA_Q_003_낙관락의_흐름.md)
- [영속성 컨텍스트 상태 변이 과정](03_jpa/JPA_Q_004_영속성_컨텍스트_상태_변이_과정.md)
- [@Transient](03_jpa/JPA_105_entity-annotation-transient.md)
- [엔티티의 Detached 상태](<03_jpa/JPA_104_엔티티의 Detached 상태가 되는 경우.md>)
- [StaleObjectStateException](<03_jpa/JPA_103_StaleObjectStateException 이 발생한다면.md>)
- [@ElementCollection](03_jpa/JPA_101_ElementCollection.md)
- [JSONB 타입 설정](03_jpa/JPA_102_JSONB_타입_설정하기.md)
- [낙관락과 비관락](<02_spring/ING_20260402_01_Spring Boot 에서 낙관락, 비관락.md>)
- [낙관락 Best Practice](02*spring/ING_20260402_02*낙관락\_Best Practice.md)

## 운영과 품질

- [인터페이스 기반 Validator](validator/private_001_validator_with_interface.md)
- [IntelliJ Java 코드 컨벤션](etc_999_code_convention_설정.md)

## 학습 원칙

- 예제는 복사해 실행하기 전에 JDK, Spring Boot, Hibernate 버전 호환성을 확인한다.
- 트랜잭션 경계, 영속성 컨텍스트, 락은 테스트로 동작을 검증한 뒤 운영에 적용한다.
- 비밀값과 운영 설정은 코드나 문서 예시에 고정하지 않고 외부 설정으로 분리한다.
