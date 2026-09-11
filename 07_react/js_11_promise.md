# Promise 깊이 이해하기

Promise는 미래에 완료될 비동기 작업의 결과를 표현하는 객체다. 네트워크 요청, 타이머, 파일 읽기처럼
나중에 성공하거나 실패하는 작업을 순서와 오류 처리 규칙을 갖춰 조합할 수 있게 한다.

## 상태와 정착

Promise는 `pending`에서 시작해 한 번만 `fulfilled` 또는 `rejected` 상태로 정착한다. 정착한
Promise의 상태와 결과는 바뀌지 않는다.

```js
const promise = new Promise((resolve, reject) => {
  setTimeout(() => resolve('완료'), 1000);
});
```

생성자 안의 함수는 Promise를 만드는 즉시 실행된다. 따라서 이미 Promise인 작업을 재포장하기보다,
Promise를 반환하는 API를 그대로 조합하는 편이 좋다.

## 체이닝과 값 전달

`then`은 새 Promise를 반환한다. 콜백에서 값을 반환하면 다음 `then`으로 전달되고, Promise를
반환하면 그 Promise가 끝날 때까지 다음 단계가 기다린다.

```js
fetch('/api/products/1')
  .then((response) => {
    if (!response.ok) throw new Error('상품 조회 실패');
    return response.json();
  })
  .then((product) => product.name)
  .then((name) => console.log(name))
  .catch((error) => console.error(error));
```

`throw`와 `Promise.reject`는 가장 가까운 뒤쪽 `catch`로 전달된다. `catch`에서 오류를 처리하고
값을 반환하면 이후 체인은 다시 성공 상태로 이어진다.

## async와 await

`async` 함수는 항상 Promise를 반환한다. `await`는 Promise가 정착할 때까지 해당 async 함수의
실행을 잠시 멈춘 것처럼 작성하게 해 주며, 실패는 `try...catch`로 처리한다.

```js
async function getProduct(id) {
  try {
    const response = await fetch(`/api/products/${id}`);

    if (!response.ok) throw new Error(`HTTP ${response.status}`);

    return await response.json();
  } catch (error) {
    console.error('상품 조회 실패', error);
    throw error;
  }
}
```

## 여러 작업 조합하기

| API | 완료 조건 | 실패 처리 |
| --- | --- | --- |
| `Promise.all` | 모두 성공 | 하나라도 실패하면 즉시 reject |
| `Promise.allSettled` | 모두 정착 | 성공·실패 결과를 모두 반환 |
| `Promise.race` | 가장 먼저 정착 | 첫 성공 또는 실패를 따름 |
| `Promise.any` | 첫 성공 | 모두 실패하면 reject |

서로 의존하지 않는 요청은 동시에 시작한 뒤 `Promise.all`로 기다린다.

```js
const userPromise = fetch('/api/me').then((response) => response.json());
const noticesPromise = fetch('/api/notices').then((response) => response.json());

const [user, notices] = await Promise.all([userPromise, noticesPromise]);
```

## 취소와 흔한 실수

Promise 자체에는 범용 취소 기능이 없다. `fetch`는 `AbortController`로 취소 신호를 전달한다.

```js
const controller = new AbortController();
const response = await fetch('/api/products', { signal: controller.signal });
controller.abort();
```

- `forEach` 안의 async 콜백은 기다려지지 않는다. 순차 처리에는 `for...of`, 병렬 처리에는
  `Promise.all`을 사용한다.
- `fetch`는 HTTP 오류 상태만으로 reject되지 않으므로 `response.ok`를 확인한다.
- 처리하지 않은 reject는 오류를 숨길 수 있으므로 체인의 끝에서 오류 처리 정책을 정한다.
