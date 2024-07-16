# LRU cache(Least Recently Used)

## About LRU(Least Recently Used, 최소 최근 사용) cache

- 가장 오랫동안 참조되지 않은 데이터를 교체하는 방식
- 메모리가 제한되어있고, 자주 엑세스하는 데이터에 빠르게 엑세스할 수 있어야 하는 시나리오에서 유용

## LRU cache의 작동 원리

- 데이터가 접근되면 해당 데이터는 캐시의 가장 최근 위치로 이동
- 새로운 데이터가 추가될 때, 캐시가 가득 찬 경우 가장 오랫동안 사용되지 않은 데이터를 제거 후 추가
- 요청된 데이터가 캐시에 존재하면 캐시 히트가 발생하여, 데이터는 가장 최근 위치로 이동
- 요청된 데이터가 캐시에 존재하지 않으면 캐시 미스가 발생하여, 새로운 데이터가 캐시에 추가

### 그림으로 알아보기

이 예시에서는 가장 왼쪽에 있는 데이터가 가장 최신에 [조회/삽입] 데이터라고 할 수 있습니다.

```python
# 초기 상태 + 캐시 크기가 3인 경우
[empty, empty, empty]

# 데이터 1 접근
[1, empty, empty]

# 데이터 2 접근
[2, 1, empty]

# 데이터 3 접근
[3, 2, 1]

# 데이터 4 접근 (캐시가 가득 찬 경우):
[4, 3, 2]  # 1이 제거됨

# 데이터 2 접근 (이미 존재하는 데이터):
[2, 4, 3]

# 데이터 4 접근 (이미 존재하는 데이터):
[4, 2, 3] # 4가 가장 최근 사용된 데이터가 됨
```

## Python에서 LRU 캐시 사용

functools 모듈의 lru_cache 데코레이터를 사용하여 LRU 캐시를 쉽게 구현할 수 있습니다.

```python
from functools import lru_cache


@lru_cache(maxsize=3)
def lrucahce(n: int):
    print(f"Fetching data for {n}")
    return f'return key {n}'

# 리턴 값을 저장하고 있기 때문에 출력은 단, 한 번만 나오게 됩니다.
# Fetching data for 1
lrucahce(1)
lrucahce(1)
lrucahce(1)

lrucahce(1) # Fetching data for 1
lrucahce(2) # Fetching data for 2
lrucahce(3) # Fetching data for 3
```

## LRU 캐시 클래스 구현

```python
from collections import OrderedDict

class LRUCache:
    def __init__(self, max_size):
        self.cache = OrderedDict()
        self.max_size = max_size

    def get(self, key):
        if key in self.cache:
            # 키를 가장 최근 사용된 상태로 갱신
            value = self.cache.pop(key)
            self.cache[key] = value
            return value
        return None

    def put(self, key, value):
        if key in self.cache:
            # 기존 키를 제거하여 최근 사용된 상태로 갱신
            self.cache.pop(key)
        elif len(self.cache) >= self.max_size:
            # 캐시가 가득 찬 경우 가장 오래된 항목 제거
            self.cache.popitem(last=False)
        # 새로운 키-값 쌍 추가
        self.cache[key] = value

    def __str__(self):
        return str(self.cache)


max_size = 3  # 최대 크기를 3으로 설정
lru_cache = LRUCache(max_size)

lru_cache.put("a", "apple")
lru_cache.put("b", "banana")
lru_cache.put("c", "cherry")

print(lru_cache) # 출력: OrderedDict([('a', 'apple'), ('b', 'banana'), ('c', 'cherry')])

# 접근하여 'a'를 최신 상태로 갱신
print(lru_cache.get("a"))  # 출력: apple
print(lru_cache)  # 출력: OrderedDict([('b', 'banana'), ('c', 'cherry'), ('a', 'apple')])

# 새로운 항목 추가로 인해 가장 오래된 'b'가 제거됨
lru_cache.put("d", "durian")
print(lru_cache)  # 출력: OrderedDict([('c', 'cherry'), ('a', 'apple'), ('d', 'durian')])

# 존재하지 않는 키 접근 시 None 반환
print(lru_cache.get("b"))  # 출력: None
print(lru_cache)  # 출력: OrderedDict([('c', 'cherry'), ('a', 'apple'), ('d', 'durian')])

max_size = 3  # 최대 크기를 3으로 설정
lru_cache = LRUCache(max_size)

lru_cache.put("a", "apple")
lru_cache.put("b", "banana")
lru_cache.put("c", "cherry")

print(lru_cache)  # 출력: OrderedDict([('a', 'apple'), ('b', 'banana'), ('c', 'cherry')])

# 접근하여 'a'를 최신 상태로 갱신
print(lru_cache.get("a"))  # 출력: apple
print(lru_cache)  # 출력: OrderedDict([('b', 'banana'), ('c', 'cherry'), ('a', 'apple')])

# 새로운 항목 추가로 인해 가장 오래된 'b'가 제거됨
lru_cache.put("d", "durian")
print(lru_cache)  # 출력: OrderedDict([('c', 'cherry'), ('a', 'apple'), ('d', 'durian')])

# 존재하지 않는 키 접근 시 None 반환
print(lru_cache.get("b"))  # 출력: None
print(lru_cache)  # 출력: OrderedDict([('c', 'cherry'), ('a', 'apple'), ('d', 'durian')])


# LRUCache 클래스
__init__(self, max_size): 최대 캐시 크기를 설정하는 생성자입니다.
get(self, key): 키를 사용하여 값을 가져옵니다. 키가 존재하면 가장 최근 사용된 상태로 갱신합니다.
put(self, key, value): 키-값 쌍을 캐시에 추가합니다. 캐시가 가득 차면 가장 오래된 항목을 제거합니다.
__str__(self): 캐시의 현재 상태를 문자열로 반환합니다.

- put 메서드를 통해 캐시에 문자열 값을 저장합니다.
- get 메서드를 통해 캐시에서 값을 가져오고, 최근 사용된 상태로 갱신합니다.
- 캐시가 가득 찬 상태에서 새로운 값을 추가하면 가장 오래된 항목이 제거됩니다.
```

## 용어

LRU 알고리즘에서는 Cache Hit과 Cache Miss라는 용어에 대해서도 알아두면 좋습니다.

- Cache Hit: CPU가 참조하고자 하는 메모리가 캐시에 존재하고 있을 경우
- Cache Miss: CPU가 참조하고자 하는 메모리가 캐시에 존재하지 않을 경우
