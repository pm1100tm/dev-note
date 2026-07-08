# 브라우저는 어떤 파일을 실행하는가?

## Webpack을 이해하기 전에 알아야 할 것

브라우저는 기본적으로 HTML, CSS, JavaScript를 해석하고 실행한다.

프론트엔드 개발자가 React, TypeScript, SCSS, 이미지, 폰트 등을 사용하더라도 최종적으로 브라우저에
전달되는 결과물은 대부분 다음과 같은 정적 파일이다.

```
HTML
CSS
JavaScript
Image
Font
```

즉, 브라우저는 우리가 작성한 React, TypeScript, JSX, SCSS 자체를 그대로 실행하는 것이 아니다.

<br>

## TypeScript는 브라우저가 직접 실행하지 못한다

개발자는 TypeScript로 코드를 작성할 수 있다.

```ts
const message: string = 'Hello Webpack';

function printMessage(message: string): void {
	console.log(message);
}

printMessage(message);
```

하지만 브라우저는 TypeScript의 타입 문법을 이해하지 못한다.

```ts
const message: string;
```

여기서 : string 같은 타입 문법은 JavaScript 문법이 아니다.
따라서 TypeScript 코드는 브라우저에서 실행되기 전에 JavaScript로 변환되어야 한다.

변환 후에는 대략 이런 형태가 된다.

```js
const message = 'Hello Webpack';

function printMessage(message) {
	console.log(message);
}

printMessage(message);
```

이처럼 브라우저가 실행할 수 있는 코드는 최종적으로 JavaScript여야 한다.

<br>

## JSX도 브라우저가 직접 실행하지 못한다

React에서 자주 사용하는 JSX도 브라우저가 그대로 이해하는 문법은 아니다.

```ts
function App() {
  return <h1>Hello Webpack</h1>
}
```

JSX는 HTML처럼 보이지만 실제로는 JavaScript 확장 문법이다.
따라서 브라우저에서 실행되기 전에 JavaScript 코드로 변환되어야 한다.

대략 다음과 같은 형태로 변환된다.

```js
function App() {
	return React.createElement('h1', null, 'Hello Webpack');
}
```

즉, React 프로젝트에서 작성하는 JSX도 빌드 과정이 필요하다.

<br>

## CSS, 이미지, 폰트도 그냥 자동으로 처리되는 것이 아니다

프론트엔드 프로젝트에서는 JavaScript 파일 안에서 CSS나 이미지를 import하는 경우가 많다.

```ts
import './App.css'
import logo from './logo.png'

function App() {
  return (
    <div>
      <img src={logo} alt="logo" />
      <h1>Hello Webpack</h1>
    </div>
  )
}

export default App
```

- 하지만 원래 JavaScript 입장에서 보면 CSS 파일이나 이미지 파일을 import하는 것은 일반적인
  JavaScript 문법만으로 처리하기 어렵다.
- 이런 파일들을 브라우저가 사용할 수 있도록 적절히 변환하고 연결해주는 과정이 필요하다.
- Webpack은 이 과정에서 CSS, 이미지, 폰트 같은 리소스도 하나의 모듈처럼 다룰 수 있게 도와준다.

<br>

## 브라우저가 이해하는 파일과 개발자가 작성하는 파일은 다르다

| 구분   | 개발자가 작성하는 파일  | 브라우저가 최종적으로 받는 파일     |
| ------ | ----------------------- | ----------------------------------- |
| 마크업 | HTML, JSX, TSX          | HTML                                |
| 스타일 | CSS, SCSS, Tailwind CSS | CSS                                 |
| 로직   | JavaScript, TypeScript  | JavaScript                          |
| 이미지 | PNG, JPG, SVG, WebP     | 이미지 파일 또는 번들에 포함된 경로 |
| 폰트   | WOFF, WOFF2, TTF        | 폰트 파일                           |

프론트엔드 개발 환경에서는 개발자가 작성한 여러 파일을 브라우저가 이해할 수 있는 형태로 변환하는 과정이 필요하다.

## 그래서 빌드 도구가 필요하다

```ts
import React from 'react'
import './Button.css'

type ButtonProps = {
  children: React.ReactNode
  onClick: () => void
}

export function Button({ children, onClick }: ButtonProps) {
  return (
    <button className="button" onClick={onClick}>
      {children}
    </button>
  )
}
```

| 요소                    | 설명                     |
| ----------------------- | ------------------------ |
| `.tsx`                  | TypeScript + JSX 문법    |
| `type ButtonProps`      | TypeScript 타입 문법     |
| `<button>...</button>`  | JSX 문법                 |
| `import './Button.css'` | CSS 파일 import          |
| `export`                | ES Module(ESM) 모듈 문법 |

따라서 이 코드는 그대로 브라우저에 전달되는 것이 아니라, 빌드 도구를 거쳐 브라우저가 실행할 수 있는
JavaScript와 CSS로 변환된다.

이 역할을 하는 대표적인 도구 중 하나가 Webpack이다.
