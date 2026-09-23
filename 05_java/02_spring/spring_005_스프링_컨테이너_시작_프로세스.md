# Spring Boot 애플리케이션 구동 과정

Spring Boot 애플리케이션이 시작될 때 `main` 메서드에서 `ApplicationContext`가 준비되고
HTTP 요청을 받기까지의 흐름을 정리합니다.

## 목차

- [1. 애플리케이션 진입점](#1-애플리케이션-진입점)
- [2. SpringApplication.run() 실행](#2-springapplicationrun-실행)
- [3. 환경과 ApplicationContext 준비](#3-환경과-applicationcontext-준비)
- [4. 자동 설정과 컴포넌트 스캔](#4-자동-설정과-컴포넌트-스캔)
- [5. BeanDefinition 등록](#5-beandefinition-등록)
- [6. 빈 생성과 의존성 주입](#6-빈-생성과-의존성-주입)
- [7. 웹 서버와 애플리케이션 준비 완료](#7-웹-서버와-애플리케이션-준비-완료)
- [8. HTTP 요청 처리](#8-http-요청-처리)
- [9. 전체 흐름 요약](#9-전체-흐름-요약)

## 1. 애플리케이션 진입점

Spring Boot 애플리케이션은 보통 `@SpringBootApplication`이 붙은 클래스의 `main` 메서드에서
시작합니다.

```java
@SpringBootApplication
public class MyBlogBackendApplication {

    public static void main(String[] args) {
        SpringApplication.run(
            MyBlogBackendApplication.class, args);
    }
}
```

`@SpringBootApplication`은 다음 세 가지 역할을 한 곳에 모은 메타 어노테이션입니다.

- `@SpringBootConfiguration`: 애플리케이션 설정 클래스임을 나타냅니다.
- `@EnableAutoConfiguration`: 클래스패스와 설정에 맞는 자동 설정을 적용합니다.
- `@ComponentScan`: 현재 패키지와 하위 패키지에서 컴포넌트를 탐색합니다.

따라서 이 클래스의 패키지 위치는 중요합니다. 하위 패키지 밖에 있는 컴포넌트는 기본 스캔 대상에서 제외될 수
있습니다.

## 2. SpringApplication.run() 실행

`SpringApplication.run()`은 단순히 객체 하나를 생성하는 메서드가 아닙니다. 애플리케이션 실행에
필요한 환경을 구성하고, 스프링 컨테이너를 초기화한 뒤, 웹 애플리케이션이면 내장 서버까지 시작합니다.

주요 작업은 다음 순서로 진행됩니다.

- 실행 환경과 애플리케이션 타입을 확인합니다.
- `Environment`와 설정 정보를 준비합니다.
- 애플리케이션 타입에 맞는 `ApplicationContext`를 생성합니다.
- 자동 설정과 컴포넌트 스캔을 적용합니다.
- Bean을 생성하고 초기화합니다.
- 웹 서버와 애플리케이션 이벤트 처리기를 시작합니다.

웹 의존성이 있으면 일반적으로 서블릿 기반의
`AnnotationConfigServletWebServerApplicationContext`가 사용됩니다.

## 3. 환경과 ApplicationContext 준비

### Environment 구성

Spring Boot는 실행 인자, 환경 변수, 프로파일, `application.yml` 또는 `application.properties`를
읽어 `Environment`를 구성합니다.

- 같은 키가 여러 곳에 정의되면 우선순위가 높은 설정이 적용됩니다.
- 예를 들어 운영 환경에서는 환경 변수나 실행 인자로 데이터베이스 접속 정보를 주입하고,
  개발 환경에서는 `application-local.yml`을 사용할 수 있습니다.

### ApplicationContext 생성

`ApplicationContext`는 빈을 보관하고 생성하며, 빈 사이의 의존성을 연결하는 스프링의 핵심
컨테이너입니다. 웹 애플리케이션에서는 여기에 웹 서버와 `DispatcherServlet`에 필요한 설정도
함께 연결됩니다.

## 4. 자동 설정과 컴포넌트 스캔

### 자동 설정

`@EnableAutoConfiguration`은 클래스패스에 있는 라이브러리와 설정 조건을 확인한 뒤 필요한 Bean을
자동으로 등록합니다.

예를 들어 `spring-boot-starter-web`이 있으면 다음과 같은 웹 관련 설정이 조건에 따라 적용됩니다.

- 내장 Tomcat과 서블릿 웹 애플리케이션 설정
- `DispatcherServlet` 등록
- JSON 변환을 위한 HTTP 메시지 컨버터

자동 설정은 무조건 모든 빈을 등록하는 것이 아니라, 특정 클래스나 프로퍼티, 사용자 정의 빈이 있는지를
조건으로 판단합니다. 따라서 같은 역할의 빈을 직접 등록하면 자동 설정이 대체되거나 충돌할 수 있습니다.

### 컴포넌트 스캔

`@ComponentScan`은 기준 패키지부터 하위 패키지를 탐색하고 다음과 같은 어노테이션이 붙은 클래스를
Bean 후보로 수집합니다.

- `@Component`
- `@Service`
- `@Repository`
- `@Controller`
- `@RestController`
- `@Configuration`

## 5. BeanDefinition 등록

스프링은 스캔한 클래스를 바로 객체로 만들지 않고 먼저 `BeanDefinition`으로 등록합니다.
`BeanDefinition`에는 다음과 같은 빈의 메타데이터가 담깁니다.

- 빈의 클래스와 이름
- 싱글턴, 프로토타입 같은 스코프
- 생성자와 주입할 의존성
- 초기화 및 소멸 메서드
- 지연 생성 여부

이 정보가 `BeanFactory`와 `ApplicationContext`에 등록되면 스프링은 어떤 빈을 언제, 어떤 방식으로
생성할지 판단할 수 있습니다.

## 6. 빈 생성과 의존성 주입

컨테이너는 필요한 빈을 실제 객체로 만들고 의존성을 주입합니다.
일반적인 생성 흐름은 다음과 같습니다.

- 생성자를 호출해 객체를 생성합니다.
- 생성자나 설정 메서드에 필요한 의존성을 연결합니다.
- `BeanPostProcessor`가 Bean 생성 전후의 처리를 수행합니다.
- `@PostConstruct` 같은 초기화 메서드를 호출합니다.
- 초기화가 끝난 Bean을 컨테이너에 보관합니다.

생성자 주입은 객체 생성 시점에 필요한 의존성을 확인할 수 있어 일반적으로 권장됩니다.

`@Transactional`이나 `@Async`처럼 프록시가 필요한 기능은 Bean 초기화 과정에서 프록시 객체로
감싸질 수 있습니다.

## 7. 웹 서버와 애플리케이션 준비 완료

웹 애플리케이션이면 내장 서버가 포트를 열고 요청을 받을 준비를 합니다. 모든 싱글턴 Bean 생성과 초기화가
끝나면 `ApplicationReadyEvent`가 발생하고, `CommandLineRunner`와 `ApplicationRunner`가
실행됩니다.

이 시점부터 애플리케이션은 정상적으로 요청을 처리할 수 있습니다. 초기화 실패로 예외가 발생하면
`ApplicationContext`가 준비되지 못하고 애플리케이션 시작 자체가 실패할 수 있습니다.

## 8. HTTP 요청 처리

애플리케이션이 시작된 뒤 HTTP 요청이 들어오면 일반적으로 다음 흐름을 탑니다.

- 내장 서버가 요청을 받아 `DispatcherServlet`으로 전달합니다.
- `HandlerMapping`이 URL과 HTTP 메서드에 맞는 컨트롤러를 찾습니다.
- `HandlerAdapter`가 컨트롤러 메서드를 호출합니다.
- 컨트롤러가 서비스와 리포지토리를 호출해 비즈니스 로직을 수행합니다.
- `HttpMessageConverter`가 반환 객체를 JSON 등 응답 형식으로 변환합니다.
- 서버가 클라이언트에 HTTP 응답을 반환합니다.

## 9. 전체 흐름 요약

```shell
main()
  ↓
SpringApplication.run()
  ↓
Environment 및 ApplicationContext 준비
  ↓
자동 설정 + 컴포넌트 스캔
  ↓
BeanDefinition 등록
  ↓
빈 생성 + 의존성 주입 + 초기화
  ↓
내장 웹 서버 시작
  ↓
ApplicationReadyEvent 및 Runner 실행
  ↓
HTTP 요청 수신
  ↓
DispatcherServlet → Controller → Service → Repository
  ↓
응답 변환 및 반환
```

## 주요 어노테이션의 역할

| 어노테이션               | 역할                                            |
| ------------------------ | ----------------------------------------------- |
| `@SpringBootApplication` | 설정, 자동 설정, 컴포넌트 스캔을 활성화합니다.  |
| `@Configuration`         | 자바 기반 설정 클래스를 나타냅니다.             |
| `@Bean`                  | 메서드의 반환 객체를 스프링 빈으로 등록합니다.  |
| `@Component`             | 일반적인 스프링 관리 객체를 등록합니다.         |
| `@Service`               | 서비스 계층의 빈임을 표현합니다.                |
| `@Repository`            | 데이터 접근 계층의 빈임을 표현합니다.           |
| `@Controller`            | 뷰를 반환하는 MVC 컨트롤러를 등록합니다.        |
| `@RestController`        | 응답 본문을 반환하는 API 컨트롤러를 등록합니다. |

어노테이션의 역할을 이해하면 애플리케이션 시작 중 발생하는 빈 등록 실패, 컴포넌트 스캔 누락, 순환 참조와
같은 문제를 원인별로 확인하기 쉬워집니다.

## 요약

- 1단계: 애플리케이션 진입점
  - `@SpringBootApplication`이 붙은 클래스의 `main` 메서드에서 애플리케이션이 시작됩니다.
  - 이 어노테이션은 설정 클래스 지정, 자동 설정 활성화, 컴포넌트 스캔을 함께 적용합니다.

- 2단계: `SpringApplication.run()` 실행
  - `main` 메서드가 `SpringApplication.run()`을 호출하면 Spring Boot가 실행 과정을 시작합니다.

- 3단계: 실행 환경 준비
  - 실행 인자, 환경 변수, 프로파일, 설정 파일을 읽어 `Environment`를 구성합니다.

- 4단계: `ApplicationContext` 생성
  - 애플리케이션 유형에 맞는 컨테이너를 만들고, 빈과 애플리케이션 설정을 관리할 준비를 합니다.

- 5단계: 자동 설정과 컴포넌트 스캔
  - 클래스패스와 설정 조건에 맞는 자동 설정을 적용하고, `@Component` 계열 클래스를 빈 후보로 찾습니다.

- 6단계: `BeanDefinition` 등록
  - 빈 후보의 클래스, 스코프, 의존성 같은 메타데이터를 `BeanDefinition`으로 등록합니다.

- 7단계: 빈 생성과 의존성 주입
  - 등록된 빈 정의를 바탕으로 객체를 만들고, 필요한 의존성을 주입한 뒤 초기화합니다.

- 8단계: Spring Boot 컨테이너 준비 완료
  - 모든 필수 빈이 초기화되면 `ApplicationContext`가 준비되어 Spring Boot 컨테이너를 사용할 수 있습니다.

- 9단계: 내장 웹 서버 시작
  - 웹 애플리케이션이면 내장 웹 서버와 `DispatcherServlet`을 시작해 HTTP 요청을 처리할 준비를 합니다.

즉, Spring Boot는 실행 환경과 빈 정의를 준비한 뒤 `ApplicationContext` 안에서 빈을 생성하고
연결합니다. 웹 애플리케이션은 내장 웹 서버까지 시작되면 외부 요청을 처리할 수 있습니다.
