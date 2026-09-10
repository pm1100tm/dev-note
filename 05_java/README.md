# Java와 Spring Boot

Java 언어의 기초, JVM, Spring Boot, JPA, 트랜잭션과 운영에 필요한 주제를 정리한다.
문서는 JDK 17 이상과 Spring Boot 3 계열을 기준으로 읽되, 실제 프로젝트의 버전과
공식 문서를 함께 확인한다.

## Java 기초

- [객체지향 프로그래밍의 특징](기초001_객체지향프로그래밍의특징.md)
- [변수와 메서드가 JVM에 저장되는 위치](기초002_변수와메소드가JVM에저장되는위치.md)
- [인터페이스](기초003_인터페이스.md)
- [인터페이스의 메서드](기초004_인터페이스에서의_메서드.md)
- [추상 클래스](기초005_추상클래스.md)
- [JVM](기초006_JVM.md)
- [String.toString과 String.valueOf](<java/20260402_01_Java String.toString() vs
- String.valueOf() 차이 완벽 정리.md>)
- [Long.valueOf와 Long.parseLong](<java/20260402_02_Java Long.valueOf() vs Long.parseLong() 차이
- 완벽 정리.md>)

## Spring과 데이터 접근

- [Java 환경 설정](실습001_Java_환경설정.md)
- [Spring Boot 프로젝트 만들기](실습002_프로젝트만들기_spring_boot.md)
- [H2 Database 실습](실습003_H2DB.md)
- [Spring 프로젝트 소개](Theory_001_스프링_프로젝트_소개.md)
- [JDBC와 DataSource](Theory_002_JDBC_DataSource.md)
- [트랜잭션](Theory_003_트랜잭션.md)
- [선언적·명시적 트랜잭션](Theory_004_트랜잭션_선언적_명시적.md)
- [Spring Boot 프로젝트 구동 과정](Theory_005_Springboot_project_startup_process.md)
- [실행 JAR와 plain JAR](Theory_006_SNAP_jar_plain_jar_차이점.md)

## JPA와 동시성

- [@Transient](jpa/entity-annotation-transient.md)
- [엔티티의 Detached 상태](<jpa/JPA_엔티티의 Detached 상태가 되는 경우.md>)
- [StaleObjectStateException](<jpa/JPA_StaleObjectStateException 이 발생한다면.md>)
- [@ElementCollection](jpa/JPA_ElementCollection.md)
- [JSONB 타입 설정](jpa/JPA_JSONB_타입_설정하기.md)
- [낙관락과 비관락](<springboot/20260402_01_Spring Boot 에서 낙관락, 비관락.md>)
- [낙관락 Best Practice](springboot/20260402*02*낙관락\_Best Practice.md)

## 운영과 품질

- [SLF4J로 Spring 로그 작성하기](<etc/001_Spring 로그 제대로 쓰기 — SLF4J의 올바른 사용법.md>)
- [Enum 비교에 `==` 사용하기](<etc/002_Java Enum 비교에 == 연산자를 사용해야 하는 이유.md>)
- [인터페이스 기반 Validator](validator/001_validator_with_interface.md)
- [로컬 Jenkins 연동](<jenkins/jenkins001_로컬에서 jenkins 연동하기.md>)
- [IntelliJ Java 코드 컨벤션](TODO_code_convention.md)

## 학습 원칙

- 예제는 복사해 실행하기 전에 JDK, Spring Boot, Hibernate 버전 호환성을 확인한다.
- 트랜잭션 경계, 영속성 컨텍스트, 락은 테스트로 동작을 검증한 뒤 운영에 적용한다.
- 비밀값과 운영 설정은 코드나 문서 예시에 고정하지 않고 외부 설정으로 분리한다.
