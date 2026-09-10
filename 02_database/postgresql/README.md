# PostgreSQL

PostgreSQL을 설치하고 접속한 뒤, 역할·권한·DDL·데이터 타입·백업과 복원까지
실무에서 자주 사용하는 내용을 정리합니다.

## 문서 목록

- [macOS 설치와 접속 문제 해결](01.install-mac.md)
- [기본 명령과 역할·권한](02.command.md)
- [DDL(Data Definition Language)](03.ddl.md)
- [데이터베이스 복사와 백업·복원](04.copy.md)
- [PostgreSQL 데이터 타입](05.data-type.md)
- [`pg_dump` 명령 모음](006_dump_command.md)
- [`LATERAL` 문법](007_lateral.md)
- [쿼리 결과를 세로로 보기](008_query_result_vertically.md)

## 사용 전 확인할 것

1. PostgreSQL 서버가 실행 중인지 확인합니다.
2. 접속 대상 호스트, 포트, 데이터베이스, 역할을 명시합니다.
3. 운영 데이터베이스에서는 `DROP`, `TRUNCATE`, `pg_terminate_backend` 실행 전에
   대상과 영향 범위를 다시 확인합니다.
4. 백업 파일은 복원 테스트를 통과해야 백업으로 간주합니다.
