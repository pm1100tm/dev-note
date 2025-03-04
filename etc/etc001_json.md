# JSON 이란?

- JavaScript Object Notation의 약자이다.
- 다양한 프로그래밍 언어에서 구현 용이성 때문에 널리 사용된다.
- 인터넷상의 데이터 교환, 특히 AJAX 기반 통신에서 중요한 역할을 한다.
- 서버 통신(AJAX)을 위한 기본 데이터 형식이 되었다. (XML 대체)

JSON은

- 속성-값 쌍의 배열 데이터 유형으로 순서가 지정된 값 목록 표현 가능하다.
- 언어 독립적이기 때문에, C, C++, C#, Java, JavaScript 등 다양한 언어에서 사용 가능하다.
- JSON의 공식 인터넷 미디어 유형은 application/json 이다.
- 파일 확장자는 .json 이다.

## 직렬화(JavaScript 개체에서 JSON으로)

JavaScript 객체를 JSON 문자열로 변환하려면 JSON.stringify()를 사용한다.

```javascript
const person = {
  name: 'John Doe',
  age: 30,
  isStudent: false,
  hobbies: ['reading', 'coding'],
  address: {
    city: 'Example City',
    zipCode: '12345',
  },
};

const jsonString = JSON.stringify(person);
```

## 역직렬화(JSON에서 JavaScript 개체로)

JSON 문자열을 다시 JavaScript 객체로 변환하려면 JSON.parse()를 사용한다.

```javascript
const jsonString =
  '{"name":"John Doe","age":30,"isStudent":false,"hobbies":["reading","coding"],"address":{"city":"Example City","zipCode":"12345"}}';

const person = JSON.parse(jsonString);
console.log(person);
```
