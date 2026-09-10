# JSON: 데이터 교환 형식

JSON(JavaScript Object Notation)은 언어에 독립적인 텍스트 기반 데이터 형식이다. HTTP API에서
요청과 응답 본문을 전달할 때 주로 사용하며 미디어 타입은 `application/json`이다.

## 문법 규칙

- 객체의 키는 큰따옴표로 감싼 문자열이어야 한다.
- 값은 문자열, 숫자, 불리언, `null`, 배열, 객체만 사용할 수 있다.
- 함수, `undefined`, 주석, 끝의 쉼표는 JSON 문법이 아니다.

```js
const text = '{"name":"Jin","roles":["user"],"active":true}';
const user = JSON.parse(text);

const body = JSON.stringify({ name: user.name, active: user.active });
```

`JSON.parse`는 형식이 잘못된 입력에서 예외를 던진다. 외부 입력은 `try...catch`로 처리하고,
파싱에 성공해도 API 응답의 구조와 타입을 별도로 검증해야 한다.

```js
function parseJson(text) {
  try {
    return JSON.parse(text);
  } catch {
    return null;
  }
}
```

`JSON.stringify`는 객체를 문자열로 바꾼다. 날짜는 문자열이 되고, 순환 참조 객체는 직렬화할 수
없다는 점을 주의한다.
