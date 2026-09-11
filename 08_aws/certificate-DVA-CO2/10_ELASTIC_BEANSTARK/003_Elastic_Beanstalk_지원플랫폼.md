# 🚀 Elastic Beanstalk 지원 플랫폼

Elastic Beanstalk는 애플리케이션 코드를 AWS에 쉽게 배포할 수 있도록 여러 언어, 런타임,
Docker 환경을 지원합니다.

초보자 관점에서는 Elastic Beanstalk의 지원 플랫폼을 다음처럼 이해하면 됩니다.

```shell
내 애플리케이션이 어떤 언어와 런타임으로 만들어졌는지에 따라
Elastic Beanstalk 실행 환경을 선택합니다.
```

예를 들어 Spring Boot 애플리케이션이면 Java 계열 플랫폼을 선택하고, Express.js
애플리케이션이면 Node.js 플랫폼을 선택합니다.

<br>

## 1. 지원 플랫폼이란 무엇입니까?

Elastic Beanstalk에서 플랫폼은 애플리케이션을 실행하기 위한 기본 실행 환경입니다.

플랫폼에는 보통 다음 요소가 포함됩니다.

- 운영체제
- 언어 런타임
- 웹 서버 또는 애플리케이션 서버
- 배포 스크립트
- 로그 및 헬스 체크 기본 설정

즉, 개발자가 EC2에 직접 접속해서 Java, Node.js, Python, Tomcat, Nginx 같은 실행 환경을
하나씩 설치하지 않아도 되도록 Beanstalk가 미리 준비한 실행 템플릿입니다.

<br>

## 2. 슬라이드 기준 지원 플랫폼

| 플랫폼                  | 설명                                                          |
| ----------------------- | ------------------------------------------------------------- |
| Go                      | Go 언어 애플리케이션을 배포할 때 사용합니다.                  |
| Java SE                 | Java 애플리케이션을 직접 실행할 때 사용합니다.                |
| Java with Tomcat        | WAR 파일 또는 Tomcat 기반 Java 웹 애플리케이션에 사용합니다.  |
| .NET Core on Linux      | Linux 환경에서 .NET Core 애플리케이션을 실행할 때 사용합니다. |
| .NET on Windows Server  | Windows Server 기반 .NET 애플리케이션에 사용합니다.           |
| Node.js                 | Node.js 기반 웹 애플리케이션 또는 API 서버에 사용합니다.      |
| PHP                     | PHP 웹 애플리케이션에 사용합니다.                             |
| Python                  | Python 웹 애플리케이션 또는 API 서버에 사용합니다.            |
| Ruby                    | Ruby 애플리케이션에 사용합니다.                               |
| Packer Builder          | 사용자 정의 플랫폼 이미지를 만들 때 사용합니다.               |
| Single Container Docker | 하나의 Docker 컨테이너로 실행되는 애플리케이션에 사용합니다.  |
| Multi-container Docker  | 여러 컨테이너로 구성된 애플리케이션에 사용합니다.             |
| Preconfigured Docker    | 미리 구성된 Docker 환경을 기반으로 배포할 때 사용합니다.      |

<br>

## 3. 언어별 플랫폼 선택 예시

애플리케이션 기술 스택에 따라 플랫폼을 선택하면 됩니다.

| 애플리케이션 예시                   | 선택할 플랫폼           |
| ----------------------------------- | ----------------------- |
| Spring Boot JAR 애플리케이션        | Java SE                 |
| 전통적인 Java WAR 애플리케이션      | Java with Tomcat        |
| Express.js API 서버                 | Node.js                 |
| Django 또는 Flask 애플리케이션      | Python                  |
| Laravel 애플리케이션                | PHP                     |
| Ruby on Rails 애플리케이션          | Ruby                    |
| Go API 서버                         | Go                      |
| Dockerfile 기반 애플리케이션        | Single Container Docker |
| 여러 컨테이너가 필요한 애플리케이션 | Multi-container Docker  |

핵심은 애플리케이션이 어떤 방식으로 실행되는지입니다.

Spring Boot처럼 자체 내장 서버로 실행되는 JAR 파일이면 Java SE가 자연스럽습니다.

반대로 Tomcat에 WAR 파일을 배포하는 구조라면 Java with Tomcat이 더 적합합니다.

<br>

## 4. Java SE와 Java with Tomcat 차이

DVA-C02 준비 관점에서 Java 계열은 특히 구분해 두는 편이 좋습니다.

