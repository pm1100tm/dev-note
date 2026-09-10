# Git upstream 설정

upstream은 로컬 브랜치가 기본으로 pull하고 push할 원격 브랜치다. 이를 설정하면 매번
remote와 브랜치 이름을 쓰지 않아도 `git pull`, `git push`를 사용할 수 있다.

```shell
# 현재 브랜치를 origin/feature/login에 연결
git branch --set-upstream-to=origin/feature/login

# feature/local 브랜치를 origin/feature/remote에 연결
git branch --set-upstream-to=origin/feature/remote feature/local

# 새 브랜치를 push하면서 upstream 등록
git push -u origin feature/login
```

## 확인과 해제

```shell
git branch -vv
git rev-parse --abbrev-ref --symbolic-full-name '@{upstream}'
git branch --unset-upstream
```

- 기존 원격 브랜치를 추적하려면 먼저 `git fetch origin`으로 원격 참조를 갱신한다.
- 이름 변경 후에는 새 이름을 push하고 upstream을 다시 확인한다.
- upstream은 병합 방향을 강제하지 않는다. pull의 merge 또는 rebase 전략은 별도 설정이다.
