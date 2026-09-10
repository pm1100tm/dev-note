# Git Repository와 GitBook 연동

GitBook Git Sync는 GitHub 저장소의 Markdown과 GitBook Space를 양방향으로 동기화하는 기능이다.
`main` 브랜치에 커밋을 push하면 GitBook이 해당 커밋을 가져와 문서와 목차를 갱신한다.

## 동작 구조

```text
GitHub main ──push──> GitBook Git Sync ──import──> GitBook Space ──publish──> 공개 문서
```

- GitHub에서 push한 커밋은 GitBook의 history commit으로 동기화된다.
- GitBook에서 변경 요청을 병합하면, 선택한 GitHub 브랜치에 커밋이 생성될 수 있다.
- GitBook은 GitHub Actions가 아니라 GitBook GitHub App의 권한으로 저장소에 접근한다.

## 사전 조건

- GitBook Space의 관리자 또는 Creator 권한이 필요하다.
- GitHub 저장소에 GitBook GitHub App이 설치되어 있어야 한다.
- GitBook에서 선택한 저장소와 브랜치가 실제 push 대상과 같아야 한다.
- `SUMMARY.md`에 연결한 파일 경로와 실제 파일 경로가 일치해야 한다.

## GitHub 저장소와 연결하기

### 1. GitBook Space 열기

- GitBook에서 동기화할 Space를 연다.
- 우측 상단 Space 메뉴에서 `Configure`를 선택한다.
- provider 목록에서 `GitHub Sync`를 선택한다.

### 2. GitHub 계정 인증하기

- GitHub 계정 인증 화면이 나타나면 동기화할 저장소에 접근 가능한 계정으로 로그인한다.
- 조직 저장소라면 해당 조직에 GitBook App을 설치할 권한이 있는지 확인한다.
- 이미 다른 GitBook 계정에 연결된 GitHub 계정이면 중복 계정 오류가 날 수 있다.

### 3. GitBook GitHub App 권한 설정하기

- GitHub App 설치 화면에서 저장소 접근 범위를 선택한다.
- 특정 저장소만 허용한다면 대상 저장소를 반드시 선택한다.
- GitHub 조직의 `Settings > GitHub Apps > GitBook`에서 저장소 접근 범위를 다시 확인할 수 있다.

### 4. 저장소와 브랜치 선택하기

- GitBook에서 GitHub 계정 또는 조직, 저장소, 브랜치를 차례로 선택한다.
- 일반적으로 문서를 관리하는 `main` 브랜치를 선택한다.
- 보호 브랜치 정책이 GitBook의 push를 막는지 확인한다.

### 5. 최초 동기화 방향 선택하기

- GitHub → GitBook: 저장소에 이미 있는 Markdown 문서를 GitBook으로 가져온다.
- GitBook → GitHub: GitBook Space의 기존 문서를 빈 저장소 또는 대상 브랜치에 보낸다.
- 기존 문서 저장소가 있다면 보통 `GitHub → GitBook`을 선택한다.

### 6. 목차와 이미지 확인하기

- 저장소 루트의 `SUMMARY.md`가 GitBook 목차의 기준이다.
- 링크 경로는 `SUMMARY.md` 기준의 상대 경로로 작성하고, 공백·괄호가 있는 경로는 `<...>`로 감싼다.
- Markdown 이미지 경로도 문서 파일 기준의 상대 경로가 실제 파일을 가리키는지 확인한다.

## `x` 상태가 보일 때 점검 순서

`x`는 대개 해당 커밋을 GitBook이 가져오거나 해석하는 과정에서 오류가 났다는 뜻이다. 정확한
오류 메시지는 GitBook Space의 Git Sync 상태 또는 동기화 history에서 먼저 확인한다.

### 1. 오류 메시지와 실패 커밋 확인

- GitBook Space의 `Configure > GitHub Sync` 또는 동기화 상태 화면을 연다.
- 실패한 커밋 해시, 시간, 오류 메시지를 확인한다.
- GitHub의 해당 커밋과 GitBook이 선택한 브랜치가 같은지 확인한다.

### 2. GitHub App 권한 확인

- GitHub App이 저장소에 설치되어 있고 해당 저장소 접근 권한이 있는지 확인한다.
- 조직 저장소라면 조직 정책이나 SSO 승인이 App 접근을 막지 않는지 확인한다.
- 권한을 변경한 뒤 GitBook에서 연결을 다시 인증하거나 재연결한다.

### 3. `SUMMARY.md`와 파일 경로 확인

```shell
# SUMMARY.md에 적힌 링크 대상이 실제로 있는지 확인하는 예시
rg -n '\]\(' SUMMARY.md
git status --short
```

- `SUMMARY.md`의 링크가 삭제·이동된 파일을 가리키면 import가 실패하거나 목차가 깨질 수 있다.
- 대소문자가 다른 경로는 macOS에서는 지나가도 Linux 기반 서비스에서 실패할 수 있다.
- 파일명에 공백 또는 괄호가 있으면 Markdown 링크의 대상 경로를 `<...>`로 감싼다.

### 4. 문제를 수정하고 새 커밋으로 재시도

```shell
git add SUMMARY.md path/to/fixed-file.md
git commit -m "docs: fix GitBook sync paths"
git push origin main
```

GitBook은 새 커밋이 push되면 import를 다시 시도한다. 같은 오류가 반복되면 오류 메시지와
실패 커밋 해시를 포함해 GitBook 지원팀 또는 조직 관리자에게 문의한다.

## 흔한 원인과 해결

| 증상 | 주된 원인 | 조치 |
| --- | --- | --- |
| 저장소가 목록에 없음 | App의 저장소 권한 또는 설치 범위 문제 | GitHub App의 repository access를 수정한다. |
| 인증 실패 | GitHub 계정 연결 또는 조직 SSO 문제 | GitBook에서 GitHub를 다시 인증하고 SSO를 승인한다. |
| push 후 동기화 없음 | 다른 브랜치를 보고 있거나 App webhook·권한 문제 | GitBook의 선택 브랜치와 App 권한을 확인한다. |
| 목차·페이지 오류 | `SUMMARY.md` 링크 또는 파일 경로 오류 | 실제 파일명·대소문자·상대 경로를 수정한다. |
| GitBook의 push 실패 | 보호 브랜치가 App의 push를 차단 | 브랜치 규칙에 GitBook App을 허용하거나 PR 흐름을 사용한다. |

## 보안 주의사항

- GitBook App에는 필요한 저장소만 접근하도록 제한한다.
- API 키, 개인 키, `.env` 파일, 고객 데이터는 Markdown과 Git history에 커밋하지 않는다.
- 실수로 비밀값을 push했다면 파일 삭제만 하지 말고 즉시 키를 폐기·교체한다.

## 공식 문서

- [GitHub Sync 활성화](https://gitbook.com/docs/getting-started/git-sync/enabling-github-sync)
- [Git Sync 문제 해결](https://gitbook.com/docs/getting-started/git-sync/troubleshooting)
