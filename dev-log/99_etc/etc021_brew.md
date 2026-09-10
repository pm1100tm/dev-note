# Homebrew

Homebrew 는 macOS 에서 가장 많이 사용하는 패키지 매니저이다.

터미널에서 `brew install ...` 형태로 CLI 도구, 개발 라이브러리, GUI 앱 등을 설치하고 관리할 수 있다.

## 핵심 개념

| 용어    | 설명                                | 예시                                  |
| ------- | ----------------------------------- | ------------------------------------- |
| Formula | CLI 도구나 라이브러리 패키지        | `git`, `wget`, `postgresql@16`        |
| Cask    | GUI 앱 패키지                       | `google-chrome`, `visual-studio-code` |
| Tap     | 기본 저장소 외의 추가 저장소        | `homebrew/cask`, `hashicorp/tap`      |
| Keg     | 패키지가 실제로 설치되는 디렉터리   | `/opt/homebrew/Cellar/git/...`        |
| Cellar  | Formula 들이 저장되는 루트 디렉터리 | `/opt/homebrew/Cellar`                |

## 설치

공식 설치 스크립트:

```shell
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
```

설치 후 `brew` 명령어가 동작하는지 확인한다.

```shell
brew --version
brew doctor
```

## 설치 경로

Mac CPU 종류에 따라 기본 설치 경로가 다르다.

| 환경                    | 기본 경로       |
| ----------------------- | --------------- |
| Apple Silicon, M1/M2/M3 | `/opt/homebrew` |
| Intel Mac               | `/usr/local`    |

현재 Homebrew prefix 확인:

```shell
brew --prefix
```

Apple Silicon 기준으로 PATH 가 잡혀 있지 않으면 `~/.zshrc` 에 추가한다.

```shell
echo 'eval "$(/opt/homebrew/bin/brew shellenv)"' >> ~/.zshrc
source ~/.zshrc
```

## 기본 명령어

### 패키지 검색

```shell
brew search <package>
```

예시:

```shell
brew search postgresql
```

### 패키지 정보 확인

```shell
brew info <package>
```

예시:

```shell
brew info node
brew info postgresql@16
```

### Formula 설치

```shell
brew install <package>
```

예시:

```shell
brew install git
brew install wget
brew install postgresql@16
```

### Cask 설치

GUI 앱은 `--cask` 옵션을 사용한다.

```shell
brew install --cask <app>
```

예시:

```shell
brew install --cask google-chrome
brew install --cask visual-studio-code
brew install --cask docker
```

### 설치된 패키지 목록

```shell
brew list
brew list --formula
brew list --cask
```

### 패키지 삭제

```shell
brew uninstall <package>
```

예시:

```shell
brew uninstall wget
brew uninstall --cask google-chrome
```

의존성 때문에 삭제가 막히면 신중하게 확인 후 사용한다.

```shell
brew uninstall --ignore-dependencies <package>
```

## 업데이트와 업그레이드

Homebrew 자체와 Formula 정보를 최신화한다.

```shell
brew update
```

설치된 패키지 중 업그레이드 가능한 목록을 확인한다.

```shell
brew outdated
```

설치된 패키지를 업그레이드한다.

```shell
brew upgrade
```

특정 패키지만 업그레이드할 수도 있다.

```shell
brew upgrade <package>
```

## 정리

오래된 패키지 버전과 캐시를 정리한다.

```shell
brew cleanup
```

삭제될 항목을 미리 보고 싶으면 `-n` 옵션을 사용한다.

```shell
brew cleanup -n
```

## 서비스 관리

DB, Redis 처럼 백그라운드로 실행되는 프로그램은 `brew services` 로 관리할 수 있다.

### 서비스 목록 확인

```shell
brew services list
```

### 서비스 시작

```shell
brew services start <package>
```

예시:

```shell
brew services start postgresql@16
brew services start redis
```

### 서비스 중지

