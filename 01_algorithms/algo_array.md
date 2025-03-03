# 배열(Array)

- 데이터가 연속적으로 이어져서 랜덤 엑세스를 지원하는 데이터 구조이다.
- 랜덤 엑세스는 인덱스를 통해서 각 요소에 접근하게 하는 것을 가능하게 해준다.
- 배열에 관한 문제는 대부분 정렬(sorting)과 관련있다.
  - heap sort(unstable)
  - quick sort(unstable)
  - merge sort(stable)
- 정렬 알고리즘은 O(N log N)이며, 이것을 암기한다.
- search 연산은 O(N)이다. 배열의 모든 요소들을 돌면서 찾아야 하기 때문이다.
- 그러나 배열이 정렬되어있다면, binary search를 사용할 수 있으며 이것은 O(logN)이다.

## quick sort

- pivot 과 partitioning 개념이 들어있다.
- 기준점 하나를 잡은 뒤(pivot) 나머지 요소들은 partitioning 해준다.

## binary search

- 배열이 어떤 규칙에 의해서 정렬이 되어있다고 가정한다.
- left, right 포인터를 설정하고, 그 중간이 되는 pivot 포인터를 찾는다.
- pivot 포인터가 가리키는 값와 타겟 수가 같다면 리턴하고
- 작다면, right 포인터를 pivot + 1 로 옮겨주고
- 크다면, left 포인터를 pivot -1 로 옮겨준다.
