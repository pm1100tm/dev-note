# git rm

`git rm`은 파일을 작업 디렉터리와 Git의 추적 대상에서 함께 제거하고 그 삭제를 스테이징한다.
파일은 유지하되 Git 추적만 중단하려면 `--cached`를 사용한다.

```shell
# 파일 삭제를 스테이징
git rm path/to/file

# 로컬 파일은 유지하고 Git 추적만 해제
git rm --cached path/to/file

# 디렉터리 재귀 삭제
git rm -r path/to/directory
```

`git rm --cached .env` 뒤에는 `.gitignore`에 `.env`를 추가해야 다음 `git add`에서 다시
추적되지 않는다. 이미 원격에 push한 비밀값은 파일을 지운 뒤에도 이력에 남을 수 있으므로
즉시 폐기·교체하고 별도의 이력 정리 절차를 검토한다.
