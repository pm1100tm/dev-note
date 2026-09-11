# Codex 지침 경로 설정

## 기본 실행

대화형 Codex CLI를 실행한다.

```shell
> codex
```

```shell
codex

프롬프트를 바로 넘겨서 시작할 수도 있다.

codex "이 프로젝트 구조를 분석해줘"
```

## 핵심 명령어

| 명령어 기능      | 설명                                     |
| ---------------- | ---------------------------------------- |
| codex            | 대화형 CLI 실행                          |
| codex exec       | Codex를 비대화형으로 실행                |
| codex review     | 코드 리뷰 실행                           |
| codex login      | 로그인 관리                              |
| codex logout     | 저장된 인증 정보 제거                    |
| codex resume     | 이전 세션 이어서 실행                    |
| codex apply      | Codex가 만든 diff를 로컬 작업트리에 적용 |
| codex mcp        | MCP 서버 관리                            |
| codex plugin     | Codex 플러그인 관리                      |
| codex doctor     | 설치, 설정, 인증, 런타임 상태 진단       |
| codex update     | Codex CLI 업데이트                       |
| codex app        | Codex 데스크톱 앱 실행 또는 설치         |
| codex completion | 쉘 자동완성 스크립트 생성                |
| codex sandbox    | Codex 샌드박스 안에서 명령 실행          |
| codex help       | 도움말 출력                              |

## 공통 주요 옵션

대부분의 Codex 명령어에서 공통으로 사용할 수 있는 옵션이다.

### -C, --cd <DIR>

Codex가 사용할 작업 디렉터리를 지정한다.

```shell
codex -C /path/to/project
codex exec -C /path/to/project "테스트를 실행해줘"
```

---

## codex review

코드 리뷰를 비대화형으로 실행한다.

```shell
codex review
```

현재 작업 중인 변경사항 전체를 리뷰하려면:

```shell
codex review --uncommitted
```

특정 브랜치 기준으로 리뷰하려면:

```shell
codex review --base main
```

특정 커밋을 리뷰하려면:

```shell
codex review --commit <SHA>
```

주요 옵션:

| 옵션 기능                                 | 설명                             |
| ----------------------------------------- | -------------------------------- |
| --uncommitted staged, unstaged, untracked | 변경사항 리뷰                    |
| --base <BRANCH>                           | 특정 base 브랜치와 비교해서 리뷰 |
| --commit <SHA>                            | 특정 커밋이 만든 변경사항 리뷰   |
| --title <TITLE>                           | 리뷰 요약에 표시할 제목 지정     |

---

## codex resume

이전 대화형 세션을 이어서 실행한다.

```shell
codex resume
```

가장 최근 세션을 바로 이어서 실행:

```shell
codex resume --last
```

모든 세션을 표시:

```shell
codex resume --all
```

주요 옵션:

| 옵션 기능                 | 설명                                   |
| ------------------------- | -------------------------------------- |
| --last                    | 가장 최근 세션 이어서 실행             |
| --all                     | 현재 디렉터리 필터 없이 모든 세션 표시 |
| --include-non-interactive | 비대화형 세션도 포함                   |

---

## codex login

```shell
codex login
codex logout
codex login status

# API 키로 로그인:
printenv OPENAI_API_KEY | codex login --with-api-key

# Access Token으로 로그인:
printenv CODEX_ACCESS_TOKEN | codex login --with-access-token
```

---

## codex mcp

외부 MCP 서버를 관리한다.

```shell
codex mcp list
codex mcp get <NAME>
codex mcp add <NAME> --url <URL>
codex mcp remove <NAME>
codex mcp login <NAME>
codex mcp logout <NAME>
```

MCP는 Codex가 외부 도구, 문서, 데이터 소스, 앱 커넥터와 연결될 수 있게 해주는 확장 인터페이스다.

---

## codex plugin

Codex 플러그인을 관리한다.

