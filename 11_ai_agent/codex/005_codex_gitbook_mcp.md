# Codex에 GitBook MCP 연결하기

> 확인일: 2026-09-11. 이 문서는 Codex CLI와 GitBook의 원격 MCP를 기준으로 한다.

- GitBook MCP는 Codex가 GitBook 문서를 검색하거나 GitBook 공간에 변경을 제안하도록 연결하는 통로다.
- GitBook에는 목적이 다른 MCP 서버가 두 개 있으므로, 먼저 필요한 범위를 구분해야 한다.

| 구분          | 서버 URL                       | 권한과 용도                                                                  |
| ------------- | ------------------------------ | ---------------------------------------------------------------------------- |
| GitBook MCP   | `https://mcp.gitbook.com/mcp`  | OAuth 또는 PAT로 로그인한 계정 범위에서 문서·공간을 읽고 변경 요청을 만든다. |
| 게시 문서 MCP | `<게시-문서-URL>/~gitbook/mcp` | 공개된 최신 문서를 검색·읽기만 한다. 초안과 미게시 변경은 볼 수 없다.        |

이 문서에서 말하는 "GitBook MCP"는 기본적으로 첫 번째, 즉 GitBook 작업용 MCP를 뜻한다.
내 GitBook 문서를 Codex의 참고 자료로만 쓰려면 두 번째 서버만 등록하는 것이 권한 범위가 작다.

공식 자료:

