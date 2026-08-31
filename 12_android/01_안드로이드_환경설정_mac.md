# 안드로이드 환경 설정 - Mac

- Java 버전 설치 / Java 환경변수 설정
- Android Studio 설치 / ANDROID_HOME 환경변수 설정
- Android Studio 가상 디바이스 설치

## Java 설치 및 설정

- [실습001*Java*환경설정\_참고](../05_java/실습001_Java_환경설정.md)

## Android Studio

- 브라우저 '안드로이드 스튜디오 다운로드 검색'
- 설치

## 확인

```shell
❯ java --version
openjdk 17.0.20.1 2026-08-18
OpenJDK Runtime Environment Temurin-17.0.20.1+1 (build 17.0.20.1+1)
OpenJDK 64-Bit Server VM Temurin-17.0.20.1+1 (build 17.0.20.1+1, mixed mode, sharing)


❯ adb --version
Android Debug Bridge version 1.0.41
Version 37.0.1-15733141
Installed as /Users/wondushim/Library/Android/sdk/platform-tools/adb
Running on Darwin 25.6.0 (arm64)


╰─ emulator --version                                                                                           ─╯
INFO         | Android emulator version 37.1.11.0 (build_id 15917651) (CL:N/A)
INFO         | Graphics backend: gfxstream
ERROR        | No AVD specified. Use '@foo' or '-avd foo' to launch a virtual device named 'foo'
```

> Z SHELL 에서 안드로이드 설정 블록

```shell
export ANDROID_HOME="$HOME/Library/Android/sdk"
export ANDROID_SDK_ROOT="$ANDROID_HOME"
export PATH="$ANDROID_HOME/platform-tools:$PATH"
export PATH="$HOME/Library/Android/sdk/build-tools/36.0.0:$PATH"
export PATH="$ANDROID_HOME/emulator:$ANDROID_HOME/platform-tools:$ANDROID_HOME/build-tools/36.0.0:$PATH"
```