```shell
codex plugin list
codex plugin add <PLUGIN_NAME>
codex plugin remove <PLUGIN_NAME>
codex plugin marketplace
```

플러그인은 skill, MCP 서버, 앱, 훅, 리소스 등을 묶어서 Codex 기능을 확장하는 단위다.

---

## codex doctor

Codex 로컬 환경을 진단한다.

```shell
codex doctor
```

요약만 보기:

```shell
codex doctor --summary
```

JSON 리포트 출력:

```shell
codex doctor --json
```

주요 옵션:

| 옵션 기능                        | 설명                                |
| -------------------------------- | ----------------------------------- |
| --summary                        | 요약 진단만 출력                    |
| --json redacted machine-readable | 리포트 출력                         |
| --all                            | 상세 출력에서 긴 목록까지 모두 표시 |
| --no-color                       | ANSI 색상 제거                      |
| --ascii ASCII                    | 상태 표시 사용                      |

---

## /compact — 긴 대화의 컨텍스트 압축

`/compact`는 지금까지의 대화를 요약해 모델이 참고하는 컨텍스트 공간을 확보하는 기능이다. 긴 코드 분석이나 디버깅을
진행한 뒤, 중요한 결정사항을 유지하면서 같은 작업을 이어갈 때 사용한다.

터미널에서 `codex compact`를 실행하는 방식이 아니라, `codex`로 대화형 CLI를 연 뒤 대화 입력창에 슬래시 명령을 입력한다.

```shell
codex
```

Codex 입력창에서 다음 명령을 입력한다.

```shell
/compact
```

확인 안내가 표시되면 안내에 따라 진행한다. 이전 대화가 간결한 요약으로 대체되며, 압축이 끝나면 같은 대화에서 다음 요청을
입력할 수 있다.

### 먼저 알아둘 점: 언제, 어디에 입력하는가?

Codex와 한동안 대화하다가 **같은 작업은 계속하되, 쌓인 대화 내용을 간추리고 싶을 때** 사용한다.
여기서 컨텍스트란 Codex가 다음 답변을 만들 때 참고하는 대화, 파일을 읽은 결과, 명령 실행 결과 등의 정보다.

예를 들어 오류 하나를 찾기 위해 로그를 여러 번 붙이고, 파일 열 개를 읽고, 여러 원인을 검토했다면
대화에는 이미 필요 없어진 중간 조사 내용도 많이 쌓여 있다. `/compact`는 이런 이전 내용을 핵심 요약으로 줄인다.

사용자가 해야 하는 필수 입력은 아래 한 줄뿐이다. **이미 실행 중인 Codex의 대화 입력창에 입력하고 Enter를 누른다.**

```shell
/compact
```

미리 "요약해줘"라고 요청하는 것은 필수 절차가 아니다. `/compact` 자체가 대화 압축을 수행한다.
명령이 처리된 뒤에는 평소처럼 다음 작업을 요청하면 된다.

### 구체적인 예시: Spring Boot 로그인 오류를 조사하던 중

아래는 사용법을 설명하기 위한 가상의 상황이다. 파일명, 오류 원인, 요약 내용은 실제 프로젝트 분석 결과가 아니다.

**1. 먼저 Codex와 오류 원인을 조사한다.**

Codex 대화 입력창에서 다음과 같이 대화를 이어갔다고 가정한다.

```shell
사용자: 로그인 API가 401을 반환해. SecurityConfig.java부터 확인해줘.
Codex: 보안 설정을 확인했습니다. JWT 필터 처리도 확인해야 합니다.

사용자: JwtAuthenticationFilter.java도 확인해줘. 아래는 요청 로그야.
        ... 긴 로그 ...
Codex: 로그인 요청에도 JWT 필터가 실행되고 있습니다.

사용자: 토큰이 없는 로그인 요청도 통과해야 해. API 응답 형식은 바꾸지 마.
Codex: 로그인 경로에서는 토큰 검사를 건너뛰도록 수정하는 방향을 검토하겠습니다.

사용자: 아직 수정하지 말고 관련 테스트까지 살펴봐.
Codex: 테스트를 확인했습니다. 이제 필터 수정과 회귀 테스트 추가가 남았습니다.
```

