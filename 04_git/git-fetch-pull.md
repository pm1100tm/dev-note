# git fetch와 git pull

둘 다 원격 저장소의 변경을 가져오지만, `fetch`는 원격 추적 브랜치만 갱신하고 `pull`은
현재 브랜치에 통합까지 수행한다.

```shell
# 원격 정보만 갱신
git fetch origin
git log --oneline HEAD..origin/main

# 현재 브랜치에 통합
git pull --ff-only

# rebase 방식으로 통합
git pull --rebase
```

| 명령 | 결과 |
| --- | --- |
| `git fetch` | `origin/main` 같은 원격 추적 브랜치만 갱신한다. |
| `git pull` | fetch 후 merge 또는 rebase를 수행한다. |
| `git pull --ff-only` | fast-forward가 불가능하면 실패해 의도치 않은 merge commit을 막는다. |

- 변경을 검토하거나 충돌 가능성을 판단할 때는 먼저 `fetch`를 사용한다.
- 작업 트리가 깨끗하지 않으면 `pull` 전 커밋하거나 stash해 충돌 범위를 줄인다.
- 팀의 통합 방식이 정해져 있다면 `merge`와 `rebase`를 섞지 않고 그 규칙을 따른다.
