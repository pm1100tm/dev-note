# 📚 Mac M1 Homebrew postgre 완전 삭제 및 재설치

Docker 로 PostgreSQL 를 띄우면서 파이썬 스크립트로 database 와 schema 를 생성할 때,
알 수 없는 오류가 계속하여 발생하였다.

- 임의로 설정한 role 이 생성되지 않음
- database 가 생성되지 않음
- schema 가 생성되지 않음
- Datagrip 에서 해당 포트에 대해서 붙을 수 없음

## 🥲 원인은 Homebrew 로 이전에 설치한 old 한 버전의 PostgreSQL 이 로컬에 존재

- 로컬에는 PostgreSQL 을 설치한 적이 없다고 생각함
- DB 는 무조건 Docker 로 붙고 있다고 생각함

기존에 로컬에 Homebrew 로 설치되어있는 것이 원인이 되었다.

> 🎃 정확한 이유를 알려면 재현을 해야 하지만, 지금은 그럴 리소스가 없으므로..
>
> 기회가 되면 꼭 재현해보고 어디서 원인이 발생하는지 알아보기로 한다.

## 자, 그럼 해결 해보자

- 로컬에 Homebrew 로 설치된 기존의 PostgreSQL 완전 삭제

### ☕️ Homebrew PostgreSQL 완전 삭제 방법(M1 기준)

✅ 목표 상태

- postgresql@14, postgresql@16 완전 제거
- 데이터 디렉터리 (/opt/homebrew/var/postgresql\*) 전부 삭제
- LaunchAgent / 서비스 전부 제거
- psql, postgres 바이너리 깨끗한 상태

```shell
#######################################################################
# 현재 상태 확인 (선택)
brew list | grep postgres

╰─ brew list | grep postgres
postgresql@14

brew services list | grep postgres

╰─ brew services list | grep postgres
postgresql@14         started         swd  ~/Library/LaunchAgents/homebrew.mxcl.postgresql@14.plist


#######################################################################
# PostgreSQL 서비스 전부 중지
brew services stop postgresql@16
brew services stop postgresql@14


#######################################################################
# Homebrew 패키지 제거 (핵심)
brew uninstall --ignore-dependencies postgresql@16
brew uninstall --ignore-dependencies postgresql@14

⚠️ --ignore-dependencies
→ 다른 패키지에 끌려서 남는 것 방지


#######################################################################
# Homebrew 캐시 / 잔여 Formula 정리
brew cleanup


#######################################################################
# 🔥 데이터 디렉터리 완전 삭제 (가장 중요)
# Homebrew PostgreSQL 데이터는 여기에 있음 (M1 기준):
ls /opt/homebrew/var | grep postgres

# 보통 이런 것들이 있음:
# - postgresql@14
# - postgresql@16
# - postgres
# 👉 전부 삭제
rm -rf /opt/homebrew/var/postgresql*


#######################################################################
# 설정 파일 & 소켓 파일 제거
rm -rf /opt/homebrew/etc/postgresql*
rm -rf /opt/homebrew/var/log/postgresql*

# 소켓 남아있으면 접속 꼬임:
rm -rf /tmp/.s.PGSQL.*


#######################################################################
# LaunchAgent 찌꺼기 제거 (중요)
ls ~/Library/LaunchAgents | grep postgres

# 있다면 전부 삭제:
rm -f ~/Library/LaunchAgents/homebrew.mxcl.postgresql*

# 그리고 로드 해제 (혹시 살아있을 경우):
launchctl unload ~/Library/LaunchAgents/homebrew.mxcl.postgresql*.plist 2>/dev/null


#######################################################################
# PATH / 심볼릭 링크 정리 확인
which psql
which postgres

# 👉 아무것도 안 나오는 게 정상
# 만약 나온다면:
ls -l /opt/homebrew/bin | grep postgres

# 찌꺼기 있으면 수동 삭제:
rm -f /opt/homebrew/bin/psql
rm -f /opt/homebrew/bin/postgres


#######################################################################
# 완전 삭제 검증 ✅
brew list | grep postgres
psql --version
# 👉 둘 다 아무것도 안 나와야 함


#######################################################################
# 이제 PostgreSQL 16 깨끗하게 재설치
brew install postgresql@16

# PATH 등록 (zsh 기준):
echo 'export PATH="/opt/homebrew/opt/postgresql@16/bin:$PATH"' >> ~/.zshrc
source ~/.zshrc


#######################################################################
# DB 초기화 & 실행
initdb /opt/homebrew/var/postgresql@16

# 서비스 실행:
brew services start postgresql@16

# 확인
psql --version
psql postgres
```

---

## 🧠 왜 이렇게까지 해야 하냐면

| 문제          | 원인                             |
| ------------- | -------------------------------- |
| **role 없음** | 이전 버전 데이터 디렉터리 재사용 |
| **포트 충돌** | 소켓 파일 잔존                   |
| **버전 꼬임** | bin 은 16, data 는 14            |
| **psql 이상** | PATH에 옛 바이너리               |
