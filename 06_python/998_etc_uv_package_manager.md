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

## 가상환경 설정

```shell
# venv 폴더 생성
uv venv

# 버전 지정하여 생성
uv venv --python 3.12

# ex
╰─ uv venv
Using CPython 3.12.12
Creating virtual environment at: .venv
Activate with: source .venv/bin/activate

# 프로젝트 초기화
uv init

# 프로젝트 초기화 + 파이썬 버전 추가
uv init --no-workspace --python 3.12

# 가상환경 없이 프로젝트 생성
uv init my-project --no-venv

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
