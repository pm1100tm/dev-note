# DOM Collection 다루기

DOM API는 여러 요소를 반환할 때 `NodeList` 또는 `HTMLCollection`을 사용한다. 둘은 배열처럼
보이지만 Array가 아니므로, 지원하는 메서드와 갱신 방식이 다르다.

```js
const nodeList = document.querySelectorAll('.item');
const collection = document.getElementsByClassName('item');
```

- `querySelectorAll`은 일반적으로 정적인 `NodeList`를 반환한다.
- `getElementsByClassName`은 DOM 변경을 반영하는 live `HTMLCollection`을 반환한다.
- 배열 메서드가 필요하면 `Array.from`으로 변환한다.

```js
const titles = Array.from(document.querySelectorAll('h2'));
const texts = titles.map((element) => element.textContent);
```

React에서는 렌더링할 목록을 직접 DOM에서 찾기보다 state를 기반으로 JSX를 렌더링한다. DOM API는
포커스 제어와 외부 라이브러리 연동 같은 제한된 상황에서 `ref`와 함께 사용한다.
