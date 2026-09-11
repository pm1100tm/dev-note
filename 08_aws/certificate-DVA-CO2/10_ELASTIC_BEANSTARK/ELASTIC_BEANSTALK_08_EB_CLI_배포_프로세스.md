# 🚀 EB CLI와 Elastic Beanstalk 배포 프로세스

Elastic Beanstalk는 AWS 콘솔에서도 사용할 수 있지만, EB CLI를 사용하면 터미널에서 환경 생성,
상태 확인, 로그 확인, 배포를 쉽게 수행할 수 있습니다.

초보자 관점에서는 EB CLI를 다음처럼 이해하면 됩니다.

```shell
EB CLI = Elastic Beanstalk를 터미널에서 다루기 쉽게 해주는 명령어 도구
```

특히 CI/CD 파이프라인에서 자동 배포를 구성할 때 유용합니다.

<br>

## 1. EB CLI란 무엇입니까?

EB CLI는 Elastic Beanstalk 작업을 명령어로 수행할 수 있게 해주는 별도 CLI 도구입니다.

AWS CLI가 AWS 전체 서비스를 다루는 범용 도구라면, EB CLI는 Elastic Beanstalk 작업에 특화된
도구입니다.

예를 들어 콘솔에서 클릭으로 하던 작업을 아래처럼 명령어로 처리할 수 있습니다.

```shell
eb create
eb status
eb health
eb logs
eb deploy
```

개발자 입장에서는 로컬 개발 환경이나 배포 파이프라인에서 Beanstalk 배포를 반복 실행하기 쉬워집니다.

<br>

## 2. EB CLI 기본 명령어

자주 사용하는 EB CLI 명령어는 다음과 같습니다.

| 명령어         | 용도                                           |
| -------------- | ---------------------------------------------- |
| `eb create`    | 새 Elastic Beanstalk Environment를 생성합니다. |
| `eb status`    | 현재 Environment 상태를 확인합니다.            |
| `eb health`    | 애플리케이션 헬스 상태를 확인합니다.           |
| `eb events`    | Environment 이벤트 로그를 확인합니다.          |
| `eb logs`      | 애플리케이션 또는 인스턴스 로그를 가져옵니다.  |
| `eb open`      | 배포된 애플리케이션 URL을 브라우저로 엽니다.   |
| `eb deploy`    | 새 애플리케이션 버전을 배포합니다.             |
| `eb config`    | Environment 설정을 확인하거나 수정합니다.      |
| `eb terminate` | Environment를 종료합니다.                      |

<br>

## 3. 기본 배포 흐름

Elastic Beanstalk 배포 프로세스는 크게 아래 흐름으로 이해하면 됩니다.

```shell
1. 의존성 파일을 준비합니다.
2. 애플리케이션 코드를 zip으로 패키징합니다.
3. Elastic Beanstalk에 업로드합니다.
4. 새로운 Application Version이 생성됩니다.
5. Environment에 새 Version을 배포합니다.
6. 각 EC2 인스턴스에서 의존성을 설치하고 애플리케이션을 시작합니다.
```

즉, Beanstalk는 단순히 파일을 저장하는 서비스가 아니라, 업로드된 코드 묶음을 각 EC2 인스턴스에
배포하고 실행까지 연결합니다.

<br>

## 4. 의존성 파일 준비

Elastic Beanstalk는 애플리케이션의 의존성을 보고 실행 환경에서 필요한 패키지를 설치합니다.

언어별 대표 의존성 파일은 다음과 같습니다.

| 언어 / 플랫폼 | 대표 의존성 파일                          |
| ------------- | ----------------------------------------- |
| Python        | `requirements.txt`                        |
| Node.js       | `package.json`                            |
| Java          | `pom.xml`, `build.gradle`, JAR/WAR 패키지 |
| PHP           | `composer.json`                           |
| Ruby          | `Gemfile`                                 |

자료에서는 특히 Python과 Node.js 예시를 강조합니다.

```shell
Python  -> requirements.txt
Node.js -> package.json
```

예를 들어 Python 애플리케이션에서 `requirements.txt`가 없거나 잘못 작성되면 필요한 패키지가
설치되지 않아 애플리케이션 시작에 실패할 수 있습니다.

<br>

## 5. zip 패키징

- Elastic Beanstalk에 배포할 코드는 zip 파일로 패키징할 수 있습니다.
- zip 파일에는 애플리케이션 코드와 의존성 설명 파일이 포함되어야 합니다.
- 예를 들어 Node.js 애플리케이션이면 다음 파일들이 포함될 수 있습니다.

