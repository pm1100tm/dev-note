# Git

Git은 파일 변경 이력을 커밋으로 기록하고, 브랜치와 원격 저장소를 통해 협업하는 분산 버전
관리 시스템이다. 이 문서는 일상적인 변경 확인부터 원격 브랜치·태그·SSH 계정 관리까지 다룬다.

## 학습 순서

- [브랜치 이름 변경](git_001_branch-rename.md)
- [커밋 메시지 템플릿](git_002_commit-message.md)
- [변경 사항 비교](git_003_git-diff.md)
- [fetch와 pull](git_004_git-fetch-pull.md)
- [로그 조회](git_005_git-log.md)
- [여러 GitHub 계정의 SSH 설정](git_006_git-multiple-account-ssh.md)
- [안전한 강제 push](git_007_git-push-force-with-lease.md)
- [원격 추적 브랜치가 연결되지 않을 때](git_008_git-remote-추척이_안될때.md)
- [원격 저장소 관리](git_009_git-remote.md)
- [파일 삭제와 추적 해제](git_010_git-rm.md)
- [upstream 설정](git_011_git-set-upstream.md)
- [태그 관리](git_012_git-tag.md)
- [다른 브랜치의 최신 업데이트 받기](git_013_B브랜치에서_A브랜치_최신_pull.md)
- [브랜치 보호 설정하기](git_014_브랜치_보호_설정하기.md)

## 작업 전 원칙

- `status`, `diff`, `log`로 현재 상태와 변경 대상을 확인한 후 수정·전송한다.
- 공유 브랜치의 이력을 바꾸는 rebase, amend, force push는 팀 규칙과 PR 상태를 먼저 확인한다.
- 토큰·비밀번호·개인 키는 커밋하지 않는다. 이미 커밋했다면 삭제만으로 충분하지 않을 수 있다.
