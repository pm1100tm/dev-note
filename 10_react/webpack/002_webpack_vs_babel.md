# Webpack과 Babel의 차이

Webpack과 Babel은 자주 함께 사용되지만 역할이 다르다.

| 도구    | 역할                                                                                          |
| ------- | --------------------------------------------------------------------------------------------- |
| Webpack | 여러 파일과 의존성을 묶는 번들러(Bundle Bundler)                                              |
| Babel   | 최신 JavaScript 문법을 구형 브라우저에서도 실행할 수 있도록 변환하는 트랜스파일러(Transpiler) |

예를 들어 Babel은 이런 코드를 변환한다.

```js
const add = (a, b) => a + b;
```

```js
var add = function add(a, b) {
	return a + b;
};
```

반면 Webpack은 여러 파일을 분석하고 묶는다.

```
App.tsx
Button.tsx
style.css
logo.png
    ↓
bundle.js
```

즉, Babel은 “문법 변환”이고 Webpack은 “번들링”이다.
