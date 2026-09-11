# React 리렌더링 이해하기

렌더링은 React가 현재 props, state, context를 바탕으로 화면이 어떻게 보여야 하는지 계산하는 과정이다.
리렌더링은 이 계산을 다시 수행하는 것이며, 실제 DOM 변경은 그 뒤의 커밋 단계에서 필요한 경우에만
일어난다.

## 리렌더링이 시작되는 경우

- 컴포넌트가 처음 화면에 나타날 때
- 해당 컴포넌트의 state 업데이트가 예약될 때
- 부모 컴포넌트가 렌더링되어 자식 컴포넌트를 다시 평가할 때
- 컴포넌트가 읽는 context 값이 바뀔 때

props 변경은 대개 부모의 리렌더링 과정에서 새 props가 전달되는 형태로 나타난다. `React.memo`로
감싼 컴포넌트는 props가 이전과 같다고 판단되면 부모가 렌더링되어도 렌더링을 건너뛸 수 있다.

```tsx
import { memo } from 'react';

type ProfileProps = { name: string };

const Profile = memo(function Profile({ name }: ProfileProps) {
  return <p>{name}</p>;
});
```

## 렌더 단계와 커밋 단계

렌더 단계에서는 React가 컴포넌트를 호출하고 다음 UI를 계산한다. 이 단계에서는 DOM을 직접 변경하지
않아야 하며, 같은 입력이면 같은 결과를 반환하는 순수한 코드로 작성해야 한다.

커밋 단계에서는 계산된 변경을 실제 DOM에 적용한다. 브라우저가 화면을 그린 뒤 `useEffect`가 실행되며,
DOM 크기 측정처럼 화면이 그려지기 전에 동기적으로 처리해야 하는 작업에는 `useLayoutEffect`를 사용한다.

## state는 스냅샷처럼 동작한다

이벤트 핸들러 안의 state 값은 해당 렌더링 시점의 값이다. 여러 업데이트가 이전 값에 의존한다면
함수형 업데이트를 사용한다.

```tsx
function Counter() {
  const [count, setCount] = useState(0);

  function increaseThreeTimes() {
    setCount((current) => current + 1);
    setCount((current) => current + 1);
    setCount((current) => current + 1);
  }

  return <button onClick={increaseThreeTimes}>{count}</button>;
}
```

## 최적화 전에 확인할 것

- 렌더링 자체는 정상적인 동작이므로, 횟수만 보고 문제로 판단하지 않는다.
- 동일한 state 값을 설정하면 React가 렌더링을 생략할 수 있다.
- `memo`와 메모이제이션은 비교 비용도 있으므로, 느린 렌더링을 측정한 뒤 적용한다.
- 객체·배열·함수를 매번 새로 만들면 memoized 컴포넌트의 props 비교가 실패할 수 있다.

리렌더링을 피하는 것보다 상태 구조를 단순하게 유지하고, 실제로 느린 부분을 찾아 개선하는 편이
대부분의 경우 더 효과적이다.