```shell
app.js
package.json
package-lock.json
Procfile
```

Python 애플리케이션이면 다음 파일들이 포함될 수 있습니다.

```shell
application.py
requirements.txt
Procfile
```

중요한 점은 zip 안의 루트 구조입니다.

애플리케이션 파일이 불필요한 상위 폴더 안에 한 번 더 들어가면 Beanstalk가 기대한 파일을 찾지 못할 수 있습니다.

```shell
좋은 예:
app.js
package.json

나쁜 예:
my-app-folder/
└── app.js
└── package.json
```

<br>

## 6. 콘솔 배포 방식

AWS 콘솔을 사용하는 배포 흐름은 다음과 같습니다.

```shell
1. 애플리케이션 코드를 zip 파일로 준비합니다.
2. Elastic Beanstalk 콘솔에서 zip 파일을 업로드합니다.
3. 업로드된 zip 파일로 새 Application Version이 생성됩니다.
4. 해당 Version을 Environment에 배포합니다.
```

콘솔 방식은 처음 학습하거나 수동으로 테스트할 때 이해하기 쉽습니다.

다만 반복 배포나 자동화에는 EB CLI 또는 CI/CD 파이프라인을 사용하는 편이 더 적합합니다.

<br>

## 7. EB CLI 배포 방식

EB CLI를 사용하면 명령어로 새 Application Version을 만들고 배포할 수 있습니다.

대표 흐름은 다음과 같습니다.

```shell
eb create
eb status
eb deploy
eb health
eb logs
```

예시 흐름은 다음과 같습니다.

```shell
# 최초 환경 생성
eb create my-env

# 현재 상태 확인
eb status

# 새 코드 배포
eb deploy

# 헬스 상태 확인
eb health

# 로그 확인
eb logs
```

`eb deploy`를 실행하면 EB CLI가 애플리케이션 코드를 패키징하고 Elastic Beanstalk에 업로드한 뒤
새 버전으로 배포합니다.

<br>

## 8. EC2 인스턴스에서 일어나는 일

Elastic Beanstalk가 zip 파일을 배포하면 각 EC2 인스턴스에서는 대략 다음 작업이 일어납니다.

```shell
zip 파일 다운로드
  ↓
애플리케이션 코드 압축 해제
  ↓
의존성 설치
  ↓
애플리케이션 시작
  ↓
헬스 체크 수행
```

- 예를 들어 Node.js 애플리케이션이면 `package.json`을 기준으로 의존성을 설치합니다.
- Python 애플리케이션이면 `requirements.txt`를 기준으로 필요한 패키지를 설치합니다.
- 애플리케이션이 정상적으로 시작되지 않으면 Environment Health가 나빠지고, `eb logs`나
  `eb events`로 원인을 확인해야 합니다.

<br>

## 9. CI/CD 파이프라인에서 EB CLI가 유용한 이유

EB CLI는 자동 배포 파이프라인에서 유용합니다.

예를 들어 GitHub Actions, Jenkins, CodeBuild 같은 도구에서 아래 흐름을 구성할 수 있습니다.

```shell
소스 코드 변경
  ↓
테스트 실행
  ↓
빌드 또는 패키징
  ↓
eb deploy
  ↓
배포 상태 확인
```

자동화 관점에서 중요한 점은 수동 콘솔 클릭을 줄이고, 같은 배포 절차를 반복 가능하게 만드는 것입니다.

운영 환경에서는 `eb deploy` 이후 헬스 체크, 이벤트 확인, 롤백 전략까지 함께 설계해야 합니다.

<br>

## 10. 배포 실패 시 확인할 것

Elastic Beanstalk 배포가 실패하면 다음 항목을 먼저 확인해야 합니다.

- 의존성 파일이 올바른지 확인합니다.
- zip 파일 구조가 올바른지 확인합니다.
- 애플리케이션 시작 명령이 올바른지 확인합니다.
- 환경 변수가 누락되지 않았는지 확인합니다.
- EC2 인스턴스 로그를 확인합니다.
- Environment 이벤트를 확인합니다.
- Health 상태가 왜 나빠졌는지 확인합니다.

관련 명령어는 다음과 같습니다.

```shell
eb status
eb health
eb events
eb logs
```

<br>

## 11. DVA-C02 시험 포인트

