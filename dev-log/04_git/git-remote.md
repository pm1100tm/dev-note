# Git remote

remote는 로컬 저장소가 참조하는 원격 저장소의 별칭이다. 일반적으로 clone하면 `origin`이
등록되며, fetch URL과 push URL을 확인한 뒤 변경한다.

```shell
git remote
git remote -v
git remote show origin
git remote add upstream git@github.com:organization/repository.git
git remote set-url origin git@github.com:account/repository.git
git remote remove upstream
```

- `remote -v`는 fetch와 push URL을 모두 보여 준다.
- `remote show origin`은 원격 브랜치와 추적 관계를 보여 주며 네트워크에 연결할 수 있다.
- fork 기반 작업에서는 원본 저장소를 `upstream`, 개인 fork를 `origin`으로 두는 관례가 많다.
- URL을 변경하기 전 현재 remote를 확인해 다른 저장소로 push하는 사고를 막는다.