| 구분        | Java SE                                      | Java with Tomcat                            |
| ----------- | -------------------------------------------- | ------------------------------------------- |
| 실행 방식   | Java 애플리케이션을 직접 실행합니다.         | Tomcat 애플리케이션 서버 위에서 실행합니다. |
| 대표 패키지 | JAR                                          | WAR                                         |
| 대표 예시   | Spring Boot 실행 JAR                         | 전통적인 Servlet/JSP 웹 애플리케이션        |
| 서버 구조   | 애플리케이션이 내장 서버를 가질 수 있습니다. | Tomcat이 웹 애플리케이션을 구동합니다.      |

예를 들어 Spring Boot 애플리케이션을 `java -jar app.jar` 형태로 실행할 경우 Java SE 플랫폼을 고려합니다.

전통적인 Java 웹 애플리케이션을 `.war` 파일로 패키징해서 Tomcat에 올리는 구조라면 Java with Tomcat 플랫폼을 고려합니다.

<br>

## 5. Docker 플랫폼

Elastic Beanstalk는 Docker 기반 배포도 지원합니다.

슬라이드에는 다음 Docker 옵션이 나옵니다.

```shell
Single Container Docker
Multi-container Docker
Preconfigured Docker
```

<br>

### 5.1 Single Container Docker

Single Container Docker는 하나의 컨테이너로 애플리케이션을 실행하는 방식입니다.

예를 들어 백엔드 API 서버 하나만 Docker 이미지로 실행하는 경우에 적합합니다.

```shell
Docker Container
└── Spring Boot API
```

<br>

### 5.2 Multi-container Docker

Multi-container Docker는 여러 컨테이너를 함께 실행하는 방식입니다.

예를 들어 애플리케이션 서버와 Nginx를 함께 실행해야 하는 경우를 생각할 수 있습니다.

```shell
Container 1: Nginx
Container 2: Application Server
```

다만 운영 환경에서 복잡한 멀티 컨테이너 구조를 장기적으로 운영할 경우 ECS나 EKS가 더 적합할 수 있습니다.

DVA-C02에서는 Elastic Beanstalk도 Docker 배포를 지원함을 기억하면 충분합니다.

<br>

### 5.3 Preconfigured Docker

Preconfigured Docker는 미리 구성된 Docker 기반 환경을 사용하는 방식입니다.

시험에서는 세부 구현보다 "Elastic Beanstalk가 Docker 옵션도 지원합니다"라는 큰 흐름이 더 중요합니다.

<br>

## 6. Packer Builder

Packer Builder는 사용자 정의 플랫폼을 만들 때 사용합니다.

기본 제공 플랫폼으로 충분하지 않은 경우가 있습니다.

예를 들어:

- 특정 OS 패키지를 미리 설치해야 하는 경우
- 회사 표준 보안 설정이 적용된 이미지를 사용해야 하는 경우
- 기본 Beanstalk 플랫폼에 없는 런타임 구성이 필요한 경우

이런 경우 Packer Builder를 통해 커스텀 플랫폼을 만들 수 있습니다.

다만 DVA-C02에서는 깊은 사용법보다 "기본 플랫폼 외에 커스텀 플랫폼을 만들 수 있는 옵션이 있습니다"
정도로 이해하면 충분합니다.

<br>

## 7. 플랫폼 선택 시 확인할 것

Elastic Beanstalk 플랫폼을 선택할 때는 다음을 확인해야 합니다.

- 애플리케이션 언어가 무엇인지 확인합니다.
- 실행 파일 형태가 JAR, WAR, Docker 이미지, 소스 코드 중 무엇인지 확인합니다.
- 필요한 런타임 버전이 플랫폼에서 지원되는지 확인합니다.
- 환경 변수, 포트, 시작 명령이 올바르게 설정되어 있는지 확인합니다.
- 운영 환경에서는 플랫폼 버전 업데이트와 보안 패치 전략을 고려합니다.

플랫폼 선택을 잘못하면 배포는 성공해도 애플리케이션이 정상적으로 시작되지 않을 수 있습니다.

예를 들어 WAR 파일을 Java SE 플랫폼에 올리면 Tomcat이 없어서 기대한 방식으로 실행되지 않습니다.

반대로 `java -jar`로 실행되는 Spring Boot JAR 파일을 Tomcat 플랫폼에 올리면 패키징 방식과
실행 방식이 맞지 않을 수 있습니다.

<br>

## 8. DVA-C02 시험 포인트

- Elastic Beanstalk는 여러 언어와 런타임을 지원하는 애플리케이션 배포 서비스입니다.
- 지원 플랫폼에는 Go, Java SE, Java with Tomcat, .NET, Node.js, PHP, Python, Ruby,
  Docker 등이 포함됩니다.
