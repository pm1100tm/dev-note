# REST API 기초

REST는 HTTP를 이용해 리소스를 다루는 API 설계 스타일이다. URL은 동사보다 리소스를 나타내고,
HTTP 메서드와 상태 코드는 수행한 작업과 결과를 표현한다.

## 리소스와 HTTP 메서드

```text
GET    /products        상품 목록 조회
GET    /products/42     상품 한 건 조회
POST   /products        상품 생성
PATCH  /products/42     상품 일부 수정
DELETE /products/42     상품 삭제
```

서버는 각 요청을 독립적으로 처리한다. 이를 무상태성이라고 하며, 인증 정보와 요청 처리에 필요한
데이터는 요청마다 전달한다.

## 자주 쓰는 상태 코드

- `200 OK`: 요청을 성공적으로 처리했다.
- `201 Created`: 리소스를 새로 만들었다.
- `204 No Content`: 성공했지만 응답 본문이 없다.
- `400 Bad Request`: 요청 형식이나 값이 잘못됐다.
- `401 Unauthorized`: 인증이 필요하거나 인증 정보가 유효하지 않다.
- `403 Forbidden`: 인증되었지만 권한이 없다.
- `404 Not Found`: 대상 리소스가 없다.
- `409 Conflict`: 현재 상태와 충돌한다.
- `500 Internal Server Error`: 서버 내부 오류다.

## fetch 요청 예시

```js
async function getProduct(id) {
  const response = await fetch(`/api/products/${id}`);

  if (!response.ok) {
    throw new Error(`상품 조회 실패: ${response.status}`);
  }

  return response.json();
}
```

`fetch`는 HTTP 4xx·5xx 응답만으로 reject되지 않는다. `response.ok`를 확인해 성공과 실패를
명시적으로 나눈다.
