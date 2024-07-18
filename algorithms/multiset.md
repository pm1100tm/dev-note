# multiset

multiset(다중집합)은 동일한 요소가 여러 번 나타날 수 있는 집합의 일반화된 개념입니다. 각 요소가 한 번만 나타날
수 있는 일반 집합과는 달리, multiset은 각 요소가 존재하는 횟수를 추적합니다.

## 🧐 다중집합(multiset)의 주요 개념

- Elements: multiset에 추가할 수 있는 항목입니다.
- Multiplicity: multiset에 각 요소가 표시되는 횟수입니다

### 🧵 파이썬에서의 multiset 구현

파이썬에는 set 라는 자료형이 있습니다. set은 중복을 허용하지 않고, set 객체를 집합으로 인식하여,
집합에 관련된 연산을 할 수 있습니다. 예를 들어, 교집합, 합집합 등의 연산을 할 수 있습니다.

그러나 multiset은 중복을 포함할 수 있기 때문에 set 자료형으로는 multiset을 구현할 수 없습니다.
대신에, 키가 요소이고 값이 각각의 개수인 dictionary 자료형을 사용하여 구현할 수 있으며, 이러한 연산을
편하게 해줄 수 있는 collections 모듈의 Counter 클래스를 사용할 수 있습니다.

> 📚 Counter는 dictionary의 서브클래스입니다.

### Counter 클래스를 활용하여 muliset 구현 및 계산

#### multiset 구현

```python
from collections import Counter


# Creating a multiset
multiset = Counter()

# Adding elements to the multiset
multiset.update([1, 2, 2, 3, 3, 3, 3])
print("multiset:", multiset) # Counter({3: 4, 2: 2, 1: 1})

# Adding more elements
multiset.update([2, 2, 4])
print("multiset:", multiset)  # Counter({2: 4, 3: 4, 1: 1, 4: 1})

# Accessing the count of a specific element
print("Count of element 3:", multiset[3]) # 4

# # Removing elements
multiset.subtract([2, 2])
print("Multiset after removal:", multiset) # Counter({3: 4, 2: 2, 1: 1, 4: 1})

# Converting to a list (elements repeated according to their counts)
multiset_as_list = list(multiset.elements())
print("Multiset as a list:", multiset_as_list) # [1, 2, 2, 3, 3, 3, 3, 4]

# Checking if an element is in the multiset
print("Is 4 in the multiset?", 4 in multiset) # True
print("Is 5 in the multiset?", 4 in multiset) # False

# Removing an element completely
del multiset[4]
print("Multiset after deleting element 4:", multiset) # Counter({3: 4, 2: 2, 1: 1})
```

#### multiset에 대한 집합 연산

```python
from collections import Counter


multiset1 = Counter([1, 2, 2, 3, 3, 3])
multiset2 = Counter([2, 3, 3, 4, 4, 4, 4])
print("multiset1:", multiset1) # Counter({3: 3, 2: 2, 1: 1})
print("multiset2:", multiset2) # Counter({4: 4, 3: 2, 2: 1})

# Convert to list to see the elements with their counts
print("multiset1 as a list:", list(multiset1.elements())) # [1, 2, 2, 3, 3, 3]
print("multiset2 as a list:", list(multiset2.elements())) # [2, 3, 3, 4, 4, 4, 4]

# Calculate intersection
intersect_multiset = multiset1 & multiset2
print("Intersection:", intersect_multiset) # Counter({3: 2, 2: 1})

# Calculate union
union_multiset = multiset1 | multiset2
print("union:", union_multiset) # Counter({4: 4, 3: 3, 2: 2, 1: 1})

# Calculate difference
difference_multiset = multiset1 - multiset2
print("difference:", difference_multiset)  # Counter({1: 1, 2: 1, 3: 1})
```

### multiset(also known as bags)는 왜, 어디에 사용되는가

요소의 빈도가 중요한 다양한 사나리오에서 유용합니다. 고유한 요소만 저장하는 집합과 달리 다중 집합은
동일 요소가 여러 번 발생할 수 있고, 그 수를 추적할 수 있습니다. 따라서, 요소의 발생 횟수를 추적하는
시나리오에 유용합니다. 예를 들어 텍스트 분석에서 단어 빈도 수, 창고에서의 재고 수 또는 이벤트 발생 횟수
추적 등이 있습니다.

#### Text Processing

- 단어 빈도 수: 문서에서 단어의 빈도를 계산하여 텍스트를 분석하거나 스팸을 감지하거나 검색 엔진용 문서
  색인을 생성합니다.
- 애너그램(Anagram) 탐지: 두 문자열의 문자 수를 비교하여 두 문자열이 아나그램인지 확인힙니다.

> 📚 Anagram
>
> - 어떠한 단어의 문자를 재배열하여 다른 뜻을 가지는 다른 단어로 바꾸는 것을 말한다.

#### Data Analysis

- 통계 분석: 히스토그램이나 빈도 분포와 같은 통계 분석을 위해 데이터 포인트의 발생을 추적합니다.
- 설문 조사 분석: 동일한 질문에 대해 여러 개의 응답이 가능한 설문조사 또는 설문지에서 응답을 계산합니다.

#### Inventory Management

- 재고 계산: 각 품목의 수량을 포함하여 상점의 재고 품목을 추적합니다
- 리소스 관리: 사용 가능한 리소스의 수가 중요한 게임 개발과 같은 앱에서 리소스를 관리합니다.

#### Event Counting

- 로깅 및 모니터링: 모니터링 및 경고 목적으로 시스템 로그에서 이벤트 발생 횟수를 계산합니다.
- 사용자 활동 추적: 사용자 행동을 이해하거나 분석 목적으로 애플리케이션에서 사용자 작업 또는 이벤트를
  추적합니다.
