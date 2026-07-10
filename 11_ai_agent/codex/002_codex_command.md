# Codex CLI 자주 쓰는 명령어와 옵션

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

## 공통 주요 옵션

대부분의 Codex 명령어에서 공통으로 사용할 수 있는 옵션이다.

### -C, --cd <DIR>

Codex가 사용할 작업 디렉터리를 지정한다.

```shell
codex -C /path/to/project
codex exec -C /path/to/project "테스트를 실행해줘"
```
