# React Hook - useEffect 란?

- useEffect 는 React 컴포넌트가 렌더링된 이후에 실행할 부수 효과를 처리하는 Hook
- 공식문서에 따르면, 컴포넌트를 외부 시스템과 동기화하기 위한 Hook 이라고 설명
  - 여기서 외부 시스템은, 서버 API/브라우저 API/타이머/DOM/Web Socket, LocalStorage
    같은 바깥의 대상을 의미

기본 문법:

```ts
import { useEffect } from 'react';

useEffect(() => {
	// 실행할 코드

	return () => {
		// 정리할 코드
	};
}, [dependencies]);
```

- 첫 번째 인자
  - 렌더링 이후 실행할 함수
- 두 번째 인자
  - 의존성 배열
- return 함수
  - cleanup 함수, 컴포넌트 언마운트 또는 effect 재실행 전에 실행

## 기본 예제

```ts
import { useEffect, useState } from 'react'

export default function Page() {
  const [count, setCount] = useState<number>(0)

  const handleClick = () => {
    setCount((prev) => prev + 1)
  }

  useEffect(() => {
    document.title = `Count: ${count}`
  }, [count])

  return (
    <>
      <div>
        <p>현재 카운트: {count}</p>
        <button onClick={handleClick}>증가</button>
      </div>
    </>
  )
}
```

- 1. 컴포넌트 렌더링
- 2. 화면에 DOM 반영
- 3. useEffect 실행
- 4. count가 변경되면 다시 렌더링
- 5. 렌더링 후 useEffect 다시 실행

### 의존성 배열이 없다면?

이 코드는 렌더링이 발생할 때마다 실행됩니다.

```ts
useEffect(() => {
	console.log('렌더링될 때마다 실행');
});
```

### 의존성 배열에 빈배열을 넣은 경우

컴포넌트가 처음 마운트될 때 한 번만 실행됩니다.

```ts
useEffect(() => {
	console.log('렌더링될 때마다 실행');
}, []);
```

주로 아래와 같은 곳에 사용합니다.

- 최초 API 요청
- 최초 이벤트 리스너 등록
- 최초 localStorage 조회

### 특정 값이 바뀔 때 마다 실행

```ts
useEffect(() => {
	console.log('keyword가 바뀔 때마다 실행');
}, [keyword]);
```

### API 호출 예시

```ts
import { useEffect, useState } from 'react';

interface Use {
	id: number;
	name: string;
	email: string;
}

export default function UserList() {
	const [users, setUsers] = useState<User[]>(null);
	const [isLoading, setIsLoading] = useState<bool>(false);
	const [errorMessage, setErrorMessage] = useState<string | null>(null);

	useEffect(() => {
		const fetchUsers = async () => {
			try {
				setIsLoading(true);
				setErrorMessage(null);

				const response = await fetch('/api/users');
				if (!response.ok) {
					throw new Error('사용자 목록 조회에 실패했습니다.');
				}

				const data: User[] = await response.json();
				setUsers(data);
			} catch (error) {
				setErrorMessage(
					error instanceof Error ? error.message : '알 수 없는 오류',
				);
			} finally {
				setIsLoading(false);
			}
		};

		fetchUsers();
	}, []);

  if (isLoading) {
    return <div>로딩 중...</div>
  }

  if (errorMessage) {
    return <div>{errorMessage}</div>
  }

  return (
    <ul>
      {users.map(user => {
        <li key={user.id}>
          {user.name} / {user.email}
        </li>
      })}
    </ul>
  )
}
```

- 컴포넌트가 처음 렌더링된 후 사용자 목록을 한 번 조회합니다.
- API 응답을 상태에 저장합니다.
- 성공하든 실패하든 로딩 상태를 종료합니다.

---

## cleanup 함수

useEffect 안에서 등록한 작업을 정리해야 할 때 return 을 사용합니다.

```shell
대표적인 예시
- setInterval 제거
- setTimeout 제거
- 이벤트 리스너 제거
- WebSocket 연결 해제
- 구독 해제
```

공식 문서에서도 timer, subscription 같은 리소스는 cleanup이 필요하다고 설명합니다.

