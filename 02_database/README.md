# 데이터베이스

데이터베이스를 설계하고 운영하면서 익힌 SQL, 인덱스, 실행 계획, PostgreSQL 명령과
실전 쿼리를 정리합니다. 예제의 기본 문법은 PostgreSQL을 기준으로 하며, 다른 DBMS와
문법이나 동작이 다른 경우 문서에서 따로 표시합니다.

## 문서 구성

### 핵심 개념

- [인덱스란 무엇이며 어떻게 설계하는가](index.md)
- [EXPLAIN과 EXPLAIN ANALYZE 비교](Q_001_explain_explain_analyze.md)
- [실행 계획 분석 체크리스트](Q_002_실행계획.md)

### PostgreSQL

- [PostgreSQL 목차](postgresql/README.md)
- 설치, `psql` 명령, DDL, 백업·복원, 데이터 타입
- `LATERAL`, 세로형 쿼리 결과, `pg_dump` 활용

### SQL Cookbook

자주 다시 찾아보는 문자열 처리, NULL, 정렬, 집합 연산, 조인, 집계, 서브쿼리 예제를
[cookbook](cookbook/)에서 확인할 수 있습니다.

## 쿼리를 작성하고 검증하는 순서

1. 요구사항과 결과의 중복·NULL·정렬 기준을 먼저 정의합니다.
2. 조인 전에 각 테이블의 카디널리티와 관계(1:1, 1:N)를 확인합니다.
3. 작은 데이터로 결과를 검증한 뒤 `EXPLAIN`으로 실행 계획을 확인합니다.
4. 실제 실행 시간이 필요하면 테스트 데이터와 트랜잭션 상태를 확인한 뒤
   `EXPLAIN ANALYZE`를 사용합니다.
5. 운영 환경에서는 DML을 실행하는 `EXPLAIN ANALYZE`의 부작용과 잠금, 실행 시간을
   반드시 검토합니다.

> 인덱스는 조회를 빠르게 만들 수 있지만 INSERT·UPDATE·DELETE 비용과 저장 공간을
> 증가시킵니다. 실행 계획과 실제 접근 패턴을 근거로 추가해야 합니다.
