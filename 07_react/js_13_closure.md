# Closure와 렉시컬 스코프

Closure는 함수와 그 함수가 선언된 렉시컬 환경의 조합이다. 내부 함수는 외부 함수 호출이 끝난 뒤에도
자신이 선언될 당시 접근할 수 있었던 변수에 접근할 수 있다.

```js
function createCounter() {
  let count = 0;

  return {
    increment() {
      count += 1;
      return count;
    },
    value() {
      return count;
    },
  };
}

const counter = createCounter();
counter.increment();
console.log(counter.value()); // 1
```

## 활용과 주의점

- 모듈 내부 상태를 외부에 직접 노출하지 않고 함수로 제어할 수 있다.
- 이벤트 핸들러와 콜백이 선언 시점의 변수를 기억하는 원리다.
- 반복문에서 `var`를 쓰면 함수 스코프를 공유해 의도치 않은 값을 참조할 수 있다. 블록 스코프인
  `let` 또는 `const`를 사용한다.

```js
for (let index = 0; index < 3; index += 1) {
  setTimeout(() => console.log(index), 0);
}
```

React Hook의 의존성 배열도 클로저와 관련 있다. Effect나 콜백이 최신 값을 참조하려면 의존성을
정확히 선언하고, 린터 경고를 이유 없이 무시하지 않는다.
