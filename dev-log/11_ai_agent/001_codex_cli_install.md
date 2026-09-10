# Codex CLI 설치 방법

- https://learn.chatgpt.com/docs/codex/cli

## homebrew 로 설치하기

```shell
❯ brew install --cask codex

# Homebrew 메타데이터 다운로드 출력은 생략한다.
==> Would install 1 cask:
codex
==> Would install 1 dependency for codex:
ripgrep
==> Do you want to proceed with the installation? [y/n]
==> Fetching downloads for: codex
# Codex cask 검증 출력은 생략한다.
==> Installing dependencies: ripgrep
==> Fetching downloads for: ripgrep
# ripgrep 의존성 다운로드·설치 출력은 생략한다.
==> Installing ripgrep
==> Pouring ripgrep--15.1.0.arm64_sonoma.bottle.tar.gz
🍺  /opt/homebrew/Cellar/ripgrep/15.1.0: 14 files, 6.5MB
==> Installing Cask codex
==> Linking Binary 'codex-aarch64-apple-darwin' to '/opt/homebrew/bin/codex'
🍺  codex was successfully installed!
```

## 설치 및 버전 확인

```shell
❯ codex --version
codex-cli 0.144.1
```

## codex cli 사용해보기

- Terminal 에 codex 입력 후 실행해보면 접속 가능하다.
- 개인 맥북에서는 codex login 명령어로 인증이 필요했다.

```shell
❯ codex

╭───────────────────────────────────────╮
│ >_ OpenAI Codex (v0.144.1)            │
│                                       │
│ model:     gpt-5.5   /model to change │
│ directory: ~/Desktop/dev-note         │
╰───────────────────────────────────────╯

  Tip: New Use /fast to enable our fastest inference with increased plan usage.

- 사용량 한도가 남아 있으면 `/usage` 명령으로 확인하고 관리할 수 있다.


› Explain this codebase

  gpt-5.5 default · ~/Desktop/dev-note
```
