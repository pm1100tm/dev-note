# FNM으로 Node.js 버전 관리하기

FNM(Fast Node Manager)은 Node.js 버전을 설치하고 프로젝트별로 전환하는 도구다. 여러 프로젝트가
서로 다른 Node.js 버전을 요구할 때 개발 환경을 재현하기 쉽게 만든다.

## 장점

- macOS, Windows, Linux에서 사용할 수 있다.
- 단일 실행 파일 기반이라 설치와 실행이 빠르다.
- `.node-version`, `.nvmrc` 파일을 읽어 프로젝트별 Node.js 버전을 전환할 수 있다.

## macOS 설치와 셸 설정

Homebrew로 설치한다.

```shell
brew install fnm
```

zsh를 사용한다면 `~/.zshrc`에 다음 설정을 추가한 후 터미널을 다시 열거나 설정 파일을 읽는다.

```shell
eval "$(fnm env --use-on-cd)"
source ~/.zshrc
```

`--use-on-cd` 옵션은 디렉터리를 이동할 때 버전 파일을 확인해 Node.js 버전을 자동 전환한다.

## 자주 쓰는 명령어

```shell
# FNM 버전과 설치 가능한 Node.js 버전 확인
fnm --version
fnm list-remote

# Node.js 설치와 사용
fnm install --lts
fnm install <version>
fnm use <version>

# 기본 버전 지정과 현재 사용 버전 확인
fnm default <version>
fnm current

# 설치한 버전 삭제와 도움말 확인
fnm uninstall <version>
fnm help
```

## 프로젝트별 버전 고정

프로젝트 루트에서 현재 Node.js 버전을 `.node-version` 파일에 기록한다.

```shell
node --version > .node-version
```

예를 들어 프로젝트가 Node.js `v22.0.0`을 요구한다면 `.node-version` 파일에는 해당 버전이
기록된다. 이후 다른 버전을 사용하다가 이 프로젝트로 다시 이동하면 FNM이 버전 파일을 읽어
자동으로 전환한다.

## 확인할 점

- 팀 프로젝트에서는 `.node-version` 또는 `.nvmrc`를 Git에 포함해 실행 환경을 공유한다.
- CI에서도 같은 Node.js 버전을 사용하도록 워크플로 설정을 맞춘다.
- Node.js 버전을 바꾼 뒤 의존성 오류가 발생하면 `node_modules`를 재설치하기 전에 lock 파일과
  지원 버전을 먼저 확인한다.
