# 로컬 구동 - lg-havc-worker

================================================================================
===맥북 M1 기준 python 3.11 버전이 없는 경우, 먼저 버전 설치가 필요함
================================================================================
1단계: pyenv 설치
brew install pyenv

2단계: 쉘(Shell) 환경 변수 등록 (가장 중요)
echo 'export PYENV_ROOT="$HOME/.pyenv"' >> ~/.zshrc
echo 'export PATH="$PYENV_ROOT/bin:$PATH"' >> ~/.zshrc
echo 'eval "$(pyenv init --path)"' >> ~/.zshrc
echo 'eval "$(pyenv init -)"' >> ~/.zshrc

# 변경된 설정 파일 적용

source ~/.zshrc

3단계: Python 3.11 설치 및 글로벌 설정

pyenv install 3.11.11
pyenv global 3.11.11
pyenv rehash

4단계: 설치 및 설정 결과 확인

================================================================================
===아래 순서대로 하면 PyCharm 없이 VS Code에서 이 프로젝트를 실행/개발할 수 있습니다.
================================================================================

1. VS Code Extension 설치

필수:

- Python - Microsoft
- Pylance - Microsoft
- Python Debugger - Microsoft
- Black Formatter - Microsoft
- isort - Microsoft
- Docker - Microsoft

권장:

- YAML - Docker/AWS 설정 파일 보기 좋게
- AWS Toolkit - AWS 리소스 확인이 필요할 때
- DotENV - .env, .env.dev, .env.prod 문법 강조
- GitLens - Git 변경 이력 확인용

2. 이 프로젝트의 패키지 관리 방식

이 프로젝트는 pip + requirements.txt 기반으로 보는 게 맞습니다.

현재 구조:

- requirements.txt: 실제 로컬/Docker 실행에 필요한 주요 패키지 목록
- pyproject.toml: 프로젝트 메타데이터, dev 의존성, pytest 설정, worker 실행 스크립트 정의
- lock 파일 없음: poetry.lock, uv.lock, Pipfile.lock 같은 고정 버전 파일은 없음

중요한 점은 pyproject.toml의 dependencies가 requirements.txt보다 적습니다. 그래서 로컬 개발에서는 requirements.txt를 먼저 설치하는 방식이 안전합니다.

3. Python 버전

pyproject.toml 기준으로 Python 3.11 이상이 필요합니다.

확인:

python3 --version

가능하면 Python 3.11 사용을 추천합니다.

4. 가상환경 생성

프로젝트 루트에서 실행하세요.

cd /Users/wondushim/Desktop/work-lg-cad/lg-havc-worker
python3.11 -m venv .venv

만약 python3.11 명령이 없다면:

python -m venv .venv

활성화:

source .venv/bin/activate

정상 활성화되면 프롬프트 앞에 (.venv)가 붙습니다.

5. 패키지 설치

python -m pip install --upgrade pip
python -m pip install -r requirements.txt
python -m pip install -e ".[dev]"

설명:

- requirements.txt: 실제 실행에 필요한 패키지 설치
- -e ".[dev]": 현재 프로젝트를 editable install 하고, pytest, black, isort, mypy, boto3-stubs 같은 개발 도구 설치
- 설치 후 worker 콘솔 명령도 사용할 수 있게 됩니다

6. VS Code에서 인터프리터 선택

VS Code에서 프로젝트 폴더를 연 뒤:

1. Cmd + Shift + P
2. Python: Select Interpreter
3. .venv/bin/python 선택

선택 후 VS Code 하단 또는 Python 출력에서 .venv가 잡혔는지 확인하세요.

7. VS Code 설정 추천

.vscode/settings.json을 만들면 아래처럼 설정하는 것을 추천합니다.

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

8. 로컬 실행 전 필요한 인프라

이 프로젝트는 단순 Python 스크립트가 아니라 SQS Worker입니다. 실행하려면 최소한 아래가 필요합니다.

- MiniStack: SQS/S3 로컬 대체
- PostgreSQL: Job 상태 저장용 DB
- .env: 로컬 환경 변수

주의할 점: 현재 docker-compose.yml에는 MiniStack과 worker는 있지만 PostgreSQL 서비스는 없습니다.
즉, DB는 별도로 떠 있어야 합니다.

기본 DB 설정은 src/config.py 기준:

DB_HOST=localhost
DB_PORT=5432
DB_NAME=lg_hvac
DB_USER=postgres
DB_PASSWORD=password

9. MiniStack 실행

로컬에서 Python worker를 VS Code로 직접 실행하려면, 먼저 MiniStack만 필요합니다.

가장 단순한 방법은 Docker Compose를 켜는 것입니다.

mkdir -p data
docker compose up ministack ministack-init

또는 전체 컨테이너 워커까지 같이 띄우려면:

docker compose up --build

다만 VS Code에서 직접 디버깅하려면 worker 컨테이너까지 띄우기보다는 MiniStack/DB만 띄우고 Python worker는 VS Code에서 실행하는 편이 낫습니다.

10. DB 초기화

PostgreSQL이 떠 있고 .env의 DB 설정이 맞는 상태에서:

source .venv/bin/activate
python scripts/init_db.py

단, 코드의 SQLAlchemy 모델은 lg_hvac schema를 사용합니다. DB에 schema가 없다면 PostgreSQL에서 먼저 만들어야 합니다.

CREATE SCHEMA IF NOT EXISTS lg_hvac;

11. Worker 실행

터미널에서 직접 실행:

source .venv/bin/activate
python -m src.main

또는 editable install 후:

worker

개발 AWS 환경으로 실행하려면:

APP_ENV=dev python -m src.main

기본 로컬 실행은 APP_ENV=local이고 .env 파일을 읽습니다.

12. VS Code Debug 설정

.vscode/launch.json:

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

이후 VS Code의 Run and Debug에서 Run Worker - local 선택 후 실행하면 됩니다.

13. 테스트 실행

source .venv/bin/activate
python -m pytest

이 프로젝트는 pyproject.toml에 pytest 설정이 있습니다.

[tool.pytest.ini_options]
pythonpath = "src"
testpaths = ["tests"]

그래서 별도 PYTHONPATH 지정 없이 python -m pytest로 충분합니다.

14. 테스트 메시지 전송

가이드 문서에는 scripts/send_test_message.py라고 되어 있지만, 실제 파일은 현재 저장소 기준으로 scripts/send_db_test_message.py입니다.

예:

python scripts/send_db_test_message.py REVIEW

다른 JobType 예:

python scripts/send_db_test_message.py GENERATION
python scripts/send_db_test_message.py EQUIPMENT_LIST_EXTRACTION
python scripts/send_db_test_message.py PIPE_LENGTH_CALCULATION

최종 실행 순서 요약

cd /Users/wondushim/Desktop/work-lg-cad/lg-havc-worker

python3.11 -m venv .venv
source .venv/bin/activate

python -m pip install --upgrade pip
python -m pip install -r requirements.txt
python -m pip install -e ".[dev]"

mkdir -p data
docker compose up ministack ministack-init

python scripts/init_db.py
python -m src.main

VS Code에서는 .venv/bin/python을 interpreter로 선택하고, 위 launch.json으로 디버깅하면 됩니다.