```shell
brew services stop <package>
```

### 서비스 재시작

```shell
brew services restart <package>
```

### 서비스 등록 해제

서비스 plist 등록까지 제거한다.

```shell
brew services cleanup
```

## Tap 관리

기본 Homebrew 저장소에 없는 패키지는 tap 을 추가해서 설치한다.

```shell
brew tap <repository>
```

예시:

```shell
brew tap hashicorp/tap
brew install hashicorp/tap/terraform
```

등록된 tap 목록:

```shell
brew tap
```

tap 제거:

```shell
brew untap <repository>
```

## 버전 고정

특정 패키지를 업그레이드 대상에서 제외하고 싶을 때 사용한다.

```shell
brew pin <package>
```

고정 해제:

```shell
brew unpin <package>
```

고정된 패키지 확인:

```shell
brew list --pinned
```

## 문제 확인

Homebrew 상태 점검:

```shell
brew doctor
```

설정 정보 확인:

```shell
brew config
```

패키지 의존성 확인:

```shell
brew deps <package>
```

특정 패키지를 누가 의존하는지 확인:

```shell
brew uses --installed <package>
```

## 자주 쓰는 패턴

### 설치 가능한 정확한 이름 찾기

```shell
brew search <keyword>
brew info <package>
brew install <package>
```

### 특정 버전 패키지 설치

```shell
brew search postgresql
brew install postgresql@16
```

버전이 붙은 Formula 는 PATH 를 직접 추가해야 하는 경우가 많다.

```shell
echo 'export PATH="/opt/homebrew/opt/postgresql@16/bin:$PATH"' >> ~/.zshrc
source ~/.zshrc
```

### 패키지 재설치

```shell
brew reinstall <package>
```

### 설치 파일 다운로드만 하기

```shell
brew fetch <package>
```

## 주의할 점

- `brew update` 는 Homebrew 의 패키지 정보를 최신화한다.
- `brew upgrade` 는 실제 설치된 패키지를 업그레이드한다.
- `brew cleanup` 은 오래된 버전과 캐시를 삭제한다.
- DB 계열 패키지는 `brew uninstall` 만으로 데이터 디렉터리가 삭제되지 않을 수 있다.
- Apple Silicon 과 Intel Mac 은 설치 경로가 다르므로 블로그 명령어를 그대로 복붙하기 전에 `brew --prefix` 를 확인한다.
- 프로젝트에서 Docker 로 DB 를 띄우는 경우, 로컬 Homebrew 서비스와 포트가 충돌할 수 있다.

## 자주 쓰는 명령어 요약

```shell
brew --version
brew doctor
brew update
brew search <package>
brew info <package>
brew install <package>
brew install --cask <app>
brew list
brew outdated
brew upgrade
brew uninstall <package>
brew cleanup
brew services list
brew services start <package>
brew services stop <package>
```

| command                         | description                       |
| ------------------------------- | --------------------------------- |
| `brew --version`                | Homebrew 버전 확인                |
| `brew doctor`                   | Homebrew 상태 점검                |
| `brew update`                   | Homebrew 와 Formula 정보를 최신화 |
| `brew search <package>`         | 설치 가능한 패키지 검색           |
| `brew info <package>`           | 패키지 상세 정보 확인             |
| `brew install <package>`        | Formula 설치                      |
| `brew install --cask <app>`     | GUI 앱 설치                       |
| `brew list`                     | 설치된 패키지 목록 확인           |
| `brew outdated`                 | 업그레이드 가능한 패키지 확인     |
| `brew upgrade`                  | 설치된 패키지 업그레이드          |
| `brew uninstall <package>`      | 패키지 삭제                       |
| `brew cleanup`                  | 오래된 버전과 캐시 정리           |
| `brew services list`            | Homebrew 서비스 목록 확인         |
| `brew services start <package>` | 서비스 시작                       |
| `brew services stop <package>`  | 서비스 중지                       |
