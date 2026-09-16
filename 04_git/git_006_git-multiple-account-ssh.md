# 여러 GitHub 계정의 SSH 설정

한 컴퓨터에서 여러 GitHub 계정을 쓸 때는 계정마다 키와 SSH Host 별칭을 분리한다. GitHub의
실제 호스트는 같아도 clone URL에 별칭을 사용하면 원하는 키를 선택할 수 있다.

## 키 생성과 등록

```shell
ssh-keygen -t ed25519 -C "work@example.com" -f ~/.ssh/id_ed25519_work
ssh-keygen -t ed25519 -C "personal@example.com" -f ~/.ssh/id_ed25519_personal
```

- 공개키(`.pub`)만 각 GitHub 계정의 SSH and GPG keys에 등록한다.
- 개인키와 passphrase는 공유하거나 Git 저장소에 추가하지 않는다.
- 키 파일 권한은 `chmod 600 ~/.ssh/id_ed25519_*`로 제한한다.

## `~/.ssh/config` 설정

```sshconfig
Host github-work
  HostName github.com
  User git
  IdentityFile ~/.ssh/id_ed25519_work
  IdentitiesOnly yes

Host github-personal
  HostName github.com
  User git
  IdentityFile ~/.ssh/id_ed25519_personal
  IdentitiesOnly yes
```

## 연결과 remote 확인

```shell
ssh -T git@github-work
git remote set-url origin git@github-personal:account/repository.git
git remote -v
git config user.name
git config user.email
```

SSH 인증 계정과 커밋 작성자 `user.name`, `user.email`은 별개다. 저장소별 작성자 설정은
`git config user.name "..."`처럼 해당 저장소에서 설정한다.
