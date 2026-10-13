# git worktree로 여러 브랜치를 동시에 작업하기

기능 개발 중에 급한 버그 수정 요청이 들어오면 보통 작업 중인 변경을 `git stash`로 치워 두거나 임시 커밋을
만든 뒤 브랜치를 바꿉니다. 수정이 끝나면 다시 원래 브랜치로 돌아와 stash를 꺼내야 하고, 그사이 IDE 인덱싱이나
빌드 결과도 다시 만들어집니다.

`git worktree`는 이 전환 비용을 없애는 명령입니다. 하나의 저장소에 작업 폴더를 여러 개 연결해서,
브랜치마다 별도의 폴더를 열어 두고 동시에 작업할 수 있게 해 줍니다.

이 문서는 개념을 먼저 설명하고, 실습으로 워크트리를 생성·수정·삭제한 뒤 실무에서 사용할 때의 주의점을 정리합니다.

- 확인일: 2026-10-13
- 확인 환경: macOS, `git version 2.54.0 (Apple Git-157)`
- 공식 문서: [git-worktree](https://git-scm.com/docs/git-worktree)

## 개념

### 저장소와 작업 트리

Git으로 관리하는 프로젝트 폴더는 크게 두 부분으로 나눠 볼 수 있습니다.

| 구분 | 위치 | 담고 있는 것 |
| --- | --- | --- |
| 저장소(repository) | `.git/` 폴더 | 커밋, 브랜치, 태그, 원격 저장소 설정 등 이력 전체 |
| 작업 트리(working tree) | `.git/`을 제외한 프로젝트 파일 | 현재 체크아웃한 커밋의 파일을 실제로 꺼내 놓은 상태 |

`git clone`이나 `git init`으로 만든 폴더는 저장소 하나에 작업 트리 하나가 붙어 있습니다. 작업 트리는
한 번에 하나의 브랜치만 보여 줄 수 있기 때문에, 다른 브랜치를 보려면 `git switch`로 파일 전체를 바꿔야
합니다.

### 워크트리는 작업 트리를 하나 더 붙이는 것

`git worktree add`를 실행하면 같은 저장소에 새 작업 트리가 연결됩니다. 새 폴더에는 다른 브랜치를
체크아웃해 둘 수 있으므로, 폴더를 옮겨 다니는 것만으로 브랜치를 오갈 수 있습니다.

```shell
~/projects/
├── my-app/              # 메인 워크트리 (main 브랜치)
│   ├── .git/            # 저장소 본체: 커밋, 브랜치, 설정
│   └── app.py
└── my-app-login/        # 연결된 워크트리 (feature/login 브랜치)
    ├── .git             # 폴더가 아닌 파일. my-app/.git 위치를 가리킴
    └── app.py
```

- 메인 워크트리(main worktree): `git clone`이나 `git init`으로 처음 만든 작업
  폴더입니다.
- 연결된 워크트리(linked worktree): `git worktree add`로 추가한 작업 폴더입니다.

연결된 워크트리의 `.git`은 폴더가 아니라 한 줄짜리 파일입니다. 이 파일은 메인 저장소의
`.git/worktrees/<이름>/`을 가리키고, Git은 그 경로를 따라가 저장소 본체를 찾습니다. 그래서 저장소를
새로 clone하지 않아도 되고, 커밋 이력도 한 벌만 디스크에 저장됩니다.

### 공유하는 것과 따로 갖는 것

워크트리를 이해하는 핵심은 "이력은 공유하고, 작업 상태는 따로 가진다"는 점입니다.

| 구분 | 항목 | 의미 |
| --- | --- | --- |
| 공유 | 커밋 객체, 브랜치, 태그 | 한 워크트리에서 만든 커밋과 브랜치가 다른 워크트리에서도 바로 보임 |
| 공유 | 원격 저장소 설정, `git fetch` 결과 | 한 곳에서 fetch하면 모든 워크트리에 반영됨 |
| 공유 | 저장소 설정(`.git/config`), stash 목록 | `git stash list`는 모든 워크트리에서 같은 목록을 보여 줌 |
| 따로 | `HEAD` | 워크트리마다 체크아웃한 브랜치나 커밋이 다름 |
| 따로 | 인덱스(staging area) | `git add` 상태가 워크트리마다 독립적임 |
| 따로 | 작업 파일 | 수정 중인 파일은 해당 폴더에만 있음 |

한 가지 제약이 있습니다. 같은 브랜치는 동시에 두 워크트리에서 체크아웃할 수 없습니다. 두 폴더가 같은 브랜치를 각자
움직이면 어느 쪽 상태가 맞는지 알 수 없게 되기 때문에, Git이 처음부터 막습니다. 실습에서 이 오류를 직접
확인합니다.

### stash, clone과 비교

| 방법 | 브랜치 전환 방식 | 장점 | 단점 |
| --- | --- | --- | --- |
| `git stash` + `git switch` | 하나의 폴더에서 파일을 교체 | 추가 폴더가 필요 없음 | 전환할 때마다 변경을 치우고 되돌려야 함 |
| 저장소를 한 번 더 `git clone` | 완전히 별개의 저장소 | 서로 영향이 전혀 없음 | 이력을 두 벌 저장하고, 커밋·브랜치를 push/fetch로 주고받아야 함 |
| `git worktree` | 같은 저장소에 폴더를 추가 | 커밋과 브랜치가 즉시 공유되고 전환 비용이 없음 | 같은 브랜치를 두 곳에서 열 수 없고, 의존성 설치는 폴더마다 필요 |

## 주요 명령 요약

| 명령 | 용도 |
| --- | --- |
| `git worktree add <경로> <브랜치>` | 기존 브랜치를 새 폴더에 체크아웃 |
| `git worktree add -b <새 브랜치> <경로> [시작점]` | 새 브랜치를 만들면서 새 폴더에 체크아웃 |
| `git worktree add --detach <경로> <커밋>` | 브랜치 없이 특정 커밋을 체크아웃 |
| `git worktree list` | 연결된 워크트리 목록 확인 |
| `git worktree move <워크트리> <새 경로>` | 워크트리 폴더 위치 변경 |
| `git worktree lock` / `unlock` | 실수로 삭제·정리되지 않도록 잠금 / 해제 |
| `git worktree remove <워크트리>` | 워크트리 폴더와 관리 정보를 함께 삭제 |
| `git worktree prune` | 폴더가 이미 사라진 워크트리의 관리 정보 정리 |

`[시작점]`을 생략하면 현재 워크트리의 `HEAD`에서 새 브랜치를 만듭니다.

## 실습

빈 실습 폴더에서 저장소를 새로 만들고, 워크트리를 생성·수정·삭제하는 전체 흐름을 따라 합니다. 기존 프로젝트에는
영향을 주지 않습니다.

### 실습 준비

전제 조건은 다음과 같습니다.

- Git이 설치되어 있고, `git config --global user.name`, `user.email`이 설정되어
  있어야 커밋할 수 있습니다.
- `git init -b` 옵션은 Git 2.28 이상에서 사용할 수 있습니다. 이 문서의 출력은 Git
  2.54.0에서 확인했습니다.

```shell
git --version
```

홈 디렉터리 아래에 실습 폴더를 만들고, 그 안에 `my-app` 저장소를 만듭니다.

```shell
mkdir -p ~/git-worktree-lab
cd ~/git-worktree-lab

git init -b main my-app
cd my-app

echo "# My App" > README.md
echo 'print("hello")' > app.py
git add .
git commit -m "init"
```

이후 출력에 나오는 커밋 해시(`f1541a7` 등)와 `/Users/<사용자>` 부분은 실행 환경마다 다릅니다. 해시
값이 달라도 정상입니다.

### 워크트리 생성

현재 위치는 `~/git-worktree-lab/my-app`입니다. 로그인 기능을 개발한다고 가정하고,
`feature/login` 브랜치를 만들면서 옆 폴더 `my-app-login`에 체크아웃합니다.

```shell
git worktree add -b feature/login ../my-app-login
```

```shell
Preparing worktree (new branch 'feature/login')
HEAD is now at f1541a7 init
```

명령을 해석하면 다음과 같습니다.

- `-b feature/login`: 새 브랜치 이름입니다. 이미 있는 브랜치라면 `-b` 없이
  `git worktree add ../my-app-login feature/login`처럼 씁니다.
- `../my-app-login`: 새 워크트리 폴더 경로입니다. 아직 없는 폴더여야 합니다.
- 시작점을 생략했으므로 현재 `HEAD`인 `main`에서 브랜치가 만들어집니다.

목록을 확인합니다.

```shell
git worktree list
```

```shell
/Users/<사용자>/git-worktree-lab/my-app        f1541a7 [main]
/Users/<사용자>/git-worktree-lab/my-app-login  f1541a7 [feature/login]
```

첫 줄이 메인 워크트리, 둘째 줄이 방금 만든 연결된 워크트리입니다. 새 폴더의 `.git`이 파일인지도 확인해 봅니다.

```shell
cat ../my-app-login/.git
ls .git/worktrees/my-app-login
```

```shell
gitdir: /Users/<사용자>/git-worktree-lab/my-app/.git/worktrees/my-app-login
commondir  gitdir  HEAD  index  logs  ORIG_HEAD  refs
```

`.git/worktrees/my-app-login/` 안에 워크트리 전용 `HEAD`와 `index`가 있습니다.
앞에서 설명한 "따로 갖는 것"이 실제로 여기에 저장됩니다.

### 워크트리에서 파일 수정과 커밋

새 워크트리로 이동합니다. 별도의 `git switch` 없이 이미 `feature/login` 브랜치입니다.

```shell
cd ../my-app-login
git branch --show-current
```

```shell
feature/login
```

파일을 추가하고 커밋합니다. 평소 하던 Git 작업과 완전히 같습니다.

```shell
echo 'def login(): pass' > login.py
git add login.py
git commit -m "feat: login"
```

이 커밋이 메인 워크트리에서도 보이는지 확인합니다. `-C <경로>` 옵션은 해당 폴더로 이동하지 않고 그 위치에서
Git 명령을 실행합니다.

```shell
git -C ../my-app branch -v
```

```shell
+ feature/login 865c1d6 feat: login
* main          f1541a7 init
```

`*`는 명령을 실행한 워크트리(`my-app`)에서 체크아웃한 브랜치이고, `+`는 다른 워크트리에서 체크아웃 중인
브랜치를 뜻합니다. push나 fetch 없이 커밋이 바로 공유된 것을 확인할 수 있습니다.

반대로 작업 파일은 공유되지 않습니다. 메인 워크트리에는 아직 `login.py`가 없습니다.

```shell
ls ../my-app
```

```shell
README.md  app.py
```

### 메인 워크트리에서 작업 결과 반영

메인 워크트리로 돌아가 `feature/login`을 병합합니다. 이 실습에서는 원격 저장소 없이 로컬에서 병합하지만,
실무에서는 이 단계가 보통 push와 Pull Request로 대체됩니다.

```shell
cd ../my-app
git merge feature/login
```

```shell
Updating f1541a7..865c1d6
Fast-forward
 login.py | 1 +
 1 file changed, 1 insertion(+)
 create mode 100644 login.py
```

### 같은 브랜치를 두 곳에서 열 수 없음 확인

메인 워크트리에서 `feature/login`으로 전환해 봅니다.

```shell
git switch feature/login
```

```shell
fatal: 'feature/login' is already used by worktree at '/Users/<사용자>/git-worktree-lab/my-app-login'
```

다른 워크트리가 이미 사용 중이라 전환이 거부됩니다. `git worktree add`로 같은 브랜치를 다른 폴더에
체크아웃하려 해도 같은 오류가 납니다.

브랜치 없이 특정 커밋만 확인하고 싶다면 `--detach`를 사용합니다. 코드 리뷰나 과거 버전 확인처럼 커밋할 필요가
없는 경우에 유용합니다.

```shell
git worktree add --detach ../my-app-review HEAD
git worktree list
```

```shell
/Users/<사용자>/git-worktree-lab/my-app         865c1d6 [main]
/Users/<사용자>/git-worktree-lab/my-app-login   865c1d6 [feature/login]
/Users/<사용자>/git-worktree-lab/my-app-review  865c1d6 (detached HEAD)
```

### 워크트리 위치 변경과 잠금

워크트리 폴더를 옮길 때는 `mv` 대신 `git worktree move`를 사용합니다. `mv`로 옮기면 `.git`
파일과 관리 정보의 경로가 어긋납니다.

```shell
git worktree move ../my-app-login ../my-app-auth
git worktree list
```

```shell
/Users/<사용자>/git-worktree-lab/my-app         865c1d6 [main]
/Users/<사용자>/git-worktree-lab/my-app-auth    865c1d6 [feature/login]
/Users/<사용자>/git-worktree-lab/my-app-review  865c1d6 (detached HEAD)
```

새 경로의 상위 폴더는 미리 있어야 합니다. 예를 들어 `../wt/login`으로 옮기려면 `wt` 폴더를 먼저
만들어야 하고, 없으면 `No such file or directory` 오류가 납니다.

외장 디스크나 네트워크 드라이브처럼 가끔 연결이 끊기는 위치의 워크트리는 잠가 둘 수 있습니다. 잠긴 워크트리는
`remove`, `move`, `prune`의 대상에서 보호됩니다.

```shell
git worktree lock --reason "외장 디스크 작업" ../my-app-auth
git worktree list --verbose
```

```shell
/Users/<사용자>/git-worktree-lab/my-app         865c1d6 [main]
/Users/<사용자>/git-worktree-lab/my-app-auth    865c1d6 [feature/login]
	locked: 외장 디스크 작업
/Users/<사용자>/git-worktree-lab/my-app-review  865c1d6 (detached HEAD)
```

잠긴 상태에서 삭제하면 거부되는 것을 확인한 뒤 잠금을 해제합니다.

```shell
git worktree remove ../my-app-auth
```

```shell
fatal: cannot remove a locked working tree, lock reason: 외장 디스크 작업
use 'remove -f -f' to override or unlock first
```

```shell
git worktree unlock ../my-app-auth
```

### 워크트리 삭제

작업이 끝난 워크트리는 `git worktree remove`로 삭제합니다. 폴더와 `.git/worktrees/`
아래의 관리 정보가 함께 지워집니다.

```shell
git worktree remove ../my-app-auth
```

정상적으로 삭제되면 아무것도 출력되지 않습니다. 이때 삭제되는 것은 워크트리 폴더이고, 브랜치와 커밋은 저장소에 그대로
남아 있습니다.

커밋하지 않은 변경이나 추적하지 않는 파일이 있으면 Git이 삭제를 거부합니다. 리뷰용 워크트리에 메모 파일을 만들어
확인해 봅니다.

```shell
echo "임시 메모" > ../my-app-review/memo.txt
git worktree remove ../my-app-review
```

```shell
fatal: '../my-app-review' contains modified or untracked files, use --force to delete it
```

`memo.txt`가 필요 없는 파일임을 확인했으므로 `--force`로 삭제합니다. `--force`는 커밋하지 않은
변경을 복구할 수 없게 지우므로, 실무에서는 먼저 해당 워크트리에서 `git status`로 남은 변경을 확인해야
합니다.

```shell
git worktree remove --force ../my-app-review
```

### 워크트리를 지운 뒤 브랜치 삭제

워크트리가 사용 중인 브랜치는 삭제할 수 없습니다. 만약 `my-app-auth`를 지우기 전에 브랜치를 지우려고
`git branch -d feature/login`을 실행했다면 다음 오류가 났을 것입니다.

```shell
error: cannot delete branch 'feature/login' used by worktree at '/Users/<사용자>/git-worktree-lab/my-app-auth'
```

지금은 워크트리를 삭제했으므로 브랜치도 삭제할 수 있습니다.

```shell
git branch -d feature/login
git worktree list
```

```shell
Deleted branch feature/login (was 865c1d6).
/Users/<사용자>/git-worktree-lab/my-app  865c1d6 [main]
```

참고로 메인 워크트리는 `git worktree remove`로 삭제할 수 없습니다. 메인 워크트리에서
`git worktree remove .`를 실행하면 `fatal: '.' is a main working tree`
오류가 납니다.

### 폴더를 직접 지웠을 때 정리

`git worktree remove` 대신 `rm -rf`나 Finder로 워크트리 폴더를 직접 지우면
`.git/worktrees/` 아래의 관리 정보가 그대로 남습니다. 일부러 이 상황을 만들어 봅니다.

```shell
git worktree add -b tmp ../my-app-tmp
rm -rf ../my-app-tmp
git worktree list
```

```shell
/Users/<사용자>/git-worktree-lab/my-app      865c1d6 [main]
/Users/<사용자>/git-worktree-lab/my-app-tmp  865c1d6 [tmp] prunable
```

`prunable`은 폴더가 사라져 정리할 수 있는 워크트리라는 표시입니다. 폴더는 없지만 관리 정보가 남아 있어서,
`tmp` 브랜치는 여전히 사용 중으로 취급됩니다.

```shell
git branch -d tmp
```

```shell
error: cannot delete branch 'tmp' used by worktree at '/Users/<사용자>/git-worktree-lab/my-app-tmp'
```

`--dry-run`으로 정리 대상을 먼저 확인한 뒤 `prune`을 실행합니다. 두 명령 모두 아래와 같은 줄을
출력하지만, `--dry-run`은 실제로 지우지 않고 대상만 보여 줍니다.

```shell
git worktree prune --dry-run --verbose
git worktree prune --verbose
```

```shell
Removing worktrees/my-app-tmp: gitdir file points to non-existent location
```

Git은 `gc` 실행 시 오래된 관리 정보를 자동으로 정리하기도 하지만, 즉시 정리하려면 `prune`을 직접
실행합니다. 정리가 끝나면 브랜치를 삭제할 수 있습니다.

```shell
git branch -d tmp
```

```shell
Deleted branch tmp (was 865c1d6).
```

### 실습 정리

실습 폴더 전체를 지웁니다. 경로를 잘못 입력하면 다른 파일이 삭제될 수 있으므로, `ls`로 대상이 실습 폴더인지
확인한 뒤 실행합니다.

```shell
cd ~
ls ~/git-worktree-lab
rm -rf ~/git-worktree-lab
```

## 실무 적용

### 자주 쓰는 상황

- 기능 개발 중 긴급 수정: 작업 폴더는 그대로 두고 `main`에서 hotfix 워크트리를 만들어 수정·배포한 뒤
  삭제합니다.
- Pull Request 리뷰: 리뷰할 브랜치를 별도 폴더에 체크아웃해 실행·테스트해 봅니다. 내 작업 폴더의 상태가
  섞이지 않습니다.
- 오래 걸리는 빌드·테스트: 한 워크트리에서 테스트를 돌리는 동안 다른 워크트리에서 다음 작업을 계속합니다.
- 버전 비교: 과거 태그를 `--detach`로 체크아웃해 현재 코드와 나란히 실행해 봅니다.
- AI 코딩 에이전트 병렬 작업: 에이전트마다 워크트리를 나눠 주면 같은 저장소에서 여러 작업을 동시에 진행해도 파일이
  서로 덮어써지지 않습니다.

### 긴급 수정 흐름 예시

원격 저장소 `origin`이 있는 실제 프로젝트에서 hotfix를 처리하는 흐름입니다. 브랜치 이름
`hotfix/payment-error`와 폴더 경로는 예시 값이므로 상황에 맞게 바꿉니다.

```shell
# 메인 워크트리(기능 개발 중인 폴더)에서 최신 원격 상태를 가져옴
git fetch origin

# origin/main에서 hotfix 브랜치를 만들어 옆 폴더에 체크아웃
git worktree add -b hotfix/payment-error ../my-app-hotfix origin/main

# hotfix 폴더에서 수정, 커밋, push
cd ../my-app-hotfix
git add .
git commit -m "fix: 결제 오류 수정"
git push -u origin hotfix/payment-error

# Pull Request 병합 후 원래 폴더로 돌아와 정리
cd ../my-app
git worktree remove ../my-app-hotfix
git branch -d hotfix/payment-error
```

원격에만 있는 브랜치를 체크아웃할 때는 브랜치 이름만 적어도 됩니다. 로컬에 같은 이름의 브랜치가 없고 원격 하나에서만
일치하는 브랜치가 있으면, Git이 원격 브랜치를 추적하는 로컬 브랜치를 만들어 줍니다. 이 동작의 조건은 [공식
문서](https://git-scm.com/docs/git-worktree)의 `add` 설명에 있습니다.

```shell
git fetch origin
git worktree add ../my-app-review feature/some-branch
```

```shell
Preparing worktree (new branch 'feature/some-branch')
branch 'feature/some-branch' set up to track 'origin/feature/some-branch'.
HEAD is now at <커밋 해시> <커밋 메시지>
```

### 주의할 점

- `.gitignore`에 등록된 파일은 새 워크트리에 생기지 않습니다. `.env`, `node_modules`,
  `.venv`, 빌드 결과물은 워크트리마다 다시 만들거나 복사해야 합니다.
- 반대로 `git worktree remove`는 무시된 파일만 있는 워크트리를 `--force` 없이 삭제합니다.
  다시 만들기 어려운 로컬 설정 파일은 워크트리 안에만 두지 않습니다.
- 워크트리마다 의존성을 설치하면 디스크 사용량이 늘어납니다. 다 쓴 워크트리는 바로 삭제하는 습관을 들이는 편이
  좋습니다.
- 여러 워크트리에서 개발 서버를 동시에 띄우면 포트가 충돌할 수 있습니다.
- stash 목록과 저장소 설정은 공유됩니다. 다른 워크트리에서 만든 stash를 실수로 꺼내지 않도록, 워크트리
  환경에서는 stash보다 임시 커밋을 쓰는 편이 안전합니다.
- 워크트리를 메인 워크트리 안에 만들면 상위 저장소에서 추적하지 않는 폴더로 보일 수 있습니다.
  `../<저장소명>-<작업명>`처럼 저장소 바깥의 형제 폴더를 사용하는 편이 관리하기 쉽습니다.
- IDE는 워크트리 폴더를 별도 프로젝트로 엽니다. 실행 설정이나 인덱싱도 폴더마다 따로 잡힙니다.

## 정리

- `git worktree`는 하나의 저장소에 작업 폴더를 여러 개 연결하는 기능입니다.
- 커밋, 브랜치, 원격 설정은 공유하고 `HEAD`, 인덱스, 작업 파일은 워크트리마다 따로 가집니다.
- 생성은 `add`, 확인은 `list`, 위치 변경은 `move`, 삭제는 `remove`, 남은 정보 정리는
  `prune`을 사용합니다.
- 같은 브랜치는 두 워크트리에서 동시에 체크아웃할 수 없고, 워크트리를 지워야 그 브랜치를 삭제할 수 있습니다.
