# CLAUDE.md

개인 개발 학습 노트를 모은 GitBook 저장소입니다. 코드·빌드·테스트는 없고, 결과물은
Markdown 문서와 목차입니다. `main`에 반영되면 GitBook Git Sync가
`gitbook-docs.yaml`(`content.directory: ./`) 기준으로 사이트를 갱신합니다.

## 문서를 추가·이동·삭제할 때

목차가 두 군데에 있어 항상 함께 맞춥니다.

- `SUMMARY.md`: GitBook 사이드바를 결정하는 유일한 목차입니다.
- 주제 폴더의 `README.md`: 그 주제의 문서 목록을 따로 관리합니다.

파일명은 폴더마다 관례가 다릅니다(`git_016_`, `KUBE_002_`, `etc_027_`, `001_`,
`a.` 등). 같은 폴더의 파일을 `ls`로 확인하고 그 패턴의 다음 번호를 씁니다.
`99_trouble_shooting/`은 `YYYYMMDD_` 날짜 접두사를 씁니다.

이미지는 문서가 있는 주제 폴더의 기존 이미지 폴더(`assets/`, `07_react`는 `asset/`)에
두고 상대 경로로 링크합니다. `.gitbook/assets/`는 예전 이미지 보관용이라 새로 추가하지
않습니다.

`SUMMARY.md` 끝의 `<!-- ... -->` 블록은 의도적으로 꺼 둔 목차입니다. 요청이 없으면 주석을
풀거나 정리하지 않습니다.

## 글 쓰는 방식

- 한국어로 씁니다. 새 문서는 최근 문서처럼 합니다체를 쓰고, 기존 문서를 고칠 때는 그
  문서의 문체와 줄바꿈 방식을 그대로 따릅니다.
- 좋은 예시는 `04_git/git_016_git-worktree.md`,
  `11_ai_agent/claude/claude_002_best_practices.md`입니다. 왜 필요한지 배경으로
  시작하고, 개념 → 실습·예시 → 주의점 순으로 정리합니다.
- 명령어 출력, 버전, 옵션처럼 바뀔 수 있는 사실은 직접 실행하거나 공식 문서로 확인한 뒤
  적고, 문서 상단에 확인일·확인 환경·출처 링크를 남깁니다.
- 코드 블록에는 언어를 지정합니다(`shell`, `java`, `yaml` 등).
- 요청받은 문서만 고칩니다. 다른 문서의 오탈자나 깨진 링크를 발견하면 고치지 말고
  알려만 줍니다.

## 공개 범위

이 저장소는 GitBook으로 공개됩니다.

- 토큰, 비밀번호, 계정 ID, 내부 호스트명·IP, 회사·고객 정보는 문서와 예시에 넣지
  않습니다. 필요하면 `<YOUR_TOKEN>`, `example.com`처럼 치환합니다.
- 비공개 메모는 `private_*.md`로 만듭니다. `.gitignore`가 제외하므로 커밋되지 않고,
  `SUMMARY.md`나 README에서 링크하지 않습니다. `aprivate_`처럼 패턴과 다른 이름은
  제외되지 않는다는 점에 주의합니다.
- 또한, 파일명에 `ING_`, `_ING_`, `(ING)`가 포함된 문서는 `SUMMARY.md`나 README에서
  링크하지 않습니다.
- `.gitignore` 대상(`temp_react`, `ig_*`, `.env*` 등)도 공개 목차에 링크하지 않습니다.

## 변경 후 확인

빌드가 없으므로 링크 검증이 유일한 자동 점검입니다. 변경한 문서와, 목차를 건드렸다면
`SUMMARY.md`와 해당 `README.md`를 넣어 실행합니다.

````shell
python3 - SUMMARY.md <변경한_문서.md> <<'EOF'
import os, re, sys
from urllib.parse import unquote
link = re.compile(r'!?\[[^\]]*\]\((<[^>]+>|[^)\s]+)')
for p in sys.argv[1:]:
    text = open(p, encoding='utf-8').read()
    text = re.sub(r'<!--.*?-->|```.*?```', '', text, flags=re.S)
    for m in link.finditer(text):
        target = m.group(1).strip('<>').split('#')[0]
        if not target or re.match(r'[a-z]+:', target):
            continue
        path = os.path.join(os.path.dirname(p), unquote(target))
        if not os.path.exists(path):
            print(f'{p}: 없는 경로 {target}')
EOF
````

`SUMMARY.md`에는 이미 깨진 링크가 여럿 있습니다(2026-10-16 기준 약 40개). 이번 변경으로
새로 생긴 항목만 고치고, 기존 항목은 요청이 있을 때만 정리합니다. 공백이나 괄호가 들어간
경로는 `[제목](<경로 이름.md>)`처럼 꺾쇠로 감쌉니다.

## 커밋

개인 저장소라 브랜치 없이 `main`에 직접 커밋합니다. 커밋은 요청받았을 때만 합니다.

`.gitmessage`의 `<type>(<scope>): <제목>` 형식을 따르고, 제목은 한국어 50자 안팎으로
마침표 없이 씁니다. 문서 추가는 `feat: <주제> 학습 기록`, 오타·목차 정리는 `docs:`,
설정 변경은 `chore:`를 씁니다. 본문에는 무엇을 왜 바꿨는지 `-` 불릿으로 적습니다.

```text
feat: git worktree 개념과 실습 정리

- 워크트리 생성·삭제 실습과 stash, clone 비교 추가
- Git README와 SUMMARY 목차에 등록
```
