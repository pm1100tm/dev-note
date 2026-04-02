# 🚀 UV 패키지 매니저

## 설치하기

```shell
# 리눅스
curl -LsSf https://astral.sh/uv/install.sh | sh

# 맥 homebrew
brew install uv

# With pip.
pip install uv
```

여기서는 brew 를 사용하여 설치함

```shell
╰─ uv --version
uv 0.9.18 (Homebrew 2025-12-16)
```

## 파이썬 버전 설치 & 설치된 목록 조회

```shell
# 파이썬 버전 설치
╰─ uv python install 3.12
cpython-3.12.12-macos-aarch64-none (download) ------------------------------ 14.45 MiB/16.21 MiB


# 설치된 파이썬 버전 조회
╰─ uv python list
cpython-3.13.11-macos-aarch64-none                  <download available>
cpython-3.13.11+freethreaded-macos-aarch64-none     <download available>
cpython-3.12.12-macos-aarch64-none  /Users/.local/share/uv/python/cpython-3.12.12-macos-aarch64-none/bin/python3.12
cpython-3.11.14-macos-aarch64-none                  <download available>
...

# 설치된 파이썬 버전 삭제
╰─ uv python uninstall 3.12
```

## 프로젝트 초기화

```shell
# 프로젝트 초기화
> uv init

# 아래의 파일 생성됨
drwxr-xr-x@  9 swd  staff  288 12 26 22:45 .git
-rw-r--r--@  1 swd  staff  109 12 26 22:45 .gitignore
-rw-r--r--@  1 swd  staff    5 12 26 22:45 .python-version
-rw-r--r--@  1 swd  staff   84 12 26 22:45 main.py
-rw-r--r--@  1 swd  staff  152 12 26 22:45 pyproject.toml
-rw-r--r--@  1 swd  staff    0 12 26 22:45 README.md
```

```shell
# 프로젝트 초기화 + 파이썬 버전 추가
> uv init --no-workspace --python 3.12
Initialized project `uvtest3`

drwxr-xr-x@  9 swd  staff  288 12 26 22:42 .git
-rw-r--r--@  1 swd  staff  109 12 26 22:42 .gitignore
-rw-r--r--@  1 swd  staff    5 12 26 22:42 .python-version
-rw-r--r--@  1 swd  staff   85 12 26 22:42 main.py
-rw-r--r--@  1 swd  staff  153 12 26 22:42 pyproject.toml
-rw-r--r--@  1 swd  staff    0 12 26 22:42 README.md
```

## 가상환경 설정

```shell
# 가상 환경 생성
> uv venv
Using CPython 3.12.12
Creating virtual environment at: .venv
Activate with: source .venv/bin/activate

👉
# 버전 지정하여 가상 환경 생성
> uv venv --python 3.12
Using CPython 3.12.12
Creating virtual environment at: .venv
Activate with: source .venv/bin/activate

# 파일 생성
.gitignore
.python-version # 파이썬 버전 고정
main.py
pyproject.toml # 의존성 & 프로젝트 메타데이터 정의
README.md

# 의존성 설치
uv add fastapi

# 개발환경 의존성 설치
# uv add --group stg pytest << 이렇게 그룹 지정도 가능
uv add --dev pytest

# pyproject.toml 에 보면 dependency-groups 의 dev 목록에 pytest 확인
[project]
name = "uvtest"
version = "0.1.0"
description = "Add your description here"
readme = "README.md"
requires-python = ">=3.12"
dependencies = [
  "fastapi>=0.127.0",
  "requests>=2.32.5",
]

[dependency-groups]
dev = [
    "pytest>=9.0.2",
]


# requirements.txt 도 한번에 설치 가능
uv add -r requirements.txt
```

- uv add 명령어는 uv sync 기능 포함
  - 따라서, 패키지를 추가하면 자동으로 동기화까지 처리되며, 자동으로 uv sync 실행하여 패키지 실행 및
    잠금 파일 uv.lock 을 업데이트 함

```shell
# dependencies만 싱크
uv sync --no-dev

# dependency-groups의 dev만 싱크
uv sync --only-dev

# 특정 그룹만 싱크 - dev가 존재하다면 dev는 필수적으로 설치됨
--group <GROUP>

# 특정 그룹만 싱크 제외
uv sync --no-group <NO_GROUP>
```

## 실행

```shell
uv run <파일명>
```

- uv run 을 입력하면 가상환경을 자동 적용하여 파이썬 스크립트를 실행함
- 만약 패키지 설치가 안되었다고 하면 패키지를 설치한 후 실행
- 즉, uv run 은 아래의 명령어 포함
  - uv venv
  - uv sync

## pyproject.toml 기반으로 requirements.txt 생성

```shell
uv pip compile pyproject.toml -o requirements.txt --no-deps

# 만약 특정 그룹의 패키지도 설치하려면 —group <그룹명> 옵션을 추가
uv pip compile pyproject.toml -o dev_requirements.txt --no-deps --group dev
```

## uv 로 설치한 list 확인

```shell
uv pip list
```

## 임시 의존성 설치 후 실행

