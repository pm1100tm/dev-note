# JSX 기초

JSX는 JavaScript 파일 안에서 UI 구조를 HTML과 비슷한 문법으로 표현하는 구문 확장이다.
브라우저가 JSX를 직접 실행하는 것은 아니며, 빌드 도구가 React 엘리먼트를 만드는 JavaScript로
변환한다.

## JSX를 사용하는 이유

JSX는 화면 구조와 그 화면에 필요한 값을 가까이 작성하게 해 준다. 마크업과 로직을 파일 기준으로
억지로 나누기보다, 컴포넌트라는 단위에서 함께 관리할 수 있다.

```tsx
type GreetingProps = {
  name: string;
};

export default function Greeting({ name }: GreetingProps) {
  return <h1>안녕하세요, {name}님!</h1>;
}
```

중괄호 `{}` 안에는 JavaScript 표현식을 넣을 수 있다. 변수, 함수 호출, 삼항 연산자는 가능하지만
`if`나 `for` 같은 문은 표현식이 아니므로 그대로 넣을 수 없다.

```tsx
const isLoggedIn = true;

const message = <p>{isLoggedIn ? '환영합니다.' : '로그인해 주세요.'}</p>;
```

## HTML과 다른 점

JSX는 HTML과 비슷하지만 JavaScript 문법을 따르는 부분이 있다.

- CSS 클래스는 `class` 대신 `className`을 사용한다.
- 이벤트 이름은 `onclick` 대신 `onClick`처럼 camelCase로 작성한다.
- 태그는 모두 닫아야 하므로 `<img />`, `<input />`처럼 작성한다.
- 컴포넌트는 대문자로 시작하고, 소문자로 시작하는 태그는 HTML 요소로 취급한다.

```tsx
function Profile() {
  return (
    <section className="profile">
      <img src="/avatar.png" alt="프로필 사진" />
      <button type="button" onClick={() => alert('clicked')}>
        확인
      </button>
    </section>
  );
}
```

## 하나의 부모 요소

컴포넌트는 하나의 JSX 값만 반환한다. 불필요한 DOM 요소를 추가하고 싶지 않다면 Fragment를
사용한다.

```tsx
function Notice() {
  return (
    <>
      <h2>공지</h2>
      <p>새로운 소식이 있습니다.</p>
    </>
  );
}
```

## JSX는 문자열이 아니다

JSX는 HTML 문자열을 조합하는 문법이 아니라 React 엘리먼트를 선언하는 문법이다. 사용자 입력을
문자열로 조합해 HTML로 삽입하는 방식은 XSS 위험이 있으므로 피하고, React의 일반적인 JSX 렌더링
흐름을 사용한다.

## 확인할 점

- 목록을 렌더링할 때는 각 항목에 안정적인 `key`를 지정한다.
- 화면을 나누는 단위는 HTML 파일이 아니라 재사용 가능한 컴포넌트다.
- JSX 안에서 복잡한 조건이 반복되면, 값을 미리 계산하거나 컴포넌트로 분리한다.
