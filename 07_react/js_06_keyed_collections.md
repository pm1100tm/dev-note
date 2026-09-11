# Keyed Collection: Map과 Set

키 기반 컬렉션은 인덱스 대신 키 또는 값의 정체성으로 데이터를 다룬다. `Map`은 키-값 쌍을,
`Set`은 중복 없는 값의 집합을 표현한다.

## Map

Map의 키에는 문자열뿐 아니라 객체와 함수도 사용할 수 있다. 삽입 순서를 기억하며, `size`로
개수를 확인한다.

```js
const scores = new Map();
scores.set('Jin', 90);
scores.set('Kim', 95);

console.log(scores.get('Jin')); // 90
console.log(scores.has('Lee')); // false
```

## Set

Set은 같은 값의 중복을 제거할 때 유용하다. 객체는 내용이 같아도 서로 다른 참조면 다른 값이다.

```js
const tags = new Set(['react', 'js', 'react']);
tags.add('typescript');

console.log([...tags]); // ['react', 'js', 'typescript']
```

JSON으로 보내야 한다면 Map과 Set을 배열 또는 일반 객체로 변환해야 한다. `JSON.stringify`는
Map과 Set의 항목을 자동으로 일반 JSON 구조로 직렬화하지 않는다.
