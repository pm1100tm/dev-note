# Git Repository와 GitBook 연동

> 확인일: 2026-09-10. GitBook 공식 문서의 현재 화면을 기준으로 작성했다. 계정에 따라 메뉴 위치가 다를 수 있다.

## 목표와 전체 흐름

새 GitHub 저장소와 GitBook 사이트를 처음 연결하는 과정을 정리한다. 예제 저장소 이름은 `developer-note`,
연결할 브랜치는 `main`으로 통일한다. 문서 파일을 직접 만든 뒤 GitBook에 가져오고, 사이트를 게시한 다음
push한 변경이 웹사이트에 반영되는 것까지 확인한다.

```text
로컬 문서 수정 → commit → GitHub main에 push
                               ↓ Git Sync
                         GitBook 문서 갱신
                               ↓ 게시된 사이트에 반영
                         방문자용 URL에서 확인
```

- Git Sync 연결과 사이트 최초 게시를 모두 완료해야 한다.
- 연결만 해 놓은 미게시 사이트는 방문자가 볼 수 없다.
- 게시 후에는 연결된 브랜치의 변경이 동기화되면 사이트도 갱신된다.

[공식 Quickstart](https://gitbook.com/docs/getting-started/quickstart)

## Step 1. GitHub 저장소와 예제 문서 만들기

### 1-1. 저장소 생성 및 clone

- GitHub에 로그인하고 **New repository**를 선택한다.
- Repository name에 `developer-note`를 입력하고 원하는 공개 범위를 선택한다. (\*레포지토리명은 마음대로)
- README 추가 옵션을 켜고 **Create repository**를 누른다.
- 생성된 저장소의 **Code**에서 HTTPS 주소를 복사한다.
- 터미널에서 저장소를 clone한다. `<github-account>`는 실제 저장소 소유 계정 또는 조직 이름으로 바꾼다.

```shell
git clone https://github.com/<github-account>/developer-note.git
cd developer-note
git branch --show-current
```

- 이후 예시는 기본 브랜치가 `main`인 경우다.
- 다른 브랜치를 사용한다면 명령의 `main`과 GitBook에서 선택할 브랜치를 동일하게 맞춘다.
- 이미 준비한 저장소를 처음 연결하는 경우에는 생성과 clone을 생략하고 해당 로컬 저장소에서 진행한다.

### 1-2. 문서와 목차 작성

편집기에서 저장소 루트에 다음 구조로 파일을 준비한다. 기존 파일이 있다면 필요한 내용을 반영하고 보존한다.

```text
developer-note/
├── README.md
├── SUMMARY.md
├── getting-started.md
└── gitbook-docs.yaml
```

`README.md`에는 첫 페이지 내용을 작성한다.

```markdown
# 개발 노트

개발하면서 배운 내용을 정리하는 문서 사이트입니다.
```

`getting-started.md`를 새로 만들고 다음을 작성한다.

```markdown
# 시작하기

GitHub와 GitBook을 연결해 문서를 관리합니다.
```

`SUMMARY.md`를 새로 만들고 두 문서를 목차에 등록한다.

```markdown
# Summary

- [개발 노트](README.md)
- [시작하기](getting-started.md)
```

| 파일                 | 역할                               |
| -------------------- | ---------------------------------- |
| `README.md`          | 첫 화면의 문서                     |
| `getting-started.md` | 동기화와 게시를 확인할 예제 문서   |
| `SUMMARY.md`         | 문서 목차와 페이지 경로            |
| `gitbook-docs.yaml`  | 사이트 구조와 콘텐츠 디렉터리 매핑 |

### 1-3. 사이트 구성 파일 작성

저장소 루트에 `gitbook-docs.yaml`을 만들고 다음을 작성한다. 이 예시는 하나의 Space에서 루트의 문서를 읽는다.

```yaml
$schema: https://api.gitbook.com/gitbook-docs.yaml
site:
  title: developer-note
  structure:
    - type: space
      key: space-1
      title: 개발 노트
      path: dev-log
      default: true
      content:
        directory: ./
```

각 항목의 의미는 다음과 같다.

| 항목                | 설명                                                                                                  |
| ------------------- | ----------------------------------------------------------------------------------------------------- |
| `$schema`           | 이 설정 파일의 구조를 정의하는 스키마 주소다. 지원하는 편집기에서 자동 완성과 유효성 검사에 사용한다. |
| `site`              | GitBook 사이트 전체의 설정을 묶는 항목이다.                                                           |
| `site.title`        | 사이트의 이름이다.                                                                                    |
| `site.structure`    | 사이트를 구성하는 Space나 섹션을 나열하는 목록이다. `-`로 각 항목을 구분한다.                         |
| `type`              | 이 항목이 문서 묶음인 Space임을 나타낸다.                                                             |
| `key`               | 동기화할 때 기존 Space를 식별하는 고유 키다. 연결 후 값을 바꾸면 새 Space로 처리될 수 있다.           |
| `title`             | 해당 Space의 표시 이름이다.                                                                           |
| `path`              | 사이트에서 해당 Space의 URL을 구성하는 경로(slug)다. 저장소 폴더 경로와는 별개이다.                   |
| `default`           | 해당 Space를 기본으로 사용할지 지정한다.                                                              |
| `content`           | 해당 Space에 동기화할 콘텐츠의 설정을 묶는 항목이다.                                                  |
| `content.directory` | 문서를 가져올 디렉터리다. `./`는 Project directory 기준이며, 이 예시에서는 저장소 루트다.             |

- `key: space-1`은 연결 후 임의로 바꾸지 않는다. GitBook은 이 값으로 기존 Space를 식별한다.
- `.gitbook.yaml`은 개별 Space의 읽기 설정이며 `gitbook-docs.yaml`과 역할이 다르다.
- 이 예시처럼 루트의 `README.md`와 `SUMMARY.md`를 쓰면 기본값으로 동작하므로 추가 생성할 필요가 없다.
  명시적으로 설정할 때의 예시는 다음과 같다.

```yaml
root: ./
structure:
  readme: README.md
  summary: SUMMARY.md
```

[공식 콘텐츠 설정 안내](https://gitbook.com/docs/docs-as-code/git-sync/content-configuration)

### 1-4. GitHub에 최초 문서 올리기

작성한 파일을 저장한 뒤 저장소 디렉터리에서 실행한다.

```shell
git add README.md SUMMARY.md getting-started.md gitbook-docs.yaml
git commit -m "docs: add initial GitBook documentation"
git push origin main
```

GitHub 웹 화면에서 `main`에 네 파일과 작성한 내용이 보이는지 확인한다. GitBook은 GitHub에 올라간 내용을
가져오므로, 이 단계를 완료한 뒤 연결을 시작한다.

## Step 2. GitBook 사이트 만들기

- [GitBook](https://app.gitbook.com/)에 로그인한다.
- 조직의 Home에서 사이드바 **Sites 옆 +**를 누른다.
- 사이트 이름을 입력하고 **Create**를 누른다. 예: `developer-note`.
- 시작 방식에서 **Sync with Git**을 선택한다.

Site는 방문자용 웹사이트이고, Space는 그 안에 연결하는 문서 묶음이다.

[공식 Quickstart](https://gitbook.com/docs/getting-started/quickstart)

## Step 3. GitHub 계정과 저장소 연결하기

- 사이트 사이드바의 **Git Sync**를 연다.
- GitHub 연결을 선택하고 대상 저장소에 접근 가능한 GitHub 계정으로 인증한다.
- App 설치가 필요하면 [GitBook GitHub App](https://github.com/apps/gitbook-com)을 저장소 소유 계정
  또는 조직에 설치한다. 특정 저장소만 허용할 경우 이 저장소를 선택한다.
- **Source repository**에서 저장소를 선택하고 브랜치를 **main**으로 지정한다.

이 안내는 사이트 단위 Git Sync를 사용한다. 시작 과정에서 이미 Git Sync 설정 화면이 열렸다면 그 화면에서 계속 진행한다.

[공식 GitHub 연결 안내](https://gitbook.com/docs/docs-as-code/git-sync/enabling-github-sync)

## Step 4. 최초 동기화 방향과 경로 지정하기

- 최초 방향을 **GitHub → GitBook**으로 맞춘다. 반대 방향이면 **Swap direction**으로 전환한다.
- **Project directory**는 예제 저장소의 루트를 사용하도록 비워 둔다.
- **Content mapping**에서 대상 Space의 디렉터리를 `./`로 확인한다.
- 저장소, `main`, 방향, 경로를 확인하고 **Sync**를 누른다.

최초 동기화는 저장소 내용을 GitBook으로 가져온다. 방향을 반대로 선택하면 GitBook 콘텐츠를 저장소로 내보내므로
**GitHub → GitBook**인지 확인한다. 이후 Git Sync는 양방향으로 작동하며, GitBook에서 변경 요청을 병합하면
GitHub에도 커밋이 생성될 수 있다.

[공식 동기화 절차](https://gitbook.com/docs/docs-as-code/git-sync/enabling-github-sync)

## Step 5. 최초 동기화 결과 확인하기

- Git Sync가 완료되고 오류가 없는지 확인한다.
- 사이트의 **Content**에서 연결된 Space를 연다.
- 첫 페이지에 Step 1에서 작성한 **개발 노트** 제목과 소개 문장이 보이는지 확인한다.
- 목차의 **시작하기**를 선택한다.
- `getting-started.md`에 작성한 **GitHub와 GitBook을 연결해 문서를 관리합니다.** 문장이 표시되는지 확인한다.

이 단계에서는 Step 1에서 만든 `SUMMARY.md`의 두 항목이 목차에 나타나고, 각각 올바른 본문을 여는지 확인한다.
아직 사이트를 게시하지 않았으므로 GitBook 편집 화면에서 확인하면 된다. 이미지를 추가했다면 이미지도 함께 확인한다.

문서가 보이지 않으면 `SUMMARY.md`에 해당 파일이 등록되어 있는지, 링크 경로와 실제 파일명이 일치하는지 확인한다.

[공식 문제 해결 안내](https://gitbook.com/docs/docs-as-code/git-sync/troubleshooting)

## Step 6. 사이트 게시하기

- 사이트의 **Preview**에서 방문자 화면을 확인한다.
- **Settings → Audience**에서 원하는 방문자 공개 범위를 확인한다. 누구나 읽는 사이트라면 공개 범위를 선택한다.
- 사이트 헤더의 **Publish**를 누른다.
- 게시 후 **Overview**에 나타나는 실제 사이트 링크를 연다.

GitHub 저장소 접근 권한과 사이트 방문자의 공개 범위는 별도 설정이므로, 게시할 콘텐츠와 Audience를 함께 확인한다.

[공식 사이트 설정](https://gitbook.com/docs/manage-your-site/site-settings)

기본 제공 주소는 다음 형태다. 아래 주소는 형식 예시이며 실제 주소는 Overview에서 복사한다.

```text
https://<organization-name>.gitbook.io/<site-title>
```

## Step 7. 호스팅된 웹사이트 보기

- Overview에서 복사한 사이트 URL을 새 브라우저 탭에 붙여 넣는다.
- 공개 사이트라면 시크릿 창에서도 열어 로그인 없이 읽을 수 있는지 확인한다.
- 목차에서 **시작하기**를 열어 Step 1의 예제 문장이 표시되는지 확인한다.
- 이후에는 이 URL을 즐겨찾기하거나 독자에게 공유한다.

`app.gitbook.com` 편집 화면 주소와 Preview 주소 대신 **게시된 사이트의 URL**을 공유한다. 별도 도메인이
필요하면 **Settings → Domain and URL**에서 설정할 수 있다.

[공식 게시 및 URL 안내](https://gitbook.com/docs/getting-started/quickstart)

## Step 8. push 후 자동 반영 확인하기

현재 작업을 먼저 commit하거나 보관해 작업 트리를 정리한 뒤 시작한다. GitBook이 생성한 원격 커밋을 먼저 가져오기 위해
다음을 실행한다.

```shell
git switch main
git pull --ff-only origin main
```

`getting-started.md`의 본문에 `동기화 확인: 첫 번째 테스트`라는 문장을 추가하고 저장한다. 이어서 실행한다.

```shell
git add getting-started.md
git commit -m "docs: verify GitBook synchronization"
git push origin main
git log -1 --oneline
```

다음 순서로 확인한다.

- GitHub `main`에서 방금 push한 커밋과 문장을 확인한다.
- GitBook의 Git Sync 상태와 해당 문서에서 반영 여부를 확인한다.
- 게시된 사이트 URL을 새로고침해 같은 문장이 보이는지 확인한다.

최초 게시가 끝난 사이트에서는 일반적인 문서 변경마다 Publish를 다시 누를 필요가 없다. 기능 브랜치에 push했다면 연결된
`main`에 PR을 병합해야 운영 사이트에 반영된다.

[공식 변경 반영 안내](https://gitbook.com/docs/getting-started/quickstart)

테스트 문장을 지운 뒤에도 같은 commit/push 순서로 반영할 수 있다.

## Step 9. 이후 새 문서 추가하기

- 저장소 루트에 `second-note.md`를 만들고 제목과 본문을 작성한다.
- `SUMMARY.md`의 기존 목차 아래에 다음 항목을 추가한다.

```markdown
- [두 번째 노트](second-note.md)
```

- 문서와 목차를 함께 commit하고 push한다.

```shell
git add second-note.md SUMMARY.md
git commit -m "docs: add second note"
git push origin main
```

- 동기화가 끝나면 게시된 사이트를 새로고침하고 **두 번째 노트**가 목차와 본문에 표시되는지 확인한다.

목차가 있는 저장소에서는 파일만 추가하고 `SUMMARY.md` 등록을 빠뜨리면 새 문서가 나타나지 않을 수 있다.

[공식 문제 해결 안내](https://gitbook.com/docs/docs-as-code/git-sync/troubleshooting)

## 문제가 생겼을 때 확인 순서

| 증상                                 | 확인 및 조치                                                                                                                                         |
| ------------------------------------ | ---------------------------------------------------------------------------------------------------------------------------------------------------- |
| 저장소가 선택 목록에 없음            | GitBook App의 설치 계정·조직과 허용 저장소를 확인한다.                                                                                               |
| push했는데 GitBook에 반영되지 않음   | 연결 브랜치와 실제 push 브랜치, Project directory, Content mapping을 비교한다.                                                                       |
| 새 문서만 보이지 않음                | `SUMMARY.md` 등록 여부와 실제 경로·대소문자를 확인한다.                                                                                              |
| GitBook에는 보이지만 웹사이트에 없음 | 해당 콘텐츠가 사이트에 연결되어 있는지, 게시 상태인지, 실제 사이트 URL을 열었는지 확인한다.                                                          |
| protected branch 권한 오류           | 저장소 관리자가 브랜치 규칙에서 `gitbook-com` App의 필요한 우회 권한을 설정해야 한다. 일반 작성자의 PR 사용만으로 App 권한 문제가 해결되지는 않는다. |
| Git Sync에 오류 또는 실패 표시       | 상세 오류와 실패 커밋을 확인하고 원인을 수정한 새 커밋을 push한다.                                                                                   |
| `git pull --ff-only` 실패            | 로컬·원격 커밋 분기를 확인하고 팀의 merge/rebase 방식으로 정리한 후 진행한다.                                                                        |

Git Sync는 초기 설정 때도 App의 저장소 쓰기 권한이 필요할 수 있다. 반복되는 동기화 오류는 오류 메시지와 커밋 해시를 함께 기록해 조사한다. [공식 Git Sync 문제 해결](https://gitbook.com/docs/docs-as-code/git-sync/troubleshooting)

## 보안 주의사항

- GitBook App에는 필요한 저장소만 접근하도록 제한한다.
- API 키, 개인 키, `.env` 파일, 고객 데이터는 Markdown과 Git history에 커밋하지 않는다.
- 실수로 비밀값을 push했다면 파일 삭제만 하지 말고 즉시 키를 폐기·교체한다.

## 공식 문서

- [GitHub Sync 활성화](https://gitbook.com/docs/docs-as-code/git-sync/enabling-github-sync)
- [Git Sync 문제 해결](https://gitbook.com/docs/docs-as-code/git-sync/troubleshooting)
