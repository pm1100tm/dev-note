# Java

Java 언어의 기초, JVM, Spring Boot, JPA, 트랜잭션과 운영에 필요한 주제를 정리한다. 문서는 JDK 17 이상과 Spring Boot 3 계열을 기준으로 읽되, 실제 프로젝트의 버전과 공식 문서를 함께 확인한다.

## Java 기초

* [객체지향 프로그래밍의 특징](001_.md)
* [변수와 메서드가 JVM에 저장되는 위치](002_-jvm.md)
* [인터페이스](003_.md)
* [인터페이스의 메서드](004_-_.md)
* [추상 클래스](005_.md)
* [JVM](006_jvm.md)
* \[String.toString과 String.valueOf]\(\<java/20260402\_01\_Java String.toString() vs
* String.valueOf() 차이 완벽 정리.md>)
* \[Long.valueOf와 Long.parseLong]\(\<java/20260402\_02\_Java Long.valueOf() vs Long.parseLong() 차이
* 완벽 정리.md>)

## Spring과 데이터 접근

* [Java 환경 설정](001_java_.md)
* [Spring Boot 프로젝트 만들기](002_-_spring_boot.md)
* [H2 Database 실습](003_h2db.md)
* [Spring 프로젝트 소개](theory_001_-_-_.md)
* [JDBC와 DataSource](theory_002_jdbc_datasource.md)
* [트랜잭션](theory_003_.md)
* [선언적·명시적 트랜잭션](theory_004_-_-_.md)
* [Spring Boot 프로젝트 구동 과정](theory_005_springboot_project_startup_process.md)
* [실행 JAR와 plain JAR](theory_006_snap_jar_plain_jar_.md)

## JPA와 동시성

* [@Transient](entity-annotation-transient.md)
* [엔티티의 Detached 상태](jpa_-detached.md)
* [StaleObjectStateException](jpa_staleobjectstateexception.md)
* [@ElementCollection](jpa_elementcollection.md)
* [JSONB 타입 설정](jpa_jsonb_-_.md)
* [낙관락과 비관락](20260402_01_spring-boot.md)
* \[낙관락 Best Practice]\(springboot/2026040&#x32;_&#x30;&#x32;_&#xB099;관락\_Best Practice.md)

## 운영과 품질

* [SLF4J로 Spring 로그 작성하기](001_spring-slf4j.md)
* [Enum 비교에 `==` 사용하기](002_java-enum.md)
* [인터페이스 기반 Validator](../../05_java/validator/001_validator_with_interface.md)
* [로컬 Jenkins 연동](jenkins001_-jenkins.md)
* [IntelliJ Java 코드 컨벤션](todo_code_convention.md)

## 학습 원칙

* 예제는 복사해 실행하기 전에 JDK, Spring Boot, Hibernate 버전 호환성을 확인한다.
* 트랜잭션 경계, 영속성 컨텍스트, 락은 테스트로 동작을 검증한 뒤 운영에 적용한다.
* 비밀값과 운영 설정은 코드나 문서 예시에 고정하지 않고 외부 설정으로 분리한다.
