# 다중집합(Multiset)

다중집합은 같은 원소의 중복을 허용하는 집합입니다. 일반 집합이 원소의 존재 여부만 표현한다면, 다중집합은 각 원소가 몇 번
존재하는지 나타내는 **중복도(multiplicity)**를 함께 관리합니다. bag이라고도 부릅니다.

예를 들어 `{사과: 2, 배: 1}`은 사과 두 개와 배 한 개로 이루어진 다중집합입니다. 원소별 개수가 중요하므로 보통 `원소 -> 개수`
형태의 해시 맵으로 구현합니다.

## 집합과의 차이

| 구분      | 집합(Set)                 | 다중집합(Multiset)             |
| --------- | ------------------------- | ------------------------------ |
| 중복 원소 | 허용하지 않음             | 허용함                         |
| 저장 정보 | 원소의 존재 여부          | 원소별 개수                    |
| 대표 활용 | 중복 제거, 포함 여부 검사 | 빈도 계산, 재고, 애너그램 검사 |

## Python의 `Counter`

Python 표준 라이브러리의 `collections.Counter`는 딕셔너리의 하위 클래스이며, 키를 원소로 값은 개수로 저장합니다.

```python
from collections import Counter


items = Counter([1, 2, 2, 3, 3, 3])
assert items == Counter({3: 3, 2: 2, 1: 1})

items.update([2, 4])
assert items[2] == 3
assert items[5] == 0  # 없는 키를 조회하면 0

items.subtract([2, 3])
del items[4]

print(list(items.elements()))
print(items.most_common(2))
```

`update`는 개수를 더하고 `subtract`는 개수를 뺍니다. `subtract` 결과로 0이나 음수인 항목도 `Counter` 내부에
남을 수 있다는 점에 주의해야 합니다. 양수인 항목만 남기려면 단항 `+` 연산을 사용할 수 있습니다.

```python
counts = Counter(a=2, b=1)
counts.subtract(Counter(a=2, b=3))

assert counts == Counter(a=0, b=-2)
assert +counts == Counter()
```

또한 `key in counter`는 개수가 양수인지가 아니라 **키가 저장되어 있는지**를 검사합니다. 개수가 양수인지 확인하려면
`counter[key] > 0`을 사용합니다.

## 다중집합 연산

두 원소의 개수를 각각 `left[x]`, `right[x]`라고 할 때 연산은 다음 의미를 갖습니다.

- 교집합 `left & right`: 원소별 개수의 최솟값
- 합집합 `left | right`: 원소별 개수의 최댓값
- 차집합 `left - right`: 개수를 뺀 뒤 양수인 결과만 유지
- 합 `left + right`: 원소별 개수를 더한 뒤 양수인 결과만 유지

```python
from collections import Counter


left = Counter([1, 2, 2, 3, 3, 3])
right = Counter([2, 3, 3, 4, 4, 4, 4])

assert left & right == Counter({3: 2, 2: 1})
assert left | right == Counter({4: 4, 3: 3, 2: 2, 1: 1})
assert left - right == Counter({1: 1, 2: 1, 3: 1})
assert left + right == Counter({3: 5, 4: 4, 2: 3, 1: 1})
```

원소의 종류 수를 `k`라고 하면 원소 조회와 개수 변경은 평균 `O(1)`, 전체 저장 공간은 `O(k)`입니다. 두 `Counter`의
집합 연산은 관련된 서로 다른 원소 수에 비례합니다.

## 활용 사례

### 애너그램 검사

두 문자열의 문자별 빈도가 같으면 서로 애너그램입니다.

```python
from collections import Counter


def is_anagram(left: str, right: str) -> bool:
    return Counter(left) == Counter(right)


assert is_anagram("listen", "silent")
assert not is_anagram("apple", "apply")
```

### 그 밖의 사례

- 문서의 단어 빈도와 로그 이벤트 횟수 집계
- 상품별 재고 수량과 게임 자원 관리
- 투표·설문 응답 집계
- 두 컬렉션의 중복을 포함한 차이 비교

순서가 중요하거나 같은 값 각각을 독립 객체로 다뤄야 한다면 다중집합만으로는 충분하지 않습니다. 그 경우 리스트, 큐 또는 별도의
식별자를 가진 객체 컬렉션을 사용해야 합니다.
