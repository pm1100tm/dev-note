# Git 태그

태그는 특정 커밋에 버전 같은 읽기 쉬운 이름을 붙인다. 릴리스에는 작성자·날짜·메시지를 담는
annotated tag를 사용하고, 원격에 push해야 다른 사람이 볼 수 있다.

## 태그 만들기와 조회

```shell
git tag -a v1.0.0 -m "Release v1.0.0"
git tag -a v1.0.0 <commit-sha> -m "Release v1.0.0"
git tag -l "v1.*"
git show v1.0.0
```

## 원격 전송과 삭제

```shell
git push origin v1.0.0
git push origin --tags
git tag -d v1.0.0
git push origin --delete v1.0.0
```

## 태그 기준 작업

```shell
git switch --detach v1.0.0
git switch -c hotfix/v1.0.1 v1.0.0
```

- 태그를 checkout하면 detached HEAD가 된다. 새 작업은 브랜치를 만들어 시작한다.
- 이미 배포한 태그를 `-f`로 재지정하면 빌드 재현성과 배포 추적이 깨진다. 새 버전을 발행한다.
- `git push origin --tags`는 모든 로컬 태그를 전송하므로 의도한 태그만 보낼 때는 이름을 명시한다.
