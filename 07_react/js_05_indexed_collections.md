# Indexed Collection: Array와 TypedArray

색인된 컬렉션은 숫자 인덱스로 요소에 접근한다. 일반 애플리케이션 데이터에는 `Array`를 사용하고,
바이너리 데이터처럼 고정된 숫자 형식이 필요할 때는 `TypedArray`를 사용한다.

```js
const items = ['apple', 'banana'];

console.log(items[0]); // apple
console.log(items.at(-1)); // banana
console.log(items[99]); // undefined
```

## 생성과 복사

```js
const fromValues = Array.of(1, 2, 3);
const copied = Array.from(fromValues);
const chars = Array.from('abc');
```

`new Array(3)`은 숫자 세 개가 든 배열이 아니라 빈 슬롯 세 개를 가진 배열을 만든다. 값이 세 개인
배열이 필요하면 `Array.of(3)` 또는 `[3]`을 사용한다.

## 변경 메서드와 비변경 메서드

- `push`, `pop`, `shift`, `unshift`, `splice`, `sort`는 원본을 변경한다.
- `map`, `filter`, `slice`, `concat`, `toSorted`는 새 배열을 반환한다.

React state에서는 원본을 바꾸는 메서드보다 새 배열을 반환하는 메서드를 우선 사용한다.

```js
const activeUsers = users.filter((user) => user.active);
const names = users.map((user) => user.name);
```
