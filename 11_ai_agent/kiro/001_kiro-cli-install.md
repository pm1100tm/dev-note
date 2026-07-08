# 맥북 kiro-cli 설치하기

- https://kiro.dev/cli/

## cmd

```shell
curl -fsSL https://cli.kiro.dev/install | bash
```

```shell
❯ kiro-cli --version
kiro-cli 2.11.1
```

## SHELL 환경 변수 추가

export PATH="$PATH:$HOME/.local/bin 를 터미널에서만 실행하면, 해당 터미널 세션에서만,
kiro-cli 를 사용할 수 있습니다. 따라서, 환경변수로 추가하여 언제 어디서나 사용할 수 있도록 합니다.

```shell
# kiro-cli PATH 설정
export PATH="$PATH:$HOME/.local/bin"
```
