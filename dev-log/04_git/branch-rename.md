# 브랜치 이름 변경

로컬 브랜치 이름을 바꾼 뒤에는 원격 브랜치를 새 이름으로 push하고, 기존 원격 브랜치의
삭제 여부와 PR 연결 상태를 확인해야 한다.

## 현재 브랜치 이름 변경

```shell
git branch --show-current
git branch -m new-branch-name
```

## 다른 로컬 브랜치 이름 변경

```shell
git branch -m old-branch-name new-branch-name
```

## 원격 브랜치도 변경

```shell
git push origin -u new-branch-name
git push origin --delete old-branch-name
```

- `-m`은 이름 변경이며, 대상 이름이 이미 있으면 실패한다.
- `-M`은 대상 이름이 있어도 덮어쓰므로 사용 전 해당 브랜치의 내용을 확인한다.
- 기본 브랜치와 열린 PR이 연결된 원격 브랜치는 삭제 전에 팀과 CI 설정을 확인한다.
