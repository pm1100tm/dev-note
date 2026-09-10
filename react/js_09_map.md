# Map: 키-값 데이터 관리

Map은 임의 타입의 키와 값을 연결하는 컬렉션이다. 반복 가능한 키-값 저장소가 필요하거나 키가
객체일 수 있을 때 일반 객체보다 적합하다.

```js
const cache = new Map([
  ['product:1', { name: '키보드' }],
]);

cache.set('product:2', { name: '마우스' });
cache.delete('product:1');
```

## 순회와 변환

```js
for (const [key, value] of cache) {
  console.log(key, value);
}

const entries = [...cache.entries()];
const object = Object.fromEntries(cache);
```

Object는 프로토타입 속성과 키 변환 규칙을 고려해야 하지만, Map은 저장한 키만 다룬다. 반대로
JSON API의 요청 본문은 일반 객체를 기대하는 경우가 많으므로, 직렬화 경계에서는 변환한다.
