# Git 로그 조회

`git log`는 커밋 이력을 확인하는 명령이다. 그래프, 작성자, 기간, 파일 경로를 함께 지정하면
원하는 변경을 빠르게 찾을 수 있다.

```shell
git log
git log -n 10 --oneline
git log --graph --decorate --oneline --all
git log -p -n 1
git log --stat
git log --since="2026-01-01" --until="2026-01-31"
git log --author="name@example.com"
git log -- path/to/file
```

- `--oneline`은 해시와 제목만 표시해 이력 흐름을 훑기 좋다.
- `--graph --all`은 브랜치와 merge 관계를 함께 표시한다.
- `-p`는 각 커밋의 patch를, `--stat`은 파일별 변경량을 보여 준다.
- `-- path/to/file` 앞의 `--`는 옵션과 파일 경로를 구분한다.
