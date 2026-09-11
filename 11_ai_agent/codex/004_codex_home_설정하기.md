# Codex home 설정하는 방법

MacOS 기준 codex 를 설치하면, home 에 .codex 폴더가 생성됩니다.
폴더에 들어가서 보면 아래와 같은 파일들이 있고, 여기서 중요한 것은 config.toml 이며, 여기에 여러가지 설정들이
잡혀있습니다.

```shell
AGENTS.md
ambient-suggestions
auth.json
browser
cache
chrome-native-hosts-v2.json
computer-use
config.toml
goals_1.sqlite
goals_1.sqlite-shm
goals_1.sqlite-wal
history.jsonl
installation_id
...
```

- 즉, **codex 명령어를 실행하면** 기기의 HOME에 있는 `~/.codex` 부터 현재 작업 디렉토리까지 지침을 탐색합니다.
- 빈 지침 파일은 무시합니다.
- 즉, HOME에 있는 `~/.codex`는 global 지침입니다.

## 특정 프로젝트마다 AGENTS.md 를 다르게 설정하고 싶고, 이걸 스마트하게 관리하고 싶다.

프로젝트 루트 폴더 하위에 `AGENTS.md` 파일을 만들면 되지만, 다른 여러 AI Agent 를 사용한다면, 각 에이전트 마다
지침 파일을 폴더로 나눠서 관리하고 싶습니다.

조사된 결과에 따르면 가능하지만, 조금 불편할 수 있을 것 같습니다. CODEX_HOME 을 설정하면서 실행하면 됩니다.

```shell
dev-note/
└── .codex/
    ├── AGENTS.md      # 이 Codex 홈을 사용하는 실행에 적용할 지침
    └── config.toml    # 별도 Codex 설정
```

```shell
CODEX_HOME="$PWD/.codex" codex "현재 적용된 지침 파일과 주요 규칙을 요약해줘."
```

> 아예, shell 에 설정하는 것은 특정 프로젝트 경로의 CODEX_HOME 을 읽기 때문에, 적합하지 않을 수 있겠습니다.

```shell
# 이렇게 설정하면 매번 바꿔줘야함..
CODEX_HOME=/Users/<username>/Desktop/dev-note/.codex codex
```

<br>

## CODEX_HOME

`CODEX_HOME`을 지정하지 않고 프로젝트에 `.codex/AGENTS.md`만 생성하면, 프로젝트 전체 지침으로 자동 적용되지는
않습니다. 프로젝트 지침만 필요하다면 루트 `AGENTS.md`를 사용하는 것이 간단합니다. 지침과 Codex 설정을 프로젝트별로
분리하려면 위와 같이 `CODEX_HOME`을 지정할 수 있습니다.

<br>

## 주의사항과 적용 확인

- `CODEX_HOME`은 지침 파일 위치만 바꾸는 설정이 아니라 Codex 홈 자체를 변경합니다.
  기존 `~/.codex`의 설정이나 인증 상태가 그대로 사용된다고 가정하면 안 되며, 별도 로그인이 필요할 수 있습니다.
- 프로젝트의 `.codex`에 인증 정보나 세션 데이터가 저장될 수 있으므로, 로컬 전용으로 운영한다면 `.gitignore`에
  `/.codex/`를 추가하는 것이 좋습니다.
- 이 명령은 새로 실행하는 Codex CLI에 적용됩니다. 현재 열려 있는 Codex 앱 세션의 홈을 변경하지는 않습니다.
- 새 터미널에서 실행할 때도 같은 명령을 사용해야 합니다. 프로젝트에 `.codex`가 있다는 이유만으로 자동 선택되지는 않습니다.
- 지침을 작성하거나 변경한 뒤에는 해당 홈을 지정해 새 Codex 세션을 시작하고, 적용 중인 지침을 확인합니다.
- 같은 위치에 비어 있지 않은 `AGENTS.override.md`가 있으면 해당 파일이 우선합니다.

공식 문서: [Custom instructions with AGENTS.md — 지침 탐색 및 CODEX_HOME 설정](https://developers.openai.com/codex/guides/agents-md)
