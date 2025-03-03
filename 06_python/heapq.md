# Python heapq 모듈

- Python의 heapq 모듈은 힙(heap) 자료 구조를 제공합니다.
- heapq 모듈은 이진 힙(binary heap)으로 구현되어 있습니다.
- Python에서 제공하는 heapq 모듈은 기본적으로 최소힙(min-heap)을 제공한다.

## 이진 힙(binary heap)이란?

min-heap은 이진 트리입니다. 정수배열[3, 5, 8, 9, 7] 를 heapify 하면 다음과 같이 요소들이
배치됩니다.

```plain
    3
   / \
  5   8
 / \
9   7
```

- 3은 루트 노드가 되며, 가장 작은 값이며(기본 min-heap이기 때문에) index 0에 위치합니다.
- 각 노드의 값은 하위 노드의 값보다 작거나 같습니다.
- 인덱스 i 에 있는 요소의 자식은 인덱스 (2 \* i + 1) 및 (2 \* i + 2) 에 위치합니다.

## min-heap 과 max-heap

Python에서 제공하는 heapq 모듈은 기본적으로 min-heap 입니다. max-heap을 만들려면 아래와 같은
코드로 만들 수 있습니다.

```python
import heapq

# Initialize the heap
max_heap = []

# Push elements into the heap (negating the values)
heapq.heappush(max_heap, -9)
heapq.heappush(max_heap, -7)
heapq.heappush(max_heap, -5)
heapq.heappush(max_heap, -3)
heapq.heappush(max_heap, -8)

# Heap as a list (with negated values)
print("Max-Heap as list (negated):", max_heap)  # Output: [-9, -8, -5, -3, -7]

# Pop the largest element (negated, then negate back to original)
largest = -heapq.heappop(max_heap)
print("Largest element:", largest)  # Output: 9
print("Max-Heap after pop (negated):", max_heap)  # Output: [-8, -7, -5, -3]

largest = -heapq.heappop(max_heap)
print("Largest element:", largest)  # Output: 8
print("Max-Heap after pop (negated):", max_heap)  # Output: [-7, -5, -3]

largest = -heapq.heappop(max_heap)
print("Largest element:", largest)  # Output: 7
print("Max-Heap after pop (negated):", max_heap)  # Output: [-5, -3]
```

- min-heap은 항상 가장 작은 요소가 루트에 위차합니다.
- 서브 노드에서 min-heap 규칙은 지켜지지 않을 수 있습니다. 왜냐하면 추가되는 부모 노드와 비교하여
  값이 들어가기 때문입니다.

서브 노드에서 min-heap 규칙은 지켜지지 않을 수 있다는 예시는 아래와 같습니다.

```python
# [3, 5, 8, 9, 7] 에 대한 min-heap

    3
   / \
  5   8
 / \
9   7
```

- index 3, 즉 값 5의 자식 노드에서는 min-heap의 규칙이 지켜지지 않은 것을 확인할 수 있습니다.
- 루트 노드 3, 그리고 그 하위 값 5 와 8이 순차적으로 부모 노드 3의 자식 노드로 추가됩니다.
- 그 후 9라는 값이 들어갈 때 부모 노드 5 보다 9 값이 크기 때문에 그대로 부모 노드 5의 자식 노드로
  추가되게 되는 것입니다.

### 그러면 위와 같은 상태에서 2라는 값을 넣으면 어떻게 될까요?

min-heap에 값을 채워넣을 때는, heap의 맨 끝, 즉 가장 마지막 자식 노드의 위치에 추가된 후, 부모
노드와 비교하여 필요한 위치로 이동하면서 힙의 특성을 유지합니다. 이 과정을 **상향식 heap화**라고
합니다.

- 2라는 값은 부모 노드 8의 왼쪽 자식 노드로 들어가게 됩니다.
- 채워진 값 2는 부모 노드의 값 8과 비교하여, 값이 더 작은 경우에 자리를 교체합니다.
- 따라서, 원래 8은 2로, 2의 값은 8이 되어, 2가 부모 노드가 됩니다.
- 그 후, 2라는 값은 부모 노드 3과 비교하고, 자리 교체 연산을 합니다.
- 결국 2는 루트 노드에 위치하게 되며, 3은 2가 있던 자리, 즉 루트 노드의 오른쪽에 위치하게 됩니다.

최종 결과를 시각화한 모습입니다.

```plain
      2
    /   \
   5     3
  / \   /
 9   7 8
```

값을 출력해보면, [2, 5, 3, 9, 7, 8] 이 됩니다.

## heap 의 데이터 삽입 및 제거에 대한 시간 복잡도는?

시간 복잡도는 O(logN) 으로 동일하게 유지됩니다.

## dictionary 나 이중 배열 등에 대해서 heapq.heapify 함수를 실행한다면?

```python
import heapq

data = {1: 'a', 2: 'b', 3: 'c'}
heapq.heapify(data)

# TypeError: heapify() argument must be list, not dict

data = {1, 3}
heapq.heapify(data)

data = (1,2,3,4)
heapq.heapify(data)
```

- dictionary 는 에러가 발생한다.
- set 는 에러가 발생한다.
- tuple 은 에러가 발생한다.
- 그 외, 정수와 문자열도 에러가 발생한다.
- 이중 배열은 에러가 발생하지 않는다.

