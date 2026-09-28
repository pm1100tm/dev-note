# Git remote url 변경하기

기존 Git Repository 를 ssh 클론하여 작업하다가, .ssh/config 에서 여러 git 계정 사용을 위해서,
config 의 내용을 변경하였다.

이 때, git remote -v 명령어를 확인하고, origin 을 변경할 필요가 있다.

현재 아래와 같이 출력되는데,

```shell
❯ git remote -v

origin	git@github.com:pm1100tm/dev-note.git (fetch)
origin	git@github.com:pm1100tm/dev-note.git (push)
```

.ssh/config 에서 설정한 값은 조금 다르다.

```shell
# 회사 계정
Host github.com
HostName github.com
User git
IdentityFile ~/.ssh/id_rsa_<회사명>
IdentitiesOnly yes

# 개인 계정 SSH 설정
Host github.com-pm1100tm
HostName github.com
User git
IdentityFile ~/.ssh/id_rsa
IdentitiesOnly yes
```

해당 레포지토리는 git origin 이 github.com-pm1100tm 이 되어야 한다.

## set-url

remote set-url 설정으로 origin 을 변경한다.

```shell
git remote set-url origin git@github.com-pm1100tm:pm1100tm/dev-note.git
```

그 후 다시, remote -v 를 해보면

```shell
❯ git remote -v
origin	git@github.com-pm1100tm:pm1100tm/dev-note.git (fetch)
origin	git@github.com-pm1100tm:pm1100tm/dev-note.git (push)
```

origin 이 `git@github.com` -> `git@github.com-pm1100tm` 로 변경된 것을 확인할 수 있다.

remote 변경 후 다시 전처럼 push, pull 명령어를 정상적으로 사용할 수 있다.
