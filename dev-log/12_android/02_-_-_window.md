# 안드로이드 환경 설정 - Window

- Java 버전 설치 / Java 환경변수 설정
- Android Studio 설치 / ANDROID_HOME 환경변수 설정
- Android Studio 가상 디바이스 설치

## Java 버전 설치(여기서는 17 버전으로 설정함)

- [오라클 접속](https://www.oracle.com/java/technologies/javase/jdk17-archive-downloads.html)
- Windows x64 Installer 다운로드
- 설치 위치 확인
  - C:\Program Files\Java\jdk-17
- 환경변수 설정
  - win 키
  - 검색 > 환경변수 > 시스템 환경변수 편집 > 클릭
  - 환경변수 > 클릭

## 시스템 환경 변수 등록

```plain
변수명 : JAVA_HOME
변수 값: C:\Program Files\Java\jdk-17
```

### 환경변수 편집

```plain
PATH 더블 클릭하여 아래 값 등록
%JAVA_HOME%\bin
```

### Java 환경변수 등록 확인

- cmd

```shell
$ java --version

java 17.0.12 2024-07-16 LTS
Java(TM) SE Runtime Environment (build 17.0.12+8-LTS-286)
Java HotSpot(TM) 64-Bit Server VM (build 17.0.12+8-LTS-286, mixed mode, sharing)
```

---

## Android Studio 설치

- [Android Studio 다운로드](https://developer.android.com/studio?hl=ko)
- 설치
- C:\Program Files\Android\Android Studio

### ANDROID_HOME 환경변수 설정

```shell
ANDROID_HOME
C:\Users\<username>\AppData\Local\Android\Sdk
```

### PATH 서정

```shell
%ANDROID_HOME%\platform-tools
%ANDROID_HOME%\emulator
```

### ANDROID 환경변수 등록 확인

```shell
adb --version

Android Debug Bridge version 1.0.41
Version 37.0.1-15733141
Installed as ...
```

```shell
emualtor --version


INFO     | Android emulator version ...
INFO     | Graphics backend: gfxstream
ERROR    | No AVD specified. Use '@foo' or '-avd foo' to launch a virtual
device named 'foo'
```

> 위 로그에서 ERROR는 에뮬레이터가 실행되어있지 않는 상태인 경우 표시됨

### Git bash 에서 환경변수 설정 필요

- git bash 열기
- vim ~/.bash_profile 에 아래의 환경변수 등록

```shell
export PATH="$PATH:/c/Users/<username>/AppData/Local/Android/Sdk/platform-tools"
export JAVA_HOME="c/Program Files/Java/jdk-17"
export PATH="$JAVA_HOME/bin:$PATH"
```

- source ~/.bash_profile 실행하여 현재 터미널 세션에 반영
- git bash 터미널에서 아래의 명령어 실행하여 정상 출력 확인

```shell
java --version
adb --version
emulator --version
```
