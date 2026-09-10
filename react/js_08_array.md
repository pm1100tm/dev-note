# JavaScript Array 실전 사용법

배열은 객체이므로 대입과 전개 연산자는 얕은 복사를 수행한다. 중첩 객체까지 독립적으로 복사되는 것은
아니므로, 변경하는 깊이까지 새 객체를 만들어야 한다.

```js
const users = [{ id: 1, profile: { name: 'Jin' } }];
const copied = [...users];

copied[0].profile.name = 'Kim';
console.log(users[0].profile.name); // Kim
```

## 자주 쓰는 조회·변환 메서드

```js
const numbers = [1, 2, 3, 4];

numbers.find((number) => number > 2); // 3
numbers.some((number) => number > 3); // true
numbers.every((number) => number > 0); // true
numbers.filter((number) => number % 2 === 0); // [2, 4]
numbers.map((number) => number * 2); // [2, 4, 6, 8]
```

`delete array[index]`는 값을 제거하지 않고 빈 슬롯을 남긴다. 위치 기준 삭제에는 `splice`를,
불변 업데이트에는 `filter`를 사용한다.

```js
const withoutTarget = items.filter((item) => item.id !== targetId);
```

정렬은 원본을 변경한다. 원본을 보존해야 하면 `toSorted`를 지원하는 환경에서는 이를 사용하고,
그렇지 않으면 `[...items].sort(compare)`로 복사 후 정렬한다.
