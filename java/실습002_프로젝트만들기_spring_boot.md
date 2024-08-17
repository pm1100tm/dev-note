# 프로젝트 만들기 - Spring Boot

## 프로젝트 생성 - IntelliJ Ultimate 버전 기준(하단에 Comunity 버전 기준 생성 방법 기재함)

- File -> New -> Project -> Generator(Spring Boot)
- Java -> Gradle(Grooby) -> temurin17 -> Jar
- Lombok, Spring Web

## Annotation 설정

-> Enable annotation processing 체크

![annotation-processors](./assets/annotation-processors.png)

> 📚 Lombok 과 같은 외부 어노테이션 라이브러리가 컴파일 시 문제없이 동작할 수 있도록 함

### What is Lombok

Lombok은 자바 개발에서 반복적으로 작성해야 하는 보일러플레이트 코드를 줄여주는 라이브러리입니다.
이것을 사용하면 자바 클래스에서 필수적으로 작성해야 하는 getter, setter, toString, equlas,
hashCode 등의 메서드를 자동으로 생성해줍니다. 이는 코드의 가독성을 높이고 유지보수를 용이하게 합니다.

#### Lombok 사용법

```java
// 클래스의 모든 필드에 대해 getter와 setter 메서드를 자동으로 생성합니다.
import lombok.Getter;
import lombok.Setter;

@Getter
@Setter
public class Person {
    private String name;
    private int age;
}

// 클래스의 모든 필드를 포함하는 toString 메서드를 자동으로 생성합니다.
import lombok.ToString;

@ToString
public class Person {
    private String name;
    private int age;
}

// equals와 hashCode 메서드를 자동으로 생성합니다.
import lombok.EqualsAndHashCode;

@EqualsAndHashCode
public class Person {
    private String name;
    private int age;
}

// 각각 파라미터가 없는 생성자, 모든 필드를 파라미터로 받는 생성자, final 또는 @NonNull 필드만
// 파라미터로 받는 생성자를 자동으로 생성합니다.
import lombok.NoArgsConstructor;
import lombok.AllArgsConstructor;

@NoArgsConstructor
@AllArgsConstructor
public class Person {
    private String name;
    private int age;
}

// @Data 어노테이션은 @Getter, @Setter, @ToString, @EqualsAndHashCode,
// @RequiredArgsConstructor를 포함하여 모든 보일러플레이트 코드를 자동으로 생성합니다.
import lombok.Data;

@Data
public class Person {
    private String name;
    private int age;
}
```

### 단축키 Tips

- cmd + shift + r: 문구 검색
- shift + shift: 파일, 클래스 검색
- cmd + e: 최근에 검색한 파일 검색
- ctl + option + o: import 정리
- cmd + option + l: 코드 포맷 맞추기
- option + enter: 라이브러리 불러오기
- cmd + option + m: 코드 선택 후 단축키 사용하면 자동으로 메서드 분리

--

## 프로젝트 생성 - IntelliJ Comunity 기준

- start.spring.io 에서 제공하는 initializer 활용합니다.
- 세팅 후 다운로드
- Add dependency

프로젝트가 성공적으로 import 되었다면, IntelliJ 우측 탭에 Gradle 또는 Maven이 생성되어야 합니다.
또한, 커뮤니티 버전의 IntelliJ는 스프링부트 프로젝트를 인식할 수 없어, 실행 설정을 해주어야 합니다.

- 우측 상단 Add Configuration 클릭
- Run/Debug Configurations 화면
- 우측 상단 + 버튼 -> Application 클릭
  - Name
  - 모듈 선택: ex) demo.main
  - 메인 클래스 선택

## Code Style 적용

### 네이버 코딩 컨텐션 적용하기

- naver-intellij-formatter.xml 다운로드
  - [hackday-conventions-java](https://github.com/naver/hackday-conventions-java/tree/master/rule-config)
  - rule-config -> naver-intellij-formatter.xml 다운로드
- File -> Setting -> Editor -> Code Style > Java -> Scheme 항목의 오른쪽 톱니바퀴
  -> Import Scheme -> IntelliJ IDEA Code Style XML 에서 다운받은 formmter 적용
- 그 후, Command + Option + L 로 지정한 코드 스타일에 맞게 자동으로 코드가 수정되는 것 확인
- 파일의 마지막 줄에 빈줄이 반드시 생성되도록 설정
  - IntelliJ IDEA > Setting > Editor > General ->
    Ensure every saved file ends with a line break 옵션 체크
- 코드 저장시 자동으로 Formatter가 동작하도록 설정
  - IntelliJ IDEA -> Setting -> Tools > Actions on Save
    - Reformat code, Optimize imports 체크하고 OK 선택하여 설정 적용

### Gradle에서 Checkstyle 플러그인 설정

- naver-checkstyle-rules.xml, naver-checkstyle-suppressions.xml 다운로드
- 위에서 다운받은 파일을 프로젝트 루트 폴더에 위치
- build.gradle 파일에 아래와 같이 Java Plugin 속성으로 인코딩 지정 및 checkstyle 추가

```plain
plugins {
    id 'java'
    id 'org.springframework.boot' version '3.3.2'
    id 'io.spring.dependency-management' version '1.1.6'
    id 'checkstyle' // 추가
}

compileJava.options.encoding = 'UTF-8' // 추가
compileTestJava.options.encoding = 'UTF-8' // 추가

// 추가
checkstyle {
    maxWarnings = 0 // 규칙이 어긋나는 코드가 하나라도 있을 경우 빌드 fail 을 내고 싶다면 이 선언을 추가한다.
    configFile = file("${rootDir}/naver-checkstyle-rules.xml")
    configProperties = ["suppressionFile": "${rootDir}/naver-checkstyle-suppressions.xml"]
    toolVersion = "8.24"  // checkstyle 버전 8.24 이상 선언
}
```

- ![code-style-in-gradle](./assets/code-style-in-gradle.png)
