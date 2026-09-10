# Set: 중복 없는 값 관리

Set은 같은 값을 한 번만 저장하는 컬렉션이다. 태그 중복 제거, 선택된 ID 집합, 방문 여부처럼
소속 여부를 빠르게 판단해야 하는 상황에 적합하다.

```js
const selectedIds = new Set(['a', 'b']);

selectedIds.add('c');
selectedIds.delete('a');
console.log(selectedIds.has('b')); // true
```

## 배열과 변환하기

```js
const uniqueNames = [...new Set(['Jin', 'Kim', 'Jin'])];
const tags = new Set(['react', 'js']);
const tagArray = Array.from(tags);
```

객체는 내용이 같아도 참조가 다르면 중복으로 판단된다. 객체의 특정 속성 기준으로 중복을 제거하려면
그 속성을 Map의 키로 사용하거나, 별도 비교 로직을 작성한다.

```js
const byId = new Map(users.map((user) => [user.id, user]));
const uniqueUsers = [...byId.values()];
```
