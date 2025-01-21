# Git Tag 사용법

## Lightweight Tag

단순히 커밋의 이름을 붙이는 태그입니다. 메타데이터(작성자, 날짜 등)가 없습니다.

```bash
git tag <태그이름>

git tag v1.0.0
```

## Annotated Tag

작성자 정보, 날짜, 메시지 등을 포함한 정식 태그입니다.

```bash
git tag -a <태그이름> -m "<메시지>"

git tag -a v1.0.0 -m "Release version 1.0.0"
```

## 특정 커밋에 태그 추가

```bash
git tag <태그이름> <커밋해시>

git tag v1.0.0 <hashId>
```

## 저장소에 지정된 태그 확인

```bash
git tag
```

### 특정 패턴으로 필터링

```bash
git tag -l "v1.*"
```

## 태그 삭제

```bash
git tag -d <태그이름>

git tag -d v1.0.0
```

### 원격 태그 삭제

```bash
git push origin --delete <태그이름>

git push origin --delete v1.0.0
```

## 태그 푸시

```bash
git push origin <태그이름>
```

## 태그로 체크아웃

```bash
git checkout <태그이름>
```

> 주의: 태그를 체크아웃하면 detached HEAD 가 생성됩니다. 작업하려면 새로운 브랜치를 생성해야 합니다.

```bash
git checkout -b <branch-name> <tag-name>
```

## 태그 재작성

```bash
git tag -f <태그이름>
```
