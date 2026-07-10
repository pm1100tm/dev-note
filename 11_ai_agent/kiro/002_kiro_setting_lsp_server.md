# Kiro Language Server Protocol Sever 설정(LSP 서버)

## Next.js with TypeScript

```shell
npm install -g typescript-language-server typescript
typescript-language-server --version
/code init
/code init -f
/code status
```

## Java Spring

### 1. Java LSP 서버 설치

먼저 jdtls(Eclipse JDT Language Server)를 설치해야 합니다:

```shell
brew install jdtls
```

### 2. 프로젝트에서 LSP 초기화

```shell
/code init

✓ Workspace initialization started
Workspace: /Users/wondushim/Desktop/work-ai-lounge/ai-lounge-backend
Detected Languages: ["java"]
Project Markers: ["build.gradle"]

Available LSPs:
✓ jdtls (java) - initialized (687ms)

상태 표시:

- ✓ - 초기화 완료 및 준비됨
- ◐ - 현재 초기화 중
- ○ available - 설치됨 (현재 프로젝트에 불필요)
- ○ not installed - 시스템에 미설치
```

### 3. 초기화 확인

설정이 제대로 되었는지 확인:

```
/code status
```

### 4. LSP 기능 사용

초기화 후 다음 기능들을 사용할 수 있습니다:

- 심볼 검색: 클래스, 메서드 찾기
- 참조 찾기: 특정 심볼이 사용된 모든 위치
- 정의로 이동: 심볼 정의 위치 확인
- 진단: 컴파일 오류 및 경고
- hover 정보: 타입 정보 및 문서
- 리팩토링: 심볼 이름 변경

### 5. 재시작이 필요한 경우

LSP 서버가 응답하지 않거나 문제가 있을 때: (강제 재초기화)

```
/code init -f
```

### 6. 로그 확인

문제가 발생하면 로그를 확인: 자동 초기화

```
/code logs -l ERROR -n 50
```

첫 /code init 실행 후, .kiro/settings/lsp.json 파일이 생성되며, 이후 Kiro CLI 시작 시
자동으로 LSP가 초기화됩니다.
