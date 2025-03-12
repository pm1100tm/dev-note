# Git 로컬 브랜치의 원격 트래킹을 변경하는 방법

## --set-upstream-to 옵션

해당 옵션은 로컬 브랜치가 추적하는(upstream) 원격 브랜치를 지정할 때 사용합니다.

```shell
git branch --set-upstream-to=origin/feature/2 feature/1
```

위 명령은 로컬 feature/1 브랜치가 원격의 feature/2 브랜치를 추적하도록 변경합니다.

명령어의 마지막 인자로 대상 로컬 브랜치(feature/1)를 명시합니다.
만약 현재 feature/1 브랜치에 있다면 아래와 같이 단축해서 사용할 수도 있습니다.

```shell
git branch --set-upstream-to=origin/feature/2
```

## 추가 팁

### 로컬 브랜치 이름 변경

```shell
git branch -m feature/1 feature/2
```

### 새 이름의 브랜치를 원격에 업로드 및 추적 설정

```shell
git push -u origin feature/2
```

### 원격의 기존 feature/1 브랜치 삭제

```shell
git push origin --delete feature/1
```
