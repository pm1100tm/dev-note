# 커밋 메시지 템플릿

좋은 커밋 메시지는 변경의 의도와 범위를 빠르게 파악하게 한다. 제목은 간결하게 쓰고,
필요하면 본문에 무엇을 왜 바꿨는지 남긴다.

## 템플릿 설정

```shell
touch ~/.gitmessage
git config --global commit.template ~/.gitmessage
git config --global --get commit.template
```

현재 저장소에만 적용하려면 `--global`을 제외한다.

## 예시

```text
feat(auth): 로그인 실패 횟수 제한 추가

- 반복 로그인 실패 시 계정 잠금 정책을 적용한다.
- 관리자 해제 API와 감사 로그를 추가한다.

Refs: #123
```

## 권장 규칙

- 제목은 보통 50자 안팎으로 쓰고 마침표를 생략한다.
- 제목과 본문 사이는 빈 줄로 구분한다.
- 본문에는 구현 방법보다 변경 이유와 영향 범위를 우선 기록한다.
- 하나의 커밋에는 하나의 논리적 변경을 담는다.

`feat`, `fix`, `docs`, `refactor`, `test`, `build`, `ci`, `chore` 같은 type은 팀 규칙이
있을 때 일관되게 사용한다.