- uvx 명령어를 사용하면 파이썬 패키지를 설치하지 않고 임시 환경에서 즉시 실행 가능
- 과거의 방식
  - 전역 또는 가상 환경에 도구를 설치합니다. (pip install black)
  - 명령어를 실행합니다. (black .)
  - 환경을 오염시키지 않기 위해 다시 삭제합니다. (pip uninstall black)

- uvx 방식
  - 격리된 임시 환경을 만듭니다. (uvx black .)
  - black 패키지와 그 의존성을 다운로드하여 설치합니다.
  - black . 명령어를 실행합니다.

- 명령 실행이 끝나면 만들었던 임시 환경과 패키지를 모두 폐기
- uvx는 단일 패키지만 설치하고 사용하는 것이 목적에 맞습니다.
- 여러 패키지를 설치할 순 있지만 의도와 멀어지게 됩니다.

---

## CI/CD 에서 uv 사용

의존성 버전 고정 (Frozen) 전략 - ✅ uv에서 진짜 고정은 uv.lock

```shell
pyproject.toml  ← 사람이 관리
uv.lock         ← CI/운영이 신뢰하는 파일
```

로컬 개발 흐름 (정석)

```shell
# 패키지 추가
uv add fastapi

# → uv.lock 자동 갱신됨
```

```shell
# 📌 pyproject.toml
dependencies = [
  "fastapi>=0.127.0"
]

# 📌 uv.lock
fastapi==0.127.3
starlette==0.41.2
...
```

### CI/운영에서는 반드시 frozen

```shell
uv sync --frozen
```

❌ 이 경우 실패함

- pyproject.toml 수정했는데 uv.lock 갱신 안 한 경우
- 버전 충돌
- 의존성 누락

- 👉 CI에서 실패하는 게 정상
- 👉 “로컬에서 lock 안 맞춘 것이 잘못”

### Dev / Prod 분리 설치

```shell
# 운영
uv sync --frozen --no-dev

# 개발
uv sync --frozen
```

### pyproject.toml 제대로 만드는 방법

최소 구성 (실무 기준)

```shell
[project]
name = "my-service"
version = "0.1.0"
description = "FastAPI service"
readme = "README.md"
requires-python = ">=3.12"

dependencies = [
  "fastapi>=0.127.0",
  "uvicorn>=0.30.0",
  "requests>=2.32.0",
]

[dependency-groups]
dev = [
  "pytest>=9.0.0",
  "ruff>=0.6.0",
  "black>=24.0.0",
]
```

버전 정책 추천 (중요)

| 타입         | 예시             | 이유                |
| ------------ | ---------------- | ------------------- |
| 애플리케이션 | fastapi>=0.127.0 | minor 자동 업데이트 |
| 라이브러리   | ~=1.4            | 하위 호환 유지      |
| 내부 툴      | ==               | 재현성              |

- ❌ pyproject에 == 남발하지 말 것
- 👉 고정은 lock 파일이 책임진다

#### Python 버전 고정 (.python-version)

- .python-version 파일

```shell
3.12
```

### requirements.txt가 필요한 경우 (외부 시스템)

```shell
uv pip compile pyproject.toml \
  -o requirements.txt \
  --no-deps
```

```shell
uv pip compile pyproject.toml \
  -o requirements-dev.txt \
  --no-deps \
  --group dev
```

| 환경        | 명령                        |
| ----------- | --------------------------- |
| 로컬 실행   | `uv run app.py`             |
| 패키지 추가 | `uv add <pkg>`              |
| CI 설치     | `uv sync --frozen`          |
| 운영        | `uv sync --frozen --no-dev` |
| Docker      | `uv sync --frozen`          |

### 핵심

```shell
pyproject.toml = 선언
uv.lock        = 진실
CI             = --frozen
```

---

## 명령어 정리

```shell
# 설치
# Linux
curl -LsSf https://astral.sh/uv/install.sh | sh

# macOS(Homebrew)
brew install uv

# pip
pip install uv

# Python 버전 설치/삭제
uv python list
uv python install 3.12
uv python uninstall 3.12

# 가상환경
uv venv
uv venv --python 3.12

# 프로젝트 초기화
uv init

# 프로젝트 초기화 + Python 버전 지정
uv init --no-workspace --python 3.12

# 가상환경 없이 프로젝트 생성
uv init my-project --no-venv

# 일반 의존성 추가
# .venv 에 실제 설치, pyproject.toml 수정
uv add fastapi

# 임시 / 응급용 의존성 추가
# .venv 에 실제 설치, pyproject.toml 수정하지 않음
uv pip install fastapi

# 개발용 의존성 추가
uv add --dev pytest

# requirements.txt 기반 설치
uv add -r requirements.txt

# dependencies 만 설치
uv sync --no-dev

# dev 그룹만 설치
uv sync --only-dev

# 특정 그룹만 설치
uv sync --group <GROUP>

# 특정 그룹 제외 설치
uv sync --no-group <GROUP>

# 설치된 패키지 목록
uv pip list

# 가상환경 자동 적용 후 실행
uv run main.py

# 임시 실행
uvx black .
```
