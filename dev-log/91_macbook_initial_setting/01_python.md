# 파이썬 설치 및 버전 설정

## 1단계: pyenv 설치

```shell
brew install pyenv
```

## 2단계: 쉘(Shell) 환경 변수 등록 (가장 중요)

```shell
echo 'export PYENV_ROOT="$HOME/.pyenv"' >> ~/.zshrc
echo 'export PATH="$PYENV_ROOT/bin:$PATH"' >> ~/.zshrc
echo 'eval "$(pyenv init --path)"' >> ~/.zshrc
echo 'eval "$(pyenv init -)"' >> ~/.zshrc

source ~/.zshrc
```

## 3단계: Python 3.11 설치 및 글로벌 설정

```shell
pyenv install 3.11.11
pyenv global 3.11.11
pyenv rehash
```

---

## VS Code 에서 Python 설정하기(.replace Pycharm)

아래 순서대로 하면 PyCharm 없이 VS Code에서 이 프로젝트를 실행/개발할 수 있습니다.

### 1. Extension 설치

필수:

```
- Python - Microsoft
- Pylance - Microsoft
- Python Debugger - Microsoft
- Black Formatter - Microsoft
- isort - Microsoft
- Docker - Microsoft
```

권장:

```
- YAML - Docker/AWS 설정 파일 보기 좋게
- AWS Toolkit - AWS 리소스 확인이 필요할 때
- DotENV - .env, .env.dev, .env.prod 문법 강조
- GitLens - Git 변경 이력 확인용
```

### 2. VS Code 에서 인터프리터 선택

VS Code에서 프로젝트 폴더를 연 뒤:

```
1. Cmd + Shift + P
2. Python: Select Interpreter
3. .venv/bin/python 선택
```

### 3. VS Code 설정 추천

.vscode/settings.json을 만들면 아래처럼 설정하는 것을 추천합니다.

```json
{
  "python.defaultInterpreterPath": "${workspaceFolder}/.venv/bin/python",
  "python.analysis.extraPaths": ["${workspaceFolder}"],
  "python.testing.pytestEnabled": true,
  "python.testing.pytestArgs": ["tests"],
  "editor.formatOnSave": true,
  "python.formatting.provider": "none",
  "[python]": {
    "editor.defaultFormatter": "ms-python.black-formatter",
    "editor.codeActionsOnSave": {
      "source.organizeImports": "explicit"
    }
  },
  "isort.args": ["--profile", "black"]
}
```

### 4. VS Code Debug 설정

.vscode/launch.json:

```json
{
  "version": "0.2.0",
  "configurations": [
    {
      "name": "Run Worker - local",
      "type": "debugpy",
      "request": "launch",
      "module": "src.main",
      "cwd": "${workspaceFolder}",
      "envFile": "${workspaceFolder}/.env",
      "env": {
        "APP_ENV": "local"
      },
      "console": "integratedTerminal"
    }
  ]
}
```

VS Code의 Run and Debug에서 Run Worker - local 선택 후 실행하면 됩니다.

### 5. 테스트 실행

```shell
source .venv/bin/activate
python -m pytest
```

이 프로젝트는 pyproject.toml에 pytest 설정이 있습니다.

```
[tool.pytest.ini_options]
pythonpath = "src"
testpaths = ["tests"]
```

---

## 요약

```shell
cd /Users/wondushim/Desktop/work/<workdir>
python -m venv .venv
source .venv/bin/activate
python -m pip install --upgrade pip
python -m pip install -r requirements.txt
python -m pip install -e ".[dev]"
mkdir -p data
docker compose up ministack ministack-init
python scripts/init_db.py
python -m src.main
```

- VS Code에서는 .venv/bin/python을 interpreter로 선택하고, launch.json으로 디버깅