이중 배열을 heapify 하면 의도한 대로 동작하지 않을 수 있습니다.. heapq 모듈은 리스트의 요소들 간의
비교를 통해 heap을 구성하는데, 이중 배열의 경우 내부 리스트 간의 비교가 복잡해질 수 있기 때문에,
이중 배열을 heapify 하는 것은 고려하지 않는 것이 나을 수도 있습니다.

## 주요 함수

```python
# item을 힙에 추가합니다.
heapq.heappush(heap, item)

# 힙에서 가장 작은 요소를 제거하고 반환합니다.
heapq.heappop(heap)

# 힙에서 가장 작은 요소를 제거하고 item을 추가합니다.
heapq.heapreplace(heap, item):

# 리스트 x를 힙으로 변환합니다.
heapq.heapify(x):
```

여기서 유의해야 할 점은 heapify 함수입니다. heapify는 함수는 주어진 리스트를 heap 자료구조로 변환
하는데, type 메서드로 자료형을 출력해보면 여전히 list 형인 것을 확인할 수 있습니다. 그러나 이는
잘못된 것이 아닙니다. heapq 모듈은 리스트를 heap처럼 취급하도록 하며 효율적으로 min-heap 연산을
수행할 수 있게 해주며, heapq 모듈에서 제공하는 연산은 기본적으로 리스트를 기반으로 동작합니다.

> heapreplace 과 heappushpop 의 차이
>
> - heapreplace: 현재 가장 작은 값을 pop 하고 리턴한뒤, 새로운 값을 추가
> - heappushpop: 새로운 값을 추가한 뒤, 가장 작은 값을 pop 하여 리턴

## 만약 min-heap 에서 최대 값을 찾아야 한다면?

min-heap에서 최대 값을 추출하는 것은 min-heap의 특성상 바로 접근할 수 없기 때문에 직접적인 방법은
없지만, 아래의 몇 가지 방법으로 해결할 수 있습니다.

### 1. 힙 구조를 유지하면서 최대 값 찾기

min-heap 의 특성상 최대 값은 항상 자식 노드(리프 노드) 중 하나에 위치합니다. 따라서 자식 노드들
중에서 최대 값을 찾아야 합니다. 자식 노드는 힙 리스트의 대략 절반부터 끝까지의 요소들입니다.
예를 들어 길이가 n인 힙 리스트에서 자식 노드는 인덱스 n//2 부터 n - 1 까지의 요소들입니다.

```python
import heapq


def find_max_in_min_heap(heap):
    # 힙의 절반 이후의 요소들 중 최대 값을 찾는다
    n = len(heap)
    max_value = max(heap[n//2:])
    return max_value


heap = [1, 3, 6, 5, 9, 8]
print(find_max_in_min_heap(heap))  # 출력: 9
```

### 2. 모든 요소를 힙에서 제거하면서 최대 값을 찾기

모든 요소를 힙에서 하나씩 제거하면서 최대 값을 찾을 수 있습니다. 하지만 이 방벙븐 힙을 파괴합니다.

```python
import heapq


def extract_max_from_min_heap(heap):
    # 힙의 모든 요소를 제거하면서 최대 값을 찾는다
    max_value = float('-inf')
    while heap:
        value = heapq.heappop(heap)
        max_value = max(max_value, value)
    return max_value


heap = [1, 3, 6, 5, 9, 8]
heapq.heapify(heap)
print(extract_max_from_min_heap(heap))  # 출력: 9
```

### 3. 최대 힙을 사용하는 방법

최소 힙에서 최대 값을 찾는 대시, 최대 힙을 사용하면 최대 값을 쉽게 얻을 수 있습니다.

```python
import heapq


class MaxHeap:
    def __init__(self):
        self.heap = []

    def push(self, item):
        heapq.heappush(self.heap, -item)

    def pop(self):
        return -heapq.heappop(self.heap)

    def max(self):
        return -self.heap[0] if self.heap else None


max_heap = MaxHeap()
for value in [1, 3, 6, 5, 9, 8]:
    max_heap.push(value)


print(max_heap.max())  # 출력: 9
```

## heap 은 언제 사용되는가

- 최소/최대 값을 자주 필요로 할 때
  - 힙은 최소/최대 값을 빠르게 찾고 제거할 수 있으므로, 이 기능이 필요한 알고리즘에 유용합니다.
- 우선순위큐
  - 우선순위가 높은 요소를 먼저 처리해야 하는 경우 힙을 사용할 수 있습니다.
- 정렬된 데이터 유지
  - 데이터 스트림에서 상위 k개의 요소를 유지하거나, 정렬된 데이터를 유지해야 하는 경우 힙을 사용합니다.
- K번째 최소/최대 값
  - 대량의 데이터에서 상위 K개의 최소/최대 값을 찾아야 할 때 유용합니다.

### Visualizing Operations of max-heap for [9, 7, 5, 3, 8]

Insert -9:

```plain
-9
```

Insert -7:

```plain
  -9
  /
-7
```

Insert -5:

```plain
  -9
  / \
-7  -5
```

Insert -3:

```plain
    -9
    / \
  -7  -5
  /
-3
```

Insert -8:

```plain
    -9        -9
    / \       / \
  -7  -5    -8  -5
  / \       / \
-3  -8    -3  -7
```

> -8 값이 들어오면 해당 노드의 부모 -7과 자리륾 교체한다. -->

이상으로 Python 의 heapq 모듈에 대해서 알아봤습니다.

감사합니다.
