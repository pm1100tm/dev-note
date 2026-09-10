# git push --force-with-lease

`git push --force-with-lease`는 원격 브랜치를 덮어쓰기 전에, 원격이 내가 마지막으로 확인한
상태와 같은지 검사한다. 일반 `--force`보다 안전하지만 공유 브랜치에는 여전히 위험하다.

## 사용하는 상황

```shell
git fetch origin
git rebase origin/main
git push --force-with-lease origin HEAD
```

```shell
git commit --amend
git push --force-with-lease origin feature/login
```

- rebase, interactive rebase, amend는 커밋 해시를 바꾸므로 일반 push가 거부될 수 있다.
- lease 검사가 실패하면 누군가 원격에 새 커밋을 push했을 수 있다. fetch 후 이력을 검토한다.
- `--force`는 다른 사람의 원격 커밋까지 덮어쓸 수 있으므로 기본 선택으로 쓰지 않는다.
- main, release처럼 공유하는 보호 브랜치에는 force push를 허용하지 않는 것이 원칙이다.

원격 브랜치가 명확하지 않을 때는 `git push --force-with-lease origin HEAD:feature/login`처럼
대상을 명시한다.
