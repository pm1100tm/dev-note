# Virtual DOM

Virtual DOM(가상돔)에 대해서 알아보자.

## 🤔 DOM(Document Object Model) 이란?

HTML 요소들을 담고 있는 웹페이지를 Document라고 하는데, DOM이란 HTML 요소들을 트리 형태로 표현한
것을 말한다. DOM의 각 트리는 Node(노드)로 구성되어있다. 다시 말하면, DOM의 모든 것은 Node이다.

흔히 DOM을 조작한다고 할 때, getElementById 또는 querySelector 메서드를 사용하여, 원하는
HTML요소를 선택하고 조작한다. DOM의 Node는 아래와 같이 구성된다.

- Document Node(문서 노드): 문서 전체를 나타내는 노드
- Element Node(요소 노드): HTML 요소를 나타내는 노드
- Attribute Node(속성 노드): HTML 요소의 속성을 나타내는 노드
- Text Node(텍스트 노드): - HTML 요소의 텍스트를 나타내는 노드

## 🤔 Virtual DOM 이란?

실제 DOM의 복제본이다. React에서는 가상 DOM을 통하여 실제 DOM을 조작한다.

실제 DOM을 조작하여 HTML 요소를 변경하려면 다음과 같은 과정을 거치게 되는데,

- 1. HTML element 를 선택
- 2. 속성 변경
- 3. 새로운 HTML element 요소 생성
- 4. 생성된 HTML element 추가
- 5. 기존의 HTML element 제거

React에서는 가상 DOM을 사용하여 간접적으로 조작한다. 가상 DOM의 추상화를 통하여 변화를 감지하고,
최소한의 DOM 조작을 수행한다.

React는 항상 두개의 가상돔 객체를 가지고 있다.

- 렌더링 이전 화면 구조를 나타내는 가상돔
- 렌더링 이후 화면 구조를 나타내는 가상돔

Rerendering(리렌더링)이 일어날 때 마다 새로운 내용이 담긴 가상 DOM을 생성한다.
랜더링 이전과 이후의 가상 DOM을 비교하여 어떤 HTML 요소가 변경되었는지 비교한다.
React는 이것을 위해 **Diffing Algorithm** 를 사용한다.

React 에서 컴포넌트가 리렌더링 되는 조건을 간단하게 정리하면 아래와 같다.

- 상태(state)변경
- 속성(props)변경
- 부모 컴포넌트의 리렌더링
- 컴포넌트 생명주기 메서드 호출
- 컨텍스트(Context) 변경

이 과정을 통해서 변경해야 하는 부분만 실제 DOM에 적용한다.
이 과정을 **_Reconciliation(재조정)_** 이라고 한다.

여기서 알아야 할 점은, 변경된 모든 HTML 요소들을 한 번에 실제 DOM에 적용한다는 사실이다.
어떤 페이지의 HTML 요소가 들이 1개가 변경되건 10개가 변경되건 상관없이, 이를 감지하고 한 번에 실제
DOM에 적용시킨다. 즉, 10개가 변경되었을 때 실제 DOM에 적용시키는 것을 10번 반복하지 않는다.

가상 DOM을 사용하는 이유가 퍼포먼스 때문은 아니고(충분히 빠르긴 하지만..), 유지보수성이 뛰어난 것이
핵심이다. 다만, 가상 DOM이 무엇이고 왜 쓰는지 이해하였다면, 최적화 할 수 있는 기법이 존재한다.
