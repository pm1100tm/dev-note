# TypeScript 기초

TypeScript는 JavaScript에 타입 정보를 더한 언어다. 코드를 실행하기 전에 값의 형태가 맞는지
확인할 수 있어, React 컴포넌트의 props와 API 응답처럼 구조가 중요한 코드에서 특히 유용하다.

## REPL로 빠르게 실행하기

파일을 만들지 않고 TypeScript 문법을 확인하려면 `ts-node`를 사용할 수 있다.

```shell
npm install --save-dev typescript ts-node
npx ts-node
```

학습용 명령이며, 실제 프로젝트에서는 빌드 도구나 테스트 환경의 TypeScript 설정을 따른다.

## 타입 지정과 타입 추론

변수에 타입을 명시할 수 있다. 초기값이 충분히 명확하면 TypeScript가 타입을 추론하므로,
모든 변수에 타입을 반복해서 적을 필요는 없다.

```ts
const name: string = 'Kim';
const count = 3;

// count = 'three'; // 오류: number에 string을 대입할 수 없다.
```

## 객체와 함수 타입

객체가 가져야 할 속성은 `type` 또는 `interface`로 표현한다. 함수의 매개변수와 반환값도
명시하면 호출하는 쪽과 구현하는 쪽의 약속이 분명해진다.

```ts
type User = {
  id: number;
  name: string;
  email?: string;
};

function formatUser(user: User): string {
  return `${user.id}: ${user.name}`;
}
```

`?`는 선택 속성을 뜻한다. 즉, `email`은 있어도 되고 없어도 된다.

## Union Type과 좁히기

Union Type은 값이 여러 타입 중 하나일 수 있음을 표현한다. 값을 사용하기 전에는 조건문으로
타입을 좁혀야 안전하다.

```ts
function formatId(id: string | number): string {
  if (typeof id === 'number') {
    return id.toFixed(0);
  }

  return id.toUpperCase();
}
```

## Intersection Type

Intersection Type은 여러 타입의 속성을 모두 갖는 타입을 만든다. 권한 정보를 사용자 정보에
합칠 때처럼, 서로 다른 책임의 데이터를 결합할 때 사용할 수 있다.

```ts
type User = { id: number; name: string };
type Admin = { role: 'admin' };
type AdminUser = User & Admin;

const admin: AdminUser = { id: 1, name: 'Kim', role: 'admin' };
```

## Generics

Generics는 타입을 나중에 받도록 만들어, 자료 구조와 함수의 재사용성을 높인다.

```ts
function first<T>(items: T[]): T | undefined {
  return items[0];
}

const firstNumber = first([1, 2, 3]);
const firstName = first(['Kim', 'Lee']);
```

## Utility Type

Utility Type은 기존 타입을 바탕으로 새 타입을 만드는 내장 도구다. 대표적으로 수정 요청처럼
일부 속성만 받을 때 `Partial`을 사용한다.

```ts
type User = {
  id: number;
  name: string;
  email: string;
};

type UpdateUser = Partial<User>;

const request: UpdateUser = { name: 'Kim' };
```

React에서는 props, state, API 응답의 타입을 먼저 설계하면 컴포넌트 경계에서 발생하는 실수를
줄일 수 있다.
