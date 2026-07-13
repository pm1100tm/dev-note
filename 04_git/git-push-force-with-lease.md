# git push --force-with-lease

git push --force-with-lease는 원격 브랜치를 강제로 덮어쓰되, 다른 사람이 그 사이에 올린 커밋이
있으면 실패하게 하는 안전한 force push입니다.

## 왜 쓰나

보통 아래 작업 후 원격 브랜치 히스토리를 다시 써야 할 때 사용합니다.

```shell
git rebase main
git commit --amend
git rebase -i
```

이런 작업은 커밋 해시를 바꾸기 때문에 일반 push가 실패합니다.

그래서 강제 push가 필요합니다.

## --force와 차이

```shell
git push --force
```

이건 원격 브랜치를 그냥 덮어씁니다.
그 사이에 다른 사람이 push한 커밋도 날릴 수 있습니다.

반면:

```shell
git push --force-with-lease
```

이건 내가 마지막으로 알고 있던 원격 상태와 현재 원격 상태가 같을 때만 덮어씁니다.

즉:

내가 마지막으로 본 origin/feature 상태 == 실제 원격 feature 상태이면 push 성공.

그 사이 누가 push해서 원격이 바뀌었으면 push 실패.

## 언제 쓰나

주로 PR 브랜치에서 사용합니다.

```shell
git fetch origin
git rebase origin/main
git push --force-with-lease
```

또는 마지막 커밋 메시지를 고친 뒤:

```shell
git commit --amend
git push --force-with-lease
```

## 한 줄 요약

--force-with-lease는 내 브랜치 히스토리를 다시 쓴 뒤 원격에 반영할 때 쓰는,
--force보다 안전한 강제 push 옵션입니다.

기본적으로 --force 대신 --force-with-lease를 쓰는 습관이 좋습니다.
