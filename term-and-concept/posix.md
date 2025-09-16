# 📌 POSIX - Portable Operating System Interface

- POSIX = Portable Operating System Interface
- 유닉스(UNIX) 계열 운영체제의 표준 인터페이스 규격
- 쉽게 말하면, 리눅스/유닉스 시스템에서 프로그램이 파일이나 프로세스를 다룰 때 공통으로 따라야 하는 규칙
  세트

## 📌 POSIX 파일 시스템

POSIX를 따르는 파일 시스템은 다음과 같은 표준 기능을 제공합니다:

- open, read, write, close 같은 표준 파일 API 지원
- 파일과 디렉토리에 권한(permissions), 소유자, 그룹 개념 존재 (rwx, UID, GID)
- 계층적 디렉토리 구조 (/home/ubuntu/data)
- 심볼릭 링크, 하드 링크 지원
- ls, chmod, chown 같은 명령어로 제어 가능

## 📌 AWS EFS와 POSIX

- Amazon EFS는 POSIX 규격을 따르므로,
- EC2 (리눅스)에서 마치 로컬 디스크처럼 표준 파일 명령으로 접근 가능
- touch, vim, chmod, rm 같은 리눅스 명령들이 그대로 동작

👉 반대로, Windows NTFS 같은 파일 시스템은 POSIX를 따르지 않기 때문에 EFS와 호환되지 않습니다.

✅ 쉽게 설명

- POSIX = “리눅스/유닉스 세계의 약속된 규칙”
- POSIX 파일 시스템 = 리눅스 표준 방식으로 동작하는 파일 시스템
