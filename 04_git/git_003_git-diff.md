# Git diff

`git diff`는 작업 디렉터리, 스테이징 영역, 커밋 사이의 변경을 비교한다. 커밋 전에는
의도하지 않은 파일과 비밀값이 포함되지 않았는지 반드시 확인한다.

```shell
# 작업 디렉터리와 스테이징 영역 비교
git diff

# 스테이징 영역과 HEAD 비교
git diff --staged
git diff --cached

# 두 브랜치 비교
git diff main...HEAD

# 변경된 파일 이름과 상태만 확인
git diff --name-status
```

- `git diff`는 아직 `git add`하지 않은 변경을 보여 준다.
- `git diff --staged`는 다음 커밋에 포함될 변경을 보여 준다.
- `main...HEAD`는 공통 조상 이후 현재 브랜치가 만든 변경을 비교하므로 PR 검토에 유용하다.
- 큰 diff는 `--stat`, `--word-diff`, `-- path/to/file` 옵션으로 범위를 좁혀 확인한다.
