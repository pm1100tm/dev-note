# Springboot_project_startup_process - Springboot 프로젝트 구동 과정

Spring Boot 애플리케이션이 실행될 때 내부에서 어떤 일이 일어나는지 순서대로 정리합니다.

## 1. @SpringBootApplication 진입점

```java
@SpringBootApplication
public class MyBlogBackendApplication {
    public static void main(String[] args) {
        SpringApplication.run(MyBlogBackendApplication.class, args);
    }
}
```

- **@SpringBootApplication** 은 세 가지 어노테이션을 합친 메타 어노테이션입니다.
  - **@SpringBootConfiguration** → 사실상 @Configuration (설정 클래스임을 의미)
  - **@EnableAutoConfiguration** → 스프링 부트의 자동 설정(Auto Configuration) 기능을 활성화
  - **@ComponentScan** → 현재 패키지 기준 하위 패키지의 컴포넌트를 모두 스캔

➡️ 즉, 어플리케이션의 진입점 + 빈 등록 시작점이라고 이해할 수 있습니다.

## 2. SpringApplication.run() 실행

- 내부적으로 스트링 컨텍스트(ApplicationContext)를 생성합니다.
  - 기본적으로 AnnotationConfigServletWebServerApplicationContext
- 내장 톰켓(Tomcat) 같은 서블릿 컨테이너를 시작합니다. (Spring Boot Starter Web이 있을 때)
- 이후 빈 스캐닝 및 등록 과정이 진행됩니다.

## 3. 컴포넌트 스캔(@ComponentScan)

컴포넌트 스캔은 @SpringBootApplication 클래스가 위차한 패키지를 기준으로 하위 패키지 전체를
탐색합니다.

탐색 대상:

- @Controller
- @RestController
- @Service
- @Repository
- @Component
- @Configuration

➡️ 이 어노테이션들이 붙은 클래스는 **스프링 빈(Bean)**으로 등록됩니다.

## 4. BeanDefinition 생성 및 BeanFactory 등록

- 스프링은 우선 각 클래스의 메타 정보를 스캔하여, BeanDefiniiton 이라는 데이터 구조로 저장합니다.
  - 어떤 클래스인지
  - 어떤 스코프(singleton, prototype)인지
  - 어떤 의존성을 필요로 하는지
- 이 BeanDefinition 이 BeanFactory 에 등록됩니다.
- 아직 인스턴스 생성은 하지 않고, "등록만" 해둡니다.

## 5. 의존성 주입(Dependency Injection)

- BeanFactory 는 등록된 BeanDefinition 을 바탕으로 실제 객체 인서턴스를 생성합니다.
- 이 때, @Autowired, @Inject, @Resource 등이 작동하여 필요한 빈을 찾아 넣어줍니다.
- 생성자 주입, 세터 주입, 필드 주입 방식 모두 이 과정에서 실행됩니다.

➡️ 순서는 다음과 같습니다.

1. Bean 인스턴스 생성 (Reflection 사용)
2. 필요한 의존성 찾기 (타입 매칭 -> 이름 매칭)
3. 의존성 주입 (DI)
4. @PostContruct 실행 (초기화 로직)
5. 빈 초기화 완료

## 6. 애플리케이션 준비 완료

- 모든 빈이 생성되고, 의존성 주입과 초기화가 끝나면 ApplicationContext가 준비됩니다.
- 이 시점 이후 CommandLineRunner, ApplicationRunner 같은 후처리기가 실행됩니다.
- 웹 애플리케이션일 경우, 내장 톰캣이 HTTP 요청을 받을 준비를 완료합니다.

## 7. 요청 처리 (Spring MVC 기준)

1. 사용자가 HTTP 요청을 보냄
2. DispatcherServlet이 요청을 받아 HandlerMapping을 통해 적절한 컨트롤러를 찾음
3. 해당 컨트롤러 메서드 실행 → 서비스 호출 → 리포지토리 호출
4. 결과 반환 → 뷰 리졸버 / JSON 변환기에서 최종 응답 생성
5. 사용자에게 응답

## 정리

- @SpringBootApplication
- → Component Scan
- → BeanDefinition
- → Bean 생성 + 의존성 주입
- → ApplicationContext 준지
- → 내장 톰켓
- → 요청 처리 시작

```plain
[Start]
   │
   ▼
@SpringBootApplication
   │
   ▼
SpringApplication.run()
   │
   ▼
-------------------------
|   Component Scan      |
|  (@Component, @Service|
|   @Repository,        |
|   @Controller,        |
|   @Configuration)     |
-------------------------
   │
   ▼
BeanDefinition 등록
   │
   ▼
-------------------------
|   BeanFactory         |
|  (빈 생성 준비)       |
-------------------------
   │
   ▼
의존성 주입 (DI)
   │   ┌─────────────────────────────┐
   │   │ @Autowired / @Inject        │
   │   │ 생성자/세터/필드 주입       │
   │   └─────────────────────────────┘
   ▼
@PostConstruct 실행
   │
   ▼
-------------------------
| ApplicationContext    |
|   (모든 빈 준비 완료) |
-------------------------
   │
   ▼
내장 톰캣(DispatcherServlet) 실행
   │
   ▼
HTTP 요청 수신
   │
   ▼
-------------------------
| DispatcherServlet     |
|  -> HandlerMapping    |
|  -> Controller 호출   |
-------------------------
   │
   ▼
비즈니스 로직 실행 (@Service)
   │
   ▼
데이터 접근 (@Repository)
   │
   ▼
응답 반환 (ViewResolver/JSON)
   │
   ▼
[Response to Client]
```

---

## Extra

### 각 어노테이션의 역할

- @Configuration
  - 스프링 설정 클래스
- @Bean 메서드를 통해 수동으로 빈을 등록할 수 있음
  - 애플리케이션 초기 설정을 담당
- @Component
  - 가장 일반적인 빈 등록용 어노테이션
  - 특별한 의미 없이 단순히 “스프링이 관리하는 객체”를 만듦
- @Service
  - 비즈니스 로직을 담당하는 빈
  - @Component와 기능적으로 동일하지만, 의미적 구분을 위해 사용
- @Repository
  - 데이터 접근 계층(DAO)
  - DB 예외를 스프링의 데이터 접근 예외로 변환하는 기능 제공
- @Controller
  - MVC 패턴의 Controller
  - DispatcherServlet이 라우팅하여 HTTP 요청을 이 빈에게 위임
- @RestController / @Controller + @ResponseBody
  - JSON/XML 형태의 응답을 바로 반환하는 API용 컨트롤러

```

```
