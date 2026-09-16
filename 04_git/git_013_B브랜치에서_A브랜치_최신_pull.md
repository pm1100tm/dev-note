# A 브랜치에 checkout 한 상황에서, 다른 브랜치의 최신 업데이트 pull 받기

브랜치 B에 체크아웃된 상태에서 최신 main 브랜치에 PR이 머지되어, 바로 리베이스를 하고 싶은 경우,
main 브랜치로 이동 없이 pull 받을 수 있습니다.

```shell
# 작업 브랜치에 체크아웃된 상태에서, main 브랜치의 최신 커밋을 main 브랜치로 가져온다.
git fetch origin main:main
```

이렇게 하면, main 브랜치로 체크아웃해서 pull 받고, 다시 작업 브랜치로 체크아웃 하는 번거로움을
줄일 수 잇습니다.

## 실습

```shell
# 현재 메인 최신 커밋
7095e6b (HEAD -> main) ...


# 현재 작업 브랜치의 최신 커밋. 메인과 SHA 값이 같은 것을 확인할 수 있다.
7095e6b (HEAD -> feat/git-20260916) ...


# 이 상태에서 아래의 명령어 실행
git fetch origin main:main

# 그 후, 작업 브랜치에서 main 브랜치 rebase
# ✏️ 유의할 점은 변경된 작업이 있다면 stash 또는 commit 을 해두어야 한다.
git rebase -i main


# 그 후 git log --oneline
c282d09 (HEAD -> feat/git-20260916) ...      # 현재 작업 중인 브랜치의 최신 commit SHA 값
c31b3a1 (origin/main, origin/HEAD, main) ... # 업데이트 된 main 의 최신 commit SHA 값
7095e6b (origin/feat/test-20260916) ...      # 기존 main commit 의 SHA 값
```
