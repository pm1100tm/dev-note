# JSX

## 🤔 What is JSX(Java Script XML)

- HTML과 유사한 마크업을 작성할 수 있는 JavaScript의 구문확장자이다.
- JavaScript 코드 안에서 HTML과 비슷한 구문을 사용할 수 있게 해준다.

JSX 를 활용하면 아래와 같은 작업이 가능하다.

- HTML과 같은 유사한 마크업 코드 작성 가능
- JavaScript 표현식 삽입이 가능하여, 마크업 내에서 동적 콘텐츠와 JavaScript 코드 실행 가능
- 재사용 및 독립적인 UI조각

아래의 코드는 HTML, JavaScript가 아닌 **.jsx** 파일 안에 정의된 코드이다.

```javascript
const element = <h1>Hello, world!</h1>;
```

React에서는 HTML과 로직을 분리하는 대신, 이 둘을 포함하는 **컴포넌트(Component)** 라고 부르는
UI조각을 만들고 조합한다

## ⚡️ JSX 의 특징

### 1. HTML 유사 구문

```javascript
const element = <h1>Hello, JSX!</h1>;
```

### 2. JavaScript 표현식

JSX를 사용하면 중괄호({}) 안에 js 표현식을 삽입할 수 있다. 이를 통해 동적 콘텐츠를 마크업에 포함한다.

```javascript
const name = 'John';
const element = <h1>Hello, {name}!</h1>;
```

JSX는 HTML과 유사해 UI 구조와 유사한 방식으로 태그와 속성을 작성할 수 있다.

### 3. 일반적으로 React 구성요소와 함께 사용

- React에서만 사용되는 것이 아니며, JSX만 독립적으로도 사용할 수 있다.

아래는 React에서 Greeting 이라는 컴포넌트에 대한 예제 코드이다.

```jsx
// Greeting.jsx
function Greeting(props) {
  return <h1>Hello, {props.name}!</h1>;
}

export default Greeting;

// App.jsx
function App() {
  return (
    <Greeting name='John' />;
  )
}
```

## JSX는 변환이 필요하다

- JSX 코드는 브라우저에서 직접 인식되지 않아, JavaScript 코드로 변환해야 한다.
- 변환에 사용되는 도구 중의 하나가 **Babel** 이며, 이것은 JSX를 JavaScript 코드로 컴파일하는데
  사용된다.
- 컴파일이 끝나면 JSX 표현식이 정규 JavaScript 함수 호출이 되고, JavaScript의 객체로 인식된다.

### Babel 에서 테스트 해보기

1. [Babel 접속하기](https://babeljs.io/)
2. Try it out -> PRESETS -> react 체크
3. OPTIONS -> React Runtime 의 Classic / Automatic 중 하나 선택
4. Babel 변환기 하단의 **Add Plugin** 버튼을 클릭
5. @babel/plugin-transform-react-jsx 검색 후 선택

```jsx
<Greeting name='world' />;

// --- 변환
/*#__PURE__*/ React.createElement(Greeting, {
  name: 'world',
});
```

```jsx
<Button type='submit'>Send</Button>;

// --- 변환
/*#__PURE__*/ React.createElement(
  Button,
  {
    type: 'submit',
  },
  'Send',
);
```

```jsx
// JSX
<div className='test'>
  <p>Hello, world!</p>
  <Button type='submit'>Send</Button>
</div>;

// --- 변환
React.createElement(
  'div',
  {
    className: 'test',
  },
  React.createElement('p', null, 'Hello, world!'),
  React.createElement(Button, { type: 'submit' }, 'Send'),
);
```

```jsx
<div>
  <p>Count: {count}!</p>
  <button type='button' onClick={() => setCount(count + 1)}>
    Increase
  </button>
</div>;

// --- 변환
React.createElement(
  'div',
  null,
  React.createElement('p', null, 'Count: ', count, '!'),
  React.createElement(
    'button',
    { type: 'button', onClick: () => setCount(count + 1) },
    'Increase',
  ),
);
```
