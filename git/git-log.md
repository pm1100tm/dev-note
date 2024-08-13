# git log

```shell
# 로그 보기
git log

# 특정 개수의 로그 보기
git log -2

# 각 커밋의 diff 확인하기
git log -p

# 커밋의 통계 정보 확인하기
git log --stat

# 커밋을 한줄로 보기
git log --pretty=oneline

# with format: 커밋 해시 - 저자 이름, 시간 : 요약
git log --pretty=format:"%h - %an, %ar : %s"

# 조회 제한조건
## 시간으로 조회하는 옵션
--since, --after
--until, --before
```
