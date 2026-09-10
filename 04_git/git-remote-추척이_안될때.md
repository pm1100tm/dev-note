# 원격 추적 브랜치가 연결되지 않을 때

로컬 브랜치에 upstream이 없으면 IDE의 ahead/behind 표시와 `git pull` 기본 동작이 기대와
다를 수 있다. `git branch -vv`로 연결 상태를 먼저 확인한다.

```shell
git branch -vv
git status -sb
git remote -v
```

## upstream 설정

```shell
# 현재 브랜치를 origin/feature/login에 연결
git branch --set-upstream-to=origin/feature/login

# 새 원격 브랜치를 push하면서 연결
git push -u origin feature/login
```

- `-u`는 `--set-upstream`의 축약형이며, push 대상과 pull 대상 설정을 함께 기록한다.
- 로컬 브랜치와 원격 브랜치의 이름이 달라도 추적할 수 있지만, 혼동을 줄이려면 이름을 맞춘다.
- 설정 후 `git fetch origin`과 `git status -sb`로 ahead/behind 상태를 확인한다.

IDE에서 표시가 갱신되지 않으면 IDE의 Git fetch 주기, 저장소 루트 인식, 현재 checkout 브랜치도
확인한다.
