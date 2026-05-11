# Hydration

- SSR 이후 React 가 브라우저에서 다시 연결되는 과정
- "정적인 HTML" 을, "동적인 React 앱" 으로 바꾸는 과정
- 정적 HTML + React 연결 과정

> 즉,

- SSR HTML 은 처음에는 단순 문서 상태이다
- 클릭 안됨 - 상태 없음 - 이벤트 없음

이후 JavaScript 가 로드되면:

```
ReactDOM.hydrateRoot(...)
```

를 통해 아랭의 작업이 수행된다.

- 이벤트 연결
- 상태 복원
- React 활성화

## 왜 Hydration 이 필요한가?

- SSR(Server Side Rendering) 에서는 서버가 HTML 을 미리 만들어서 브라우저에 전달한다.

예를 들어 서버가 이런 HTML 을 보낸다:

```html
<button>Like</button>
```

브라우저는 즉시 화면을 보여줄 수 있다. 그러나, 이것은 단순 문서일 뿐이다.

## Hydration 이 하는 일

브라우저에서 JavaScript 가 다운로드되면, React 가 기존 HTML 을 재사용하면서 이벤트와 상태를 연결

- 서버 렌더링
  - 서버가 HTML 생성
- 브라우저가 HTML 표시
  - 아직 React 는 없음
- JS 다운로드
  - 브라우저가 React bundle 다운로드.
- Hydration 수행
  - HTML 이 이미 있네? 새로 만들지 말고 연결만 하자

### 핵심 특징

Hydration 의 핵심은 -> React 는 HTML 을 재사용한다

- 기존 DOM 재생성 X
- 기존 DOM 재사용 O

### 만약 Hydration 이 없다면?

브라우저가, SSR HTML 버리고 -> 다시 전체 렌더링

### React 18 이후 Hydration 특징

React 18 부터는, 아래의 기능들이 추가

- Selective Hydration
- Streaming SSR
- Suspense Hydration

즉, 필요한 부분부터 점진적으로 Hydration 가능

### Next.js 에서의 Hydration

Next.js App Router 에서는:

- Server Component
- Client Component

가 섞여 있다. 여기서, Client Component 만 Hydration 대상이다.

예시:

Server Component

```js
export default function Page() {
	return <div>Hello</div>;
}
```

Client Component

```js
'use client';

import { useState } from 'react';

export default function Counter() {
	const [count, setCount] = useState(0);

	return <button onClick={() => setCount(count + 1)}>{count}</button>;
}
```

- 이벤트 존재
- 상태 존재

하므로 hydration 필요

---

### 실무에서 중요한 것

**서버 HTML 과 클라이언트 HTML 이 같아야 함**

Hydration 은:

- 서버 HTML === 클라이언트 첫 렌더 결과
  - 를 가정한다

만약 다르면,

- Hydration mismatch 에러 발생

### 가장 흔한 Hydration 오류

#### 1. 랜덤값 사용

```ts
<div>{Math.random()}</div>
```

- 서버는, 0.1234
- 클라이언트는, 0.8238
- 서로 다름 -> **Hydration mismatch** 발생

#### 2. 날짜 사용

```ts
<div>{new Date()}</div>
```

- 서버 시간과 클라이언트 시간이 다를 수 있음.

#### 3. window 사용

```ts
const width = window.innerWidth;
```

- 서버에는 window 없음

#### 4. 브라우저 전용 API 사용

```ts
localStorage.getItem(...)
```

- SSR 시점에는 존재하지 않음

<br>

### 실무 해결 방법

#### 1. Client Component 분리

Client Component 분리 -> 'use client'

```ts
'use client';
```

#### 2. useEffect 사용

```ts
useEffect(() => {
	console.log(window.innerWidth);
}, []);
```

- useEffect 는 브라우저에서만 실행

#### 3. mounted 체크

실무에서 매우 많이 사용

```ts
'use client';

import { useEffect, useState } from 'react';

export default function Component() {
	const [mounted, setMounted] = useState(false);

  useEffect(() => {
    setMounted(true)
  }, [])

  if (!mounted) {
    return null
  }

  return <div>{window.innerWidth}</div>
}
```

왜 mounted 패턴을 쓰는가?

- 초기 SSR, window 없음
- Hydration 이후, 브라우저 API 접근 가능

실무에서 자주 보는 오류

```
“Text content does not match server-rendered HTML”
```

거의 대부분:

- Date
- random
- locale
- window
- localStorage

#### Next.js 에서 특히 주의할 것

- App Router 는 기본이 Server Component

따라서,

```
window
document
localStorage
navigator
```

막 쓰면 에러 발생 가능.

#### Dynamic Import 로 SSR 끄기

- 실무에서 매우 자주 사용
- 이렇게 하면 브라우저에서만 렌더링

```ts
import dynamic from 'next/dynamic';

const Chart = dynamic(() => import('./Chart'), {
	ssr: false,
});
```

---

## Hydration 성능 최적화 팁

Client Component 최소화

```
Client Component
= JS 다운로드 필요
= Hydration 필요
```

Next.js App Router 핵심 철학은:

```
가능하면 Server Component
필요한 부분만 Client Component
```
