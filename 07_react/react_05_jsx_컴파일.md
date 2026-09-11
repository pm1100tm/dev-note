# JSX 컴파일과 React 런타임

JSX는 브라우저가 이해하는 표준 JavaScript가 아니다. Babel, TypeScript, SWC 같은 변환기가 JSX를
React 엘리먼트를 만드는 함수 호출로 바꾼 뒤, 브라우저는 변환된 JavaScript를 실행한다.

## JSX가 변환되는 이유

다음 JSX는 사람이 읽기에는 화면 구조가 분명하지만, JavaScript 엔진의 문법은 아니다.

```tsx
<p>Hello, world!</p>;
```

변환 결과는 설정한 JSX 런타임에 따라 달라진다.

## Classic 런타임

Classic 런타임은 `React.createElement` 호출로 변환한다. 이 방식에서는 변환된 코드가 `React`를
참조하므로, 소스 파일에서 React를 가져와야 했던 경우가 많다.

```tsx
React.createElement('p', null, 'Hello, world!');
```

## Automatic 런타임

현재 React와 TypeScript 설정에서는 보통 Automatic 런타임을 사용한다. 변환기는 필요한 JSX
런타임 함수를 자동으로 가져오므로, JSX만을 위해 `import React from 'react'`를 작성할 필요가 없다.

```tsx
import { jsx as _jsx } from 'react/jsx-runtime';

_jsx('p', { children: 'Hello, world!' });
```

`tsconfig.json`에서는 일반적으로 다음 설정을 사용한다.

```json
{
  "compilerOptions": {
    "jsx": "react-jsx"
  }
}
```

## 컴포넌트와 props 변환 예시

대문자로 시작하는 `Greeting`은 HTML 태그 문자열이 아니라 JavaScript 변수, 즉 컴포넌트 함수를
가리킨다.

```tsx
<Greeting name="Jin" />;
```

Classic 런타임에서는 개념적으로 다음과 같은 엘리먼트를 만든다.

```tsx
React.createElement(Greeting, { name: 'Jin' });
```

반대로 소문자 `button`은 브라우저 DOM 요소를 뜻하는 문자열로 변환된다.

```tsx
<button type="button">확인</button>;

React.createElement('button', { type: 'button' }, '확인');
```

## 알아둘 점

- JSX 변환은 빌드 단계의 일이며, 컴포넌트 실행이나 DOM 변경 그 자체는 아니다.
- JSX 엘리먼트는 화면의 설명이며, React가 렌더링과 커밋 과정에서 화면 반영을 결정한다.
- 컴파일 결과를 직접 작성할 일은 드물지만, JSX 오류와 런타임 오류를 구분하는 데 도움이 된다.
