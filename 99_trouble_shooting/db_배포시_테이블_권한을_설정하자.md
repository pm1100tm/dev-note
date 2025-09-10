# PostgreSQL 배포시 테이블 권한으로 인한 permission denided to access ...

운영 DB에 테이블을 생성하고 배포를 했을 때 아래와 같은 에러가 발생하였다.

```shell
[LOG] permission denided to access ...
```

이유는 appadmin 유저로 테이블을 생성했을 때, 해당 테이블에 대한 조회/생성 등의 권한이 애플리케이션의
유저에게는 있지 않아 발생했던 이슈였다.

- 테이블 생성 유저: appadmin
- 애플리케이션을 사용하는 유저: appwas

appwas 는 Java 의 DB 관련 환경변수로 설정된 유저명이기 때문에, 실행된 애플리케이션에서 DB를 접근
할 때의, 유저는 appwas 인 것이다.

## 해결방법

✅ 1. 현재 접속 중인 사용자 확인

```sql
SELECT current_user;
```

✅ 2. 테이블에 대한 권한 확인

```sql
SELECT grantee, privilege_type
FROM information_schema.role_table_grants
WHERE table_name = 'user_refresh_token';
```

위 쿼리의 결과에서 SELECT, INSERT, UPDATE 등의 모든 권한이 appadmin 에게만 부여된 것을
확인할 수 있었다.

✅ 3. 권한 부여 방법

```sql
GRANT SELECT, INSERT, UPDATE, DELETE, REFERENCES, TRIGGER
ON TABLE user_info_refresh_token
TO appwas;
```

### ETC. 어떤 권한들이 있는가 보면

| 권한       | 설명                                                |
| ---------- | --------------------------------------------------- |
| SELECT     | 데이터를 조회할 수 있음                             |
| INSERT     | 데이터를 삽입할 수 있음                             |
| UPDATE     | 데이터를 수정할 수 있음                             |
| DELETE     | 데이터를 삭제할 수 있음                             |
| REFERENCES | 외래키 참조 권한(다른 테이블이 참조할 수 있게 허용) |
| TRIGGER    | 해당 테이블에 트리거를 생성/사용할 수 있는 권한     |
