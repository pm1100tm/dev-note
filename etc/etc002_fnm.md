# FNM

![FNM](./assets/img-fnm.png)

- Fast Node Manager
- Node.js 버전 관리 도구 중의 하나이다.
- [FNM Github](https://github.com/Schniz/fnm)

## 장점

- cross-platform 지원(macOS, Windows, Linux)
  - nvm 은 Windows를 지원하지 않으며, nvm-windows 를 사용해야 한다.
- 단일 파일, 설치가 간편하며, 속도가 빠르다
  - 설치와 구성이 nvm 보다 쉽고, nmv 에 비해서 속도가 빠르다.
- .node-version 및 .nvmrc 파일에서 작동한다.
- 특정 프로젝트로 진입했을 때 node version 변환이 가능하다.

## fnm 설치

```shell
brew install fnm
```

## command

FNM 버전 확인

```shell
fnm --version
```

설치할 수 있는 node 버전 리스트 확인

```shell
fnm list-remote
```

특정 node version 설치

```shell
fnm install <version>
```

설치한 특정 node version 삭제

```shell
fnm uninstall <version>
```

설치한 node version 사용

```shell
fnm use <version>
```

설치한 특정 node version 을 기본 version 으로 설정

```shell
fnm default <version>
```

fnm 으로 사용하고 있는 현재 node version 확인

```shell
fnm current
```

fnm 명령어 확인

```shell
fnm help
```

## 프로젝트 안으로 진입했을 때 node version 변경하기

- 프로젝트 root path에 .node-version 파일을 생성
  - node --version > .node-version

프로젝트 밖에서 다른 node version 으로 변경하고, 다시 프로젝트로 진입하면,
node version 이 자동으로 바뀌는 것을 확인할 수 있다.