```ts
import { useEffect, useState } from 'react'

export default function Timer() {
  const [seconds, setSeconds] = useState<number>(0)

  useEffect(() => {
    const intervalId = window.setInterval(() => {
      setSeconds(prev => prev + 1)
    }, 1000)

    return () => {
      window.clearInterval(intervalId)
    }
  }, [])

  return <div>{seconds}초</div>
}
```

```shell
1. Timer 컴포넌트 마운트
2. setInterval 등록
3. 1초마다 seconds 증가
4. Timer 컴포넌트 언마운트
5. clearInterval 실행
```

---

## ⭐️ useEffect에서 자주 하는 실수

### 1. 의존성 배열 누락

```ts
useEffect(() => {
	console.log(keyword);
}, []);
```

- 이 코드는 keyword가 바뀌어도 다시 실행되지 않습니다.
- keyword를 effect 안에서 사용한다면 보통 의존성 배열에 넣어야 합니다.

### 2. 무한 렌더링

```ts
const [count, setCount] = useState<number>(0);

useEffect(() => {
	setCount(count + 1);
}, [count]);
```

이 코드는 위험합니다.

```
- 렌더링
- useEffect 실행
- setCount 실행
- count 실행
- 렌더링
- 반복
```

만약 필요하다면 조건을 걸어야 합니다.

```ts
useEffect(() => {
	if (count >= 10) {
		return;
	}

	setCount((prev) => prev + 1);
}, [count]);
```

### 3. 계산 로직을 useEffect 안에 넣는 것

```ts
const [price, setPrice] = useState<number>(10000);
const [quantity, setQuantity] = useState<number>(2);
const [totalPrice, setTotalPrice] = useState<number>(0);

useEffect(() => {
	setTotalPrice(price * quantity);
}, [price, quantity]);
```

- 이건 굳이 useEffect가 필요 없습니다.

```ts
const [price, setPrice] = useState<number>(10000);
const [quantity, setQuantity] = useState<number>(2);

const totalPrice = price * quantity;
```

- useEffect는 외부 시스템과 동기화할 때 쓰는 것이 좋습니다.
- 단순 계산은 렌더링 중에 계산해도 됩니다.

### 좋은 사용 예시

```ts
import { useEffect, useState } from 'react';

interface Product {
	id: number;
	name: string;
}

export default function ProductSearch() {
	const [keyword, setKeyword] = useState<string>('');
	const [products, setProducts] = useState<Product[]>([]);

	useEffect(() => {
		if (!keyword.trim()) {
			setProducts([]);
			return;
		}

		const controller = new AbortController();

		const searchProudcts = async () => {
			const response = await fetch(`/api/products?keyword=${keyword}`, {
				signal: controller.signal,
			});

			if (!response.ok) {
				throw new Error('상품 검색에 실패했습니다.');
			}

			const data: Product[] = await response.json();
			setProduct(data);
		};

		searchProducts().catch((error) => {
			if (error instanceof DOMException && error.name === 'AbortError') {
				return;
			}
			console.error(error);
		});

		return () => {
			controller.abort();
		};
	}, keyword);

  return (
    <div>
      <input
        value={keyword}
        onChange={e => setKeyword(e.target.value)}
        placeholder="상품명 입력"
      />
      <ul>
        {products.map(product => (
          <li key={product.id}>{product.name}</li>
        ))}
      </ul>
    </div>
  )
}
```

이 코드에서 중요한 점은

```ts
const controller = new AbortController();
```

- 이전 요청을 취소하기 위한 객체
- signal: controller.signal

fetch 요청에 취소 신호를 연결합니다.

```ts
return () => {
	controller.abort();
};
```

- keyword가 바뀌기 전에 이전 요청을 취소합니다.
- 검색어가 빠르게 바뀌는 상황에서 이전 API 응답이 늦게 도착해서 최신 검색 결과를 덮어쓰는
  문제를 줄일 수 있습니다.

---

## 정리

- useEffect는 이렇게 이해하면 됩니다.

```shell
렌더링 결과를 기준으로,
React 바깥의 무언가와 동기화해야 할 때 사용하는 Hook

- 단순 계산이면 useEffect 쓰지 말기
- 외부 시스템과 연결되면 useEffect 고려하기
```

## 실무에서는 주의할 점

- 1. 의존성 배열을 정확히 넣기
- 2. timer, event listener, subscription은 cleanup 하기
- 3. state 계산용으로 남용하지 않기