- EB CLI는 Elastic Beanstalk 작업을 쉽게 수행하기 위한 별도 CLI 도구입니다.
- `eb create`는 Environment를 생성할 때 사용합니다.
- `eb status`는 Environment 상태 확인에 사용합니다.
- `eb health`는 애플리케이션 헬스 확인에 사용합니다.
- `eb events`는 Environment 이벤트 확인에 사용합니다.
- `eb logs`는 로그 확인에 사용합니다.
- `eb deploy`는 새 애플리케이션 버전 배포에 사용합니다.
- `eb terminate`는 Environment 종료에 사용합니다.
- Python 애플리케이션은 `requirements.txt`로 의존성을 설명합니다.
- Node.js 애플리케이션은 `package.json`으로 의존성을 설명합니다.
- 콘솔 배포는 zip 업로드 후 새 Application Version을 만들고 배포합니다.
- CLI 배포는 명령어로 새 Application Version을 만들고 배포합니다.
- Elastic Beanstalk는 각 EC2 인스턴스에 zip을 배포하고 의존성을 설치한 뒤 애플리케이션을 시작합니다.

<br>

## 12. 시험 예상 문제

### 문제 1

Elastic Beanstalk Environment에 새 애플리케이션 버전을 배포하기 위해 사용하는 EB CLI 명령어는
무엇입니까?

- A. `eb deploy`
- B. `eb open`
- C. `eb terminate`
- D. `eb config`

<br>

### 정답

A. `eb deploy`

<br>

### 해설

`eb deploy`는 Elastic Beanstalk Environment에 새 애플리케이션 버전을 배포할 때 사용합니다.
`eb open`은 애플리케이션 URL을 여는 명령어이며, `eb terminate`는 Environment를 종료하는 명령어입니다.
`eb config`는 Environment 설정 확인 또는 수정에 사용합니다.

<br>

### 문제 2

Node.js 애플리케이션을 Elastic Beanstalk에 배포하려고 합니다. Beanstalk가 의존성을 파악하는 데
가장 중요한 파일은 무엇입니까?

A. `package.json`
B. `requirements.txt`
C. `Gemfile`
D. `template.yaml`

<br>

### 정답

A. `package.json`

<br>

### 해설

Node.js 애플리케이션에서는 `package.json`이 의존성 정보를 설명합니다.
Elastic Beanstalk는 배포된 코드에서 의존성 파일을 참고해 필요한 패키지를 설치하고 애플리케이션을 시작합니다.
`requirements.txt`는 Python, `Gemfile`은 Ruby에서 주로 사용합니다.

<br>

### 문제 3

Python 애플리케이션을 Elastic Beanstalk에 배포하려고 합니다. 필요한 Python 패키지 목록을
설명하는 대표 파일은 무엇입니까?

A. `package.json`
B. `requirements.txt`
C. `pom.xml`
D. `Dockerfile`

<br>

### 정답

B. `requirements.txt`

<br>

### 해설

Python 애플리케이션에서는 `requirements.txt` 파일에 필요한 패키지 목록을 작성합니다.
Elastic Beanstalk는 이 파일을 기준으로 의존성을 설치한 뒤 애플리케이션을 시작할 수 있습니다.
`package.json`은 Node.js, `pom.xml`은 Maven 기반 Java 프로젝트에서 사용합니다.

<br>

### 문제 4

Elastic Beanstalk에서 배포 실패 원인을 확인하려고 합니다. 애플리케이션 로그를 가져오는 데 가장
적절한 EB CLI 명령어는 무엇입니까?

A. `eb logs`
B. `eb create`
C. `eb open`
D. `eb init`

<br>

### 정답

A. `eb logs`

<br>

### 해설

`eb logs`는 Elastic Beanstalk Environment의 로그를 확인할 때 사용합니다.
배포 실패, 애플리케이션 시작 실패, 런타임 오류를 확인할 때 유용합니다.
`eb create`는 Environment 생성, `eb open`은 URL 열기, `eb init`은 프로젝트 초기 설정에 사용합니다.

<br>

## 요약

- EB CLI는 Elastic Beanstalk를 터미널에서 쉽게 관리하기 위한 도구입니다.
- 배포 프로세스는 의존성 파일 준비, 코드 zip 패키징, Application Version 생성,
  Environment 배포, EC2 인스턴스에서 의존성 설치와 애플리케이션 시작 순서로 이해하면 됩니다.
- DVA-C02에서는 EB CLI 명령어의 용도와 Python의 `requirements.txt`, Node.js의
  `package.json` 역할을 확실히 구분해야 합니다.