이 시점에는 긴 로그, 파일 내용, 여러 차례의 분석이 대화에 쌓여 있다. 원인 조사는 끝났고 실제 수정이 남은 상태다.

**2. 같은 입력창에 `/compact`만 입력한다.**

```shell
/compact
```

Codex가 압축을 완료할 때까지 기다린다. 확인 안내가 나오면 안내에 따른다.
새 터미널을 열거나 Codex를 종료할 필요는 없다.

압축 결과는 개념적으로 다음과 같은 작업 메모에 가깝다. 아래 내용이 화면에 그대로 출력된다는 뜻은 아니며,
실제 요약의 형식과 보존되는 세부사항은 달라질 수 있다.

```shell
목표: 로그인 API의 401 오류 수정.
확인한 원인: 로그인 요청에도 JWT 필터가 토큰 검사를 수행함.
수정 대상: JwtAuthenticationFilter.java.
제약: 기존 API 응답 형식을 유지할 것.
현재 상태: 조사만 완료했으며 코드는 아직 수정하지 않음.
남은 작업: 로그인 경로 예외 처리 및 회귀 테스트 추가·실행.
```

**3. 압축이 끝나면 다음 작업을 요청한다.**

```shell
이제 로그인 경로가 JWT 검사를 건너뛰도록 수정해줘.
API 응답 형식은 유지하고, 토큰 없는 로그인 요청과 보호된 API 요청을 각각 테스트해줘.
```

- Codex는 압축된 맥락을 참고해 작업을 이어간다.
- 수정에 필요한 정확한 코드나 로그가 요약에 없다면 파일을 다시 읽거나 추가 확인이 필요할 수 있다.
- 중요한 제약은 위 예시처럼 다음 요청에 다시 적어 주어도 좋다.

### 압축 전후에 달라지는 것

| 항목                      | `/compact` 실행 후                                                |
| ------------------------- | ----------------------------------------------------------------- |
| 모델이 참고하는 이전 대화 | 긴 원문 대신 핵심 요약을 사용하므로 컨텍스트 공간이 확보됨        |
| 작업 목표와 진행 상황     | 핵심 내용을 요약해 이어가지만, 세부사항이 모두 보존되는 것은 아님 |
| 이미 수정한 소스 파일     | 그대로 유지됨. 압축이 파일 수정이나 되돌리기를 수행하지 않음      |
| 다음 요청 방법            | 같은 대화 입력창에 평소처럼 요청하면 됨                           |
| 사용량 한도               | 초기화되지 않음                                                   |

### "요약해줘"와 `/compact`는 어떻게 다른가?

`지금까지 내용을 요약해줘`는 사용자가 읽을 요약 답변을 요청하는 일반 대화다.
이 요청만으로 컨텍스트 압축 기능이 실행되는 것은 아니다.

`/compact`는 Codex의 컨텍스트 압축 기능을 직접 실행하는 명령이다.
따라서 단순히 압축하려는 목적이라면 `/compact`만 입력하면 된다.
압축 전에 결정사항이 맞는지 직접 검토하고 싶을 때만 별도로 요약을 요청하면 된다.

### 어떤 상황에서 사용하면 좋은가?

- 긴 로그와 파일 분석을 반복한 뒤, 원인이 정리되어 수정 단계로 넘어갈 때.
- 하나의 기능을 여러 차례 수정하면서 대화가 길어졌지만 같은 작업을 계속해야 할 때.
- `/status`로 컨텍스트 사용 상태를 확인한 뒤, 이전 대화를 간추릴 필요가 있다고 판단할 때.

짧은 질문 몇 번마다 실행할 필요는 없다. 완전히 다른 작업을 시작하려는 경우에는 새 대화를 여는 `/new`가 더 적합하다.

### 주의사항