- [GitBook MCP 개요](https://www.gitbook.com/blog/create-documentation-with-ai-and-mcp),
- [게시 문서 MCP](https://gitbook.com/docs/ai-for-your-readers/mcp-servers-for-published-docs.md)

## 적용 전 준비

- Codex CLI가 설치되고 `codex login`으로 Codex에 로그인되어 있어야 한다.
- GitBook MCP에서 작성 작업을 하려면 GitBook 계정과 대상 조직·공간에 대한 권한이 필요하다.
- OAuth 인증은 브라우저에서 GitBook 로그인을 진행한다. 조직의 보안 정책상 OAuth를 쓸 수 없다면
  GitBook Personal Access Token(PAT)을 사용한다.
- PAT는 비밀번호와 같이 취급한다. `config.toml`, Git 저장소, 문서, 셸 이력에 토큰 값을 직접 쓰지
  않는다.

## GitBook 작업용 MCP 설정

### OAuth로 연결하기

일반적으로 OAuth가 가장 안전하고 관리하기 쉽다. 터미널에서 다음 명령을 실행한다.

```shell
codex mcp add gitbook-mcp --url https://mcp.gitbook.com/mcp
codex mcp login gitbook-mcp
```

두 번째 명령은 브라우저 인증 흐름을 시작한다. GitBook에 로그인하고 요청된 권한을 검토해 승인한다.
인증을 마치면 새 Codex CLI 세션을 열어 사용한다.

```shell
❯ codex mcp add gitbook-mcp --url https://mcp.gitbook.com/mcp

Added global MCP server 'gitbook-mcp'.
Detected OAuth support. Starting OAuth flow…
Authorize `gitbook-mcp` by opening this URL in your browser:
https://oauth.gitbook.com/authorize?response_type=code&............................

Successfully logged in.
```

### PAT로 연결하기

자동화 환경처럼 OAuth를 사용할 수 없을 때만 PAT를 사용한다. 토큰은 환경 변수에 두고, Codex 설정에는
환경 변수 이름만 기록한다.

```shell
export GITBOOK_MCP_TOKEN='<GitBook-PAT>'

codex mcp add gitbook-mcp \
  --url https://mcp.gitbook.com/mcp \
  --bearer-token-env-var GITBOOK_MCP_TOKEN
```

`<GitBook-PAT>`는 실제 PAT로 교체하며, 위 `export`는 현재 터미널에만 적용된다. 지속 설정이 필요하면
OS의 비밀 저장소나 CI의 secret 관리 기능을 사용한다. 셸 시작 파일에 평문 토큰을 저장하는 방법은 피한다.

동일 설정을 `~/.codex/config.toml`에 직접 작성할 수도 있다.

```toml
[mcp_servers.gitbook-mcp]
url = "https://mcp.gitbook.com/mcp"
bearer_token_env_var = "GITBOOK_MCP_TOKEN"
```

`~/.codex/config.toml`은 사용자 전체 Codex 설정이다. 특정 프로젝트에서만 설정을 분리하려고
`CODEX_HOME`을 바꾸면 인증·설정 저장 위치도 함께 바뀔 수 있으므로, 기존 Codex 홈 설정을 확인한다.

## 게시 문서를 읽기 전용으로 연결하기

게시된 GitBook 사이트의 내용을 조사만 할 때는 사이트 URL 끝에 `/~gitbook/mcp`를 붙여 등록한다.
예를 들어 사이트 주소가 `https://docs.example.com`이면 다음과 같다.

```shell
codex mcp add example-docs \
  --url https://docs.example.com/~gitbook/mcp
```

이 서버는 게시된 최신 버전만 제공한다. 비공개 사이트라면 사이트의 인증 정책에 맞는 접근이 추가로
필요할 수 있다. 브라우저에서 MCP URL을 열었을 때 오류가 보이는 것은 정상일 수 있으며, MCP 클라이언트가
HTTP로 연결해야 한다.

## 설정과 연결 상태 확인

먼저 등록 정보가 기대한 URL과 인증 방식인지 확인한다.

```shell
codex mcp list
codex mcp get gitbook-mcp
codex mcp get gitbook-mcp --json
```

정상 등록 예시는 다음과 같이 서버 이름과 URL이 보이는 형태다. 실제 출력의 인증 상태 표기는 Codex 버전에
따라 달라질 수 있다.

```shell
gitbook-mcp
  url: https://mcp.gitbook.com/mcp
```

`codex mcp list`는 **등록 상태**를 확인한다. 실제 연결과 권한은 새 Codex 세션에서 GitBook 관련 요청을
한 번 수행해 확인해야 한다. 예를 들면 다음과 같이 요청한다.

```text
GitBook MCP를 사용해 내가 접근할 수 있는 조직 또는 공간을 조회해 줘.
```

OAuth가 끝나지 않았거나 토큰을 읽지 못하면 인증 오류가, 권한이 부족하면 GitBook 권한 오류가 나온다.
문제 발생 시에는 다음 순서로 확인한다.

- `codex mcp get gitbook-mcp`에서 URL 철자와 서버 이름을 확인한다.
- OAuth 방식이면 `codex mcp login gitbook-mcp`를 다시 실행한다.
- PAT 방식이면 현재 실행한 Codex 프로세스가 `GITBOOK_MCP_TOKEN` 환경 변수를 받을 수 있는지 확인한다.
- 조직·공간 권한, 비공개 사이트의 인증 정책, 네트워크에서 `https://mcp.gitbook.com/mcp` 접근 가능 여부를
  확인한다.

설정을 제거하려면 다음 명령을 사용한다. GitBook의 문서나 변경 요청을 삭제하는 명령은 아니다.

```shell
codex mcp remove gitbook-mcp
```

## Codex에서 사용하는 방법

MCP를 등록했다고 Codex가 모든 요청에서 GitBook을 자동으로 수정하는 것은 아니다. 작업 목적과 변경 대상을
명시하고, 변경 작업은 change request 생성까지 요청한 뒤 GitBook에서 검토·병합한다.

### 문서 조사와 중복 확인

```text
GitBook MCP를 사용해 'OAuth callback' 관련 기존 문서를 찾아줘.
각 문서의 제목과 링크를 요약하고, 새 문서와 겹칠 가능성을 알려줘.
```

### 기존 문체에 맞춘 초안 만들기

```text
GitBook MCP로 기존 '인증' 문서의 제목 구조와 용어를 확인해 줘.
그 형식을 따라 JWT 갱신 토큰 문서 초안을 작성해 줘. 아직 GitBook에는 반영하지 마.
```

### 변경 요청으로 반영하기

```text
방금 합의한 초안을 GitBook의 <공간-이름>에 새 페이지로 만들고,
제목은 '갱신 토큰 사용 방법'으로 해 줘. 게시하지 말고 change request를 생성해 줘.
```

`<공간-이름>`은 실제 대상 공간 이름으로 교체한다. 페이지 생성, 구조 변경, 게시 설정 변경처럼 쓰기 권한이
필요한 요청은 대상 조직·공간·변경 범위·게시 여부를 함께 적어야 의도하지 않은 수정 가능성을 줄일 수 있다.

## 설정하면 달라지는 점

| 항목      | 설정 전                                             | 설정 후                                                                                      |
| --------- | --------------------------------------------------- | -------------------------------------------------------------------------------------------- |
| 문서 맥락 | 프롬프트에 붙인 내용과 모델의 일반 지식에 의존한다. | Codex가 연결된 GitBook 문서에서 최신 게시 내용을 검색·참조할 수 있다.                        |
| 문서 반영 | 초안을 복사해 GitBook에 수동으로 붙여 넣는다.       | 작업용 MCP와 권한이 있으면 페이지·change request 같은 GitBook 작업을 Codex에 요청할 수 있다. |
| 접근 범위 | GitBook 계정에 Codex 연결이 없다.                   | OAuth 또는 PAT가 허용한 GitBook 조직·공간 범위로 제한된다.                                   |
| 초안 노출 | GitBook에 직접 접근하지 않는다.                     | 게시 문서 MCP는 초안을 읽지 못한다. 작업용 MCP의 접근 범위는 인증 계정 권한에 따른다.        |

MCP는 GitBook을 Git 저장소처럼 자동 동기화하거나, 모든 초안을 자동 게시하는 기능이 아니다. 특히 이 저장소처럼
Git Sync를 사용하는 경우에도 로컬 Markdown 변경과 GitBook MCP 변경은 별도 경로일 수 있다. 변경 요청의
내용, 대상 공간, 병합·게시 결과를 GitBook에서 검토하는 운영 절차를 유지한다.

## 참고 자료

- [GitBook: AI와 MCP로 문서 작성·게시하기](https://www.gitbook.com/blog/create-documentation-with-ai-and-mcp)
- [GitBook: 게시 문서용 MCP 서버](https://gitbook.com/docs/ai-for-your-readers/mcp-servers-for-published-docs.md)
- [Codex CLI `mcp` 명령어](002_codex_command.md)
