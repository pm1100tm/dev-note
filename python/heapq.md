# Python heapq 모듈

- Python의 heapq 모듈은 힙(heap) 자료 구조를 제공합니다.
- heapq 모듈은 이진 힙(binary heap)으로 구현되어 있습니다.
- Python에서 제공하는 heapq 모듈은 기본적으로 최소힙(min-heap)을 제공한다.

> min-heap
>
> - min-heap 은 항상 가장 작은 요소가 루트에 위치한다.
> - 서브 노드에서 min-heap은 지켜지지 않을 수 있다.

## 이진 힙(binary heap)

- min-heap은 이진 트리입니다.
- 각 노드의 값은 하위 노드의 값보다 작거나 같습니다.
- 루트 요소는(가장 작은 값)은 항상 index 0에 위치합니다.
- 인덱스 i에 있는 요소의 자식은 인덱스 (2 \* i + 1) 및 (2 \* i + 2) 에 위치합니다.

### [3, 5, 8, 9, 7] 에 대한 min-heap

```plain
    3
   / \
  5   8
 / \
9   7
```

- index 3, 즉 값 5의 자식 노드에서는 min-heap 의 규칙이 지켜지지 않은 것을 확인할 수 있다.
- 이 상태에서 2라는 값을 넣으면, 최 상단 노드(index 0)에 2라는 값이 채워지게 되고, 원래 값 3은
  8의 자리에 위치하게 되며, 8은 3의 왼쪽 자식에 위치하게 된다.

> **min-heap의 데이터 삽입 및 제거에 대한 시간 복잡도는 O(logN) 으로 동일하다.**

## 주요 함수

```python
heapq.heappush(heap, item): item을 힙에 추가합니다.
heapq.heappop(heap): 힙에서 가장 작은 요소를 제거하고 반환합니다.
heapq.heapreplace(heap, item): 힙에서 가장 작은 요소를 제거하고 item을 추가합니다.
heapq.heapify(x): 리스트 x를 힙으로 변환합니다.
```

다음은 heapq 모듈을 사용한 몇 가지 예제입니다.

```python
import heapq

# 힙 초기화
min_heap = []

# 힙에 요소 추가
heapq.heappush(min_heap, 10)
heapq.heappush(min_heap, 20)
heapq.heappush(min_heap, 5)

print("Heap after pushes:", min_heap)  # [5, 20, 10]

# 힙에서 가장 작은 요소 제거 및 반환
smallest = heapq.heappop(min_heap)
print("Smallest element:", smallest)  # 5
print("Heap after pop:", min_heap)  # [10, 20]

# 힙에서 가장 작은 요소를 제거하고 새로운 요소 추가
heapq.heapreplace(min_heap, 15)
print("Heap after replace:", min_heap)  # [15, 20]

# 힙에서 가장 작은 요소를 제거하고, 새로운 요소 추가
heapq.heappushpop(min_heap, 100)

# 리스트를 힙으로 변환
data = [9, 5, 8, 3, 7]
heapq.heapify(data)
print("Heapified list:", data)  # [3, 5, 8, 9, 7]
```

> heapreplace 과 heappushpop 의 차이
>
> - heapreplace: 현재 가장 작은 값을 pop 하고 리턴한뒤, 새로운 값을 추가
> - heappushpop: 새로운 값을 추가한 뒤, 가장 작은 값을 pop 하여 리턴

## heap 은 언제 사용되는가

- 최소/최대 값을 자주 필요로 할 때
  - 힙은 최소/최대 값을 빠르게 찾고 제거할 수 있으므로, 이 기능이 필요한 알고리즘에 유용하다.
- 우선순위큐
  - 우선순위가 높은 요소를 먼저 처리해야 하는 경우 힙을 사용할 수 있다.
- 정렬된 데이터 유지
  - 데이터 스트림에서 상위 k개의 요소를 유지하거나, 정렬된 데이터를 유지해야 하는 경우 힙을 사용한다.
- K번째 최소/최대 값
  - 대량의 데이터에서 상위 K개의 최소/최대 값을 찾아야 할 때 유용하다.

## max-heap 사용해보기

- Python의 heapq에서는 음수 값을 삽입하여 최대 힙을 시뮬레이션한다.

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

### Visualizing Operations for [9, 7, 5, 3, 8]

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

> -8 값이 들어오면 해당 노드의 부모 -7과 자리륾 교체한다.