- Java SE는 일반적으로 직접 실행되는 Java 애플리케이션에 적합합니다.
- Java with Tomcat은 Tomcat 기반 Java 웹 애플리케이션에 적합합니다.
- Docker 기반 애플리케이션도 Elastic Beanstalk에 배포할 수 있습니다.
- Single Container Docker와 Multi-container Docker를 구분해야 합니다.
- 플랫폼은 애플리케이션의 언어, 런타임, 패키징 방식에 맞게 선택해야 합니다.
- Elastic Beanstalk는 플랫폼을 제공하지만, 애플리케이션 코드와 의존성 파일은 개발자가 준비해야 합니다.

<br>

## 9. 시험 예상 문제

### 문제 1

개발자가 Spring Boot 애플리케이션을 `java -jar app.jar` 방식으로 실행하려고 합니다.
Elastic Beanstalk에서 가장 적절한 플랫폼은 무엇입니까?

A. Java SE
B. Java with Tomcat
C. PHP
D. Ruby

<br>

### 정답

A. Java SE

<br>

### 해설

Spring Boot 실행 JAR는 보통 내장 웹 서버를 포함하고 `java -jar` 방식으로 직접 실행됩니다.
이 경우 Tomcat에 WAR 파일을 배포하는 구조가 아니므로 Java SE 플랫폼이 더 적합합니다.
Java with Tomcat은 Tomcat 애플리케이션 서버 위에서 WAR 기반 웹 애플리케이션을 실행할 때 적합합니다.

<br>

### 문제 2

Elastic Beanstalk에서 Tomcat 기반 Java 웹 애플리케이션을 배포하려고 합니다.
가장 적절한 플랫폼은 무엇입니까?

A. Java SE
B. Java with Tomcat
C. Node.js
D. Single Container Docker

<br>

### 정답

B. Java with Tomcat

<br>

### 해설

Tomcat 기반 Java 웹 애플리케이션은 Tomcat 애플리케이션 서버가 필요합니다.
Elastic Beanstalk의 Java with Tomcat 플랫폼은 이런 WAR 기반 Java 웹 애플리케이션 배포에 적합합니다.
Java SE는 Java 애플리케이션을 직접 실행하는 형태에 더 적합합니다.

<br>

### 문제 3

다음 중 Elastic Beanstalk의 지원 플랫폼에 대한 설명으로 가장 올바른 것은 무엇입니까?

A. Elastic Beanstalk는 Java 애플리케이션만 지원합니다.
B. Elastic Beanstalk는 여러 언어와 Docker 기반 애플리케이션을 지원합니다.
C. Elastic Beanstalk는 Docker 기반 애플리케이션을 지원하지 않습니다.
D. Elastic Beanstalk는 애플리케이션 런타임과 무관하게 항상 같은 플랫폼을 사용합니다.

<br>

### 정답

B. Elastic Beanstalk는 여러 언어와 Docker 기반 애플리케이션을 지원합니다.

<br>

### 해설

슬라이드 기준 Elastic Beanstalk는 Go, Java, .NET, Node.js, PHP, Python, Ruby,
Docker 등 다양한 플랫폼을 지원합니다. 따라서 특정 언어 하나만 지원하는 서비스가 아닙니다.
애플리케이션의 언어, 런타임, 패키징 방식에 따라 적절한 플랫폼을 선택해야 합니다.

<br>

### 문제 4

개발팀이 하나의 Docker 컨테이너로 실행되는 API 서버를 Elastic Beanstalk에 배포하려고 합니다.
가장 적절한 플랫폼 유형은 무엇입니까?

A. Single Container Docker
B. Multi-container Docker
C. Java with Tomcat
D. Packer Builder

<br>

### 정답

A. Single Container Docker

<br>

### 해설

하나의 컨테이너로 애플리케이션을 실행하는 구조라면 Single Container Docker가 적합합니다.
여러 컨테이너를 함께 실행해야 할 경우 Multi-container Docker를 고려합니다.
Packer Builder는 사용자 정의 플랫폼을 만들 때 사용하는 옵션이며, 단일 Docker 컨테이너 실행
목적과는 다릅니다.

<br>

## 요약

- Elastic Beanstalk의 지원 플랫폼은 애플리케이션을 어떤 런타임에서 실행할지 정하는 기준입니다.
- Java, Node.js, Python, PHP, Ruby, Go, .NET 같은 언어별 플랫폼과 Docker 기반 플랫폼을
  지원합니다.
- DVA-C02에서는 플랫폼 목록을 단순 암기하기보다, 애플리케이션의 실행 방식에 맞는 플랫폼을 고르는 문제에
  대비해야 합니다.
