# 🚀 Claude Code CLI 설치와 로그인

Claude 데스크톱 앱에서 Claude를 사용할 수 있어도 터미널에서 프로젝트를 분석하거나 변경하려면
`Claude Code CLI`를 별도로 설치해야 합니다.

이 문서는 macOS와 Windows에서 Claude Code를 설치하고 계정으로 로그인한 뒤 첫 세션을 시작하는
방법을 설명합니다.

> 이 문서는 2026-09-29에 Claude Code 공식 문서를 기준으로 확인했습니다.
> 지원 운영체제와 설치 명령은 바뀔 수 있습니다.

## 공통 사전 확인

- 인터넷 연결과 터미널이 필요합니다. macOS는 Bash 또는 Zsh, Windows는 PowerShell 또는 CMD를
  사용할 수 있습니다.
- Claude Code는 macOS 13.0 이상, Windows 10 1809 이상에서 지원합니다.
- Claude Code는 Claude.ai의 Pro, Max, Team, Enterprise 요금제나 Anthropic Console
  계정으로 인증할 수 있습니다.
- 무료 Claude.ai 요금제만 사용 중이면 Claude Code에 로그인할 수 없습니다.
  먼저 사용할 요금제 또는 Console 결제를 확인합니다.
- 작업 폴더의 파일을 읽고 명령을 제안할 수 있으므로, 민감한 정보가 있는 저장소에서는 권한 요청과 변경 내용을
  검토합니다.

계정과 플랫폼 요구 사항 확인

- [Claude Code 공식 설치 문서](https://code.claude.com/docs/en/setup)에서

설치 오류가 있으면 해당 문서도 함께 확인합니다.

## macOS에 설치하기

### 권장: 공식 설치 프로그램 사용

Terminal 앱을 열고 다음 명령을 실행합니다. `curl`로 내려받은
공식 설치 스크립트를 `bash`로 실행하여 Claude Code를 설치합니다.

```shell
curl -fsSL https://claude.ai/install.sh | bash
```

설치가 끝나면 터미널 창을 완전히 닫고 새 창을 엽니다. 새 셸이 설치 경로를 인식하도록 하기 위한 과정입니다.

### Homebrew로 설치하기

Homebrew를 이미 사용하고 있고 업데이트를 직접 관리하려면 다음 명령으로 안정 채널을 설치할 수 있습니다.

```shell
brew install --cask claude-code
```

최신 채널을 사용하려면 `claude-code@latest` cask를 설치합니다.

안정 채널은 일반적으로 최신 채널보다 약 일주일 늦지만, 큰 회귀가 있는 배포판을 건너뛸 수 있습니다.
하나의 설치 방법만 선택합니다.

## Windows에 설치하기

### PowerShell에서 설치하기

PowerShell을 열고 다음 명령을 실행합니다. 관리자 권한으로 실행할
필요는 없습니다.

```powershell
irm https://claude.ai/install.ps1 | iex
```

설치가 끝나면 PowerShell 창을 닫고 새 창을 열어 설치 경로를
인식하게 합니다.

### CMD에서 설치하기

명령 프롬프트(CMD)를 사용 중이라면 PowerShell 명령 대신 다음 명령을 실행합니다.

```shell
curl -fsSL https://claude.ai/install.cmd -o install.cmd && install.cmd && del install.cmd
```

PowerShell 프롬프트는 `PS C:\\`로 시작하고, CMD 프롬프트는 `C:\\`처럼 `PS` 없이 표시됩니다.
사용하는 셸에 맞는 명령을 실행해야 합니다.

### WSL에서 설치하기

WSL 프로젝트를 사용한다면 PowerShell이나 CMD가 아니라 WSL 터미널에서 다음 명령을 실행합니다.

```shell
curl -fsSL https://claude.ai/install.sh | bash
```

Claude Code는 프로젝트가 있는 환경에서 설치하고 실행합니다.

예를 들어 WSL 파일 시스템에 있는 프로젝트는 WSL 터미널에서 `claude`를 실행합니다.

## 공통: 설치 확인

새 터미널을 열고 다음 명령을 실행합니다.

```shell
claude --version
```

버전 번호와 `Claude Code`가 출력되면 설치가 완료된 것입니다.

```shell
# 출력 예시이며 실제 버전은 설치 시점에 따라 다릅니다.
2.1.211 (Claude Code)
```

## 공통: 처음 실행하고 로그인하기

Claude Code를 사용할 프로젝트 폴더로 이동합니다.
아직 프로젝트가 없다면 빈 폴더에서 먼저 사용해도 됩니다.

```shell
cd ~/Desktop/my-project
claude
```

- 처음 실행하면 브라우저가 열리고 로그인 또는 계정 선택 화면이 표시됩니다.
- 데스크톱 앱에서 사용 중인 Claude 계정으로 로그인하고 브라우저의 인증 안내를 완료합니다.
- 완료 후 터미널로 돌아오면 대화형 세션이 시작됩니다.

> 데스크톱 앱 로그인 상태가 브라우저에 남아 있다면 계정을 선택하는 과정이 짧아질 수 있습니다.
> 다만 CLI의 인증 정보는 CLI가 최초로 실행될 때 설정되므로, 데스크톱 앱만 설치되어 있다고 CLI 명령이
> 자동으로 설치되거나 로그인되지는 않습니다.

## 공통: 첫 요청과 종료

세션이 열리면 현재 폴더를 기준으로 요청을 입력합니다. 처음에는
읽기 위주의 요청으로 프로젝트 범위를 확인하는 편이 안전합니다.

```shell
현재 프로젝트의 구조와 실행 방법을 설명해 주세요.
파일을 변경하기 전에 변경 후보와 이유를 먼저 알려 주세요.
```

작업을 마치려면 `/exit`를 입력하거나 `Ctrl+C`를 사용합니다.
새 터미널에서 다시 `claude`를 실행하면 같은 방식으로 사용할 수 있습니다.

## 공통: 업데이트

공식 설치 프로그램은 기본적으로 백그라운드 자동 업데이트를
사용합니다. 즉시 업데이트가 필요하면 다음 명령을 실행합니다.

```shell
claude update
```

Homebrew 설치는 자동 업데이트되지 않습니다. macOS에서 Homebrew로
설치했다면 다음 명령으로 업데이트합니다.

```shell
brew upgrade claude-code
```

## 공통: 설치 확인과 문제 해결

`claude` 명령을 찾지 못한다는 오류가 나오면 새 터미널을 열었는지
먼저 확인합니다. 계속 발생하면 설치 상태와 설정을 진단합니다.

```shell
claude doctor
```

`claude doctor`는 대화 세션을 시작하지 않고 설치 상태, 설정 파일 오류, 업데이트 관련 경고를 읽기
전용으로 보여 줍니다.

설치 경로가 `PATH`에 없다는 메시지가 나오면 공식 문서를 참고합니다.

- [설치 및 로그인 문제 해결 안내](https://code.claude.com/docs/en/troubleshooting)를

위의 문서에 따라서 따라 현재 사용하는 셸의 PATH를 설정합니다.

로그인 창이 열리지 않거나 인증에 실패하면 네트워크 연결, 지원 국가,
계정의 Claude Code 사용 권한을 순서대로 확인합니다. 회사 계정은
관리자가 Team 또는 Enterprise 접근을 제한했을 수 있으므로 관리자에게
확인합니다. API 키를 환경 변수에 직접 저장하는 방식은 키 노출 위험이
있으므로, 개인 로컬 환경에서도 필요성과 보관 위치를 신중히 검토합니다.