- 압축은 요약이므로 이전 대화의 모든 세부사항을 원문 그대로 유지하는 것은 아니다. 반드시 지켜야 할 제약이나 결정사항은
  `AGENTS.md` 또는 작업 문서에도 기록해 두는 것이 좋다.
- 프로젝트 파일을 압축하거나 Git 변경사항을 되돌리는 명령이 아니다.
- 사용량 한도나 이미 소비한 토큰을 초기화하는 기능이 아니다.
- 새 대화를 시작하는 `/new`와 달리, 기존 작업의 핵심 맥락을 요약해 이어가는 용도다.

공식 문서: [CLI 슬래시 명령 — /compact](https://developers.openai.com/codex/cli/slash-commands#keep-transcripts-lean-with-compact)

---

## /fast — 모델의 Fast 모드 설정

`/fast`는 지원되는 모델의 Fast 서비스 등급을 켜거나 끄는 기능이다. 모델 응답 속도를 높이는 대신 크레딧 소비가 증가한다.
현재 모델의 서비스 등급을 바꾸는 기능이며, 별도의 경량 모델로 교체하는 명령은 아니다.

`/compact`와 마찬가지로 `codex fast`라는 셸 명령이 아니라, 실행 중인 Codex의 대화 입력창에서 사용한다.

### 기본 사용법

| 입력           | 동작                     |
| -------------- | ------------------------ |
| `/fast`        | Fast 모드 켜기/끄기 전환 |
| `/fast on`     | Fast 모드 켜기           |
| `/fast off`    | Fast 모드 끄기           |
| `/fast status` | 현재 Fast 모드 상태 확인 |

명시적으로 켜거나 끄려면 `on`과 `off`를 사용하는 것이 편하다. 현재 모델이 Fast 등급을 지원하지 않으면
`/fast`가 메뉴에 표시되지 않을 수 있다.

### 사용 예시

빠른 피드백이 필요한 수정 작업을 시작하기 전에 켜고, 작업 후 상태를 확인하거나 끌 수 있다. 다음 입력도 각각 순서대로 실행한다.

```shell
/fast on
```

```shell
방금 수정한 React 컴포넌트의 타입 오류를 확인하고 수정해줘.
```

```shell
/fast status
```

```shell
/fast off
```

### 기본 설정으로 저장하기

사용하는 Codex 홈의 `config.toml`에 다음 설정을 둘 수 있다. 프로젝트별 `CODEX_HOME`을 지정했다면 해당 홈의
`config.toml`을 사용한다.

```toml
service_tier = "fast"

[features]
fast_mode = true
```

기존 파일에 병합할 때 `service_tier`는 최상위 설정 영역에 넣고, `[features]`가 이미 있으면 해당 테이블에
`fast_mode`를 추가한다. 같은 테이블을 중복 선언하지 않는다.

### 비용과 적용 범위

- Fast 모드는 지원 모델과 계정 환경에서 사용할 수 있으며, ChatGPT 로그인 시 CLI·데스크톱 앱·IDE 확장에 제공된다.
- 크레딧 소비 배율은 모델별로 다르다. 사용 전 현재 모델의 공식 요금 안내를 확인한다.
- API 키 사용 시에는 API 토큰 요금이 적용된다. ChatGPT 크레딧 배율을 API 비용에 그대로 적용하면 안 되며,
  API Priority 처리에는 별도 요금이 있다.
- 모델 응답 속도를 높이는 기능이므로 로컬 빌드, 테스트, 네트워크 요청 등 전체 작업 시간이 같은 비율로 단축되는 것은 아니다.
- `/fast`로 전환한 선택은 저장되므로, 일시적으로 켰다면 작업 후 `/fast off`로 끄고 상태를 확인한다.

공식 문서:

- [Speed — Fast mode](https://developers.openai.com/codex/speed),
- [CLI 슬래시 명령 — /fast](https://developers.openai.com/codex/cli/slash-commands#toggle-fast-mode-with-fast)
