# Virtual DOM과 재조정

React에서 흔히 말하는 Virtual DOM은 React가 UI를 계산하기 위해 메모리에 유지하는 화면의 표현이다.
실제 브라우저 DOM의 완전한 복사본이라기보다, 컴포넌트가 반환한 React 엘리먼트 트리와 이를 바탕으로
계산하는 작업 결과를 가리키는 설명으로 이해하는 편이 정확하다.

## DOM이란

DOM(Document Object Model)은 브라우저가 HTML 문서를 트리 구조의 객체로 표현한 것이다. JavaScript는
DOM API를 통해 요소를 찾고, 속성과 텍스트를 바꾸며, 요소를 추가하거나 제거할 수 있다.

```html
<main>
  <h1>상품 목록</h1>
  <p>총 3개</p>
</main>
```

브라우저는 위 구조를 `main`, `h1`, `p` 노드가 연결된 트리로 다룬다.

## React의 렌더링과 커밋

상태가 바뀌면 React는 컴포넌트를 호출해 다음 화면이 어떤 모습이어야 하는지 계산한다. 이전 결과와
새 결과를 비교해 필요한 변경을 찾는 과정을 재조정(reconciliation)이라고 한다.

그다음 커밋(commit) 단계에서 필요한 변경만 실제 DOM에 적용한다. 즉, 컴포넌트를 다시 호출했다고 해서
모든 DOM 노드를 다시 만들거나 브라우저 화면 전체를 다시 그리는 것은 아니다.

```text
상태 또는 props 변경
        ↓
컴포넌트 렌더링으로 다음 UI 계산
        ↓
이전 결과와 비교해 변경 사항 결정
        ↓
필요한 DOM 변경만 커밋
```

## key가 중요한 이유

목록에서는 `key`로 각 항목의 정체성을 알려 줘야 한다. key가 안정적이면 React는 항목의 추가, 이동,
삭제를 더 정확하게 판단하고 컴포넌트 상태도 의도한 항목에 유지할 수 있다.

```tsx
type Todo = { id: string; title: string };

function TodoList({ todos }: { todos: Todo[] }) {
  return (
    <ul>
      {todos.map((todo) => (
        <li key={todo.id}>{todo.title}</li>
      ))}
    </ul>
  );
}
```

정렬·삽입·삭제가 가능한 목록에 배열 index를 key로 사용하면, 화면 상태가 다른 항목으로 옮겨가는
문제가 발생할 수 있다.

## 성능에 대한 오해

Virtual DOM이 있다고 해서 React 앱이 자동으로 빠른 것은 아니다. 불필요하게 큰 트리를 자주 렌더링하거나
비싼 계산을 렌더링 중에 수행하면 성능 문제는 여전히 발생한다.

- 먼저 React DevTools Profiler 등으로 실제 병목을 측정한다.
- 상태를 필요한 컴포넌트 가까이에 두어 변경 범위를 줄인다.
- `memo`, `useMemo`, `useCallback`은 측정 결과가 있을 때 선택적으로 사용한다.
- DOM 직접 조작은 포커스, 외부 라이브러리 연동처럼 React만으로 표현하기 어려운 경우에 한정한다.
