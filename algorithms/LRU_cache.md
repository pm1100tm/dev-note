# LRU cache(Least Recently Used)

## 🧐 What is caching

캐싱은 데이터를 읽어오는데 시간이 오래 걸리거나, 실행하는데 오래 걸리는 연산의 결과를 미리 계산하여
필요할 때 최초로 한번만 계산하여 저장해놓고 재사용하는 기술을 의미합니다.

프론트앤드에서는 클리이언트 컴퓨터에 캐시를 두고, 이 캐시에 전에 방문했던 웹페이지의 내용을 저장해놓고
동일한 페이지를 재방문시 이 저장해놓은 사본의 페이지를 보여주는 방식을 취할 수 있습니다.

백엔드에서는 클라이언트에서 받은 요청에 대한 처리 결과를 캐시에 저장해두고, 나중에 동일한 요청이 들어왔을
때, 저장해둔 결과를 그대로 응답합니다. 이렇게 하여 조회 속도를 향상시키고, 서버 단의 부하를 줄일 수
있습니다.

네트워크에서는 프록시 서버나 CDN을 대표적인 캐싱 사례로 볼 수 있습니다.

## 캐싱 전략이란?

일반적으로 캐싱을 위해 사용되는 저장 매체는 가격이 비쌉니다. 따라서, 저장 메체의 용량을 제한하여
최대한 효과적으로 사용하는 전략을 취합니다. 이것을 캐싱 전략이라고 합니다.

캐싱 전략은 캐시 용량이 꽉 찼을 때, 어떤 데이터를 남겨두고, 어떤 데이터를 지워야할지에 대한 방법을
뜻합니다. 캐싱 전략에는 아래와 같은 종류가 있습니다.

- LRU(Least Recently Used) cache
- MRU(Most Recently Used) cache
- LFU(Least Frequently Used) cache

### 캐싱 전략 - LRU cache(Least Recently Used)

여기서는 **LRU cache** 전략에 대해서 알아보고자 합니다.

LRU cache 전략은 **최근에 사용된 데이터일수록 앞으로도 사용될 가능성이 높다**라는 가설을 바탕으로
만들어졌습니다. 이 전략은 가장 오랫동안 참조(사용)되지 않은 데이터를 삭제하여 공간을 확보하고, 조회한
데이터는 가장 최근에 사용된 데이터로 간주합니다. 메모리가 제한되어있고 자주 엑세스하는 데이터에 빠르게
엑세스 할 수 있어야 하는 시나리오에 유용합니다.

LRU cache의 세부 동작방식은 아래와 같습니다.

- 새로운 데이터가 추가될 경우,
  - 캐시의 공간이 가득차지 않은 경우, 데이터를 가장 최근 위치로 이동시킵니다.
  - 캐시의 공간이 가득찬 경우, 가장 오랫동안 사용되지 않은 데이터를 제거하고, 새로운 데이터를 가장 최근
    위치로 이동시킵니다.
- 이미 존재하는 데이터를 조회할 경우,
  - 해당 데이터를 가장 최근 위치로 이동시킵니다.

> 📚 용어도 알아두면 좋습니다.
>
> - Cache Hit: 캐시에 원하는 데이터가 있어, 디스크에 접근하지 않고 데이터를 가져오는 것
> - Cache Miss: 캐시에 원하는 데이터가 없는 것
> - Eviction: 캐시 공간이 꽉차서 데이터를 제거하는 것

### LRU cache 동작방식 알아보기

아래의 예시에서는 가장 왼쪽에 있는 데이터가 가장 최신에 [조회/삽입] 데이터라고 가정합니다.

```python
# 캐시 크기가 3인 초기 상태
[empty, empty, empty]

# 데이터 1 접근, 데이터가 없기 때문에 1추가
[1, empty, empty]

# 데이터 2 접근, 데이터가 없기 때문에 2추가, 2 가장 처음으로
[2, 1, empty]

# 데이터 3 접근, 데이터가 없기 때문에 3추가, 3 가장 처음으로
[3, 2, 1]

# 데이터 4 접근, 4 데이터 없기 때문에 4추가, 4 가장 처음으로, 가장 마지막 원소(1) 제거
[4, 3, 2]

# 데이터 2 접근, 데이터가 있고, 2 가장 처음으로
[2, 4, 3]

# 데이터 4 접근, 데이터가 있고, 4 가장 처음으로
[4, 2, 3] # 4가 가장 최근 사용된 데이터가 됨
```

## Python에서 LRU 캐시 사용

functools 모듈의 lru_cache 데코레이터를 사용하여 LRU 캐시를 쉽게 구현할 수 있습니다.

```python
import random

from functools import lru_cache


def fetch_num(n: int):
    print(f'Fetching num: {n}')
    return n


@lru_cache(maxsize=3)
def get_num(n: int) -> int:
    return fetch_num(n)


def main():
    for _ in range(10):
        get_num(random.randint(0, 10))

    print(get_num.cache_info())


if __name__ == '__main__':
    main()


# Fetching num: 1
# Fetching num: 7
# Fetching num: 2
# Fetching num: 0
# Fetching num: 9
# Fetching num: 5
# CacheInfo(hits=4, misses=6, maxsize=3, currsize=3) << 랜덤으로 변경됨
```

### maxsize 의 의미

lru_cache 데코레이의 매개변수 maxsize는 **캐시에 저장할 수 있는 최대 항목 수**를 지정합니다.
캐시가 이 크기를 초과하면 최근에 가장 적게 사용된 항목은 새 항목을 위한 공간을 확보하기 위해 제거됩니다.

명시적으로 지정하지 않은 경우 maxsize의 기본 값은 128입니다.
이 값을 None으로 설정하면 캐시가 제한 없이 커질 수 있기 때문에 주의해야합니다.

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
- __init__(self, max_size): 최대 캐시 크기를 설정하는 생성자입니다.
- get(self, key): 키를 사용하여 값을 가져옵니다. 키가 존재하면 가장 최근 사용된 상태로 갱신합니다.
- put(self, key, value): 키-값 쌍을 캐시에 추가합니다. 캐시가 가득 차면 가장 오래된 항목을 제거합니다.
- __str__(self): 캐시의 현재 상태를 문자열로 반환합니다.

- put 메서드를 통해 캐시에 문자열 값을 저장합니다.
- get 메서드를 통해 캐시에서 값을 가져오고, 최근 사용된 상태로 갱신합니다.
- 캐시가 가득 찬 상태에서 새로운 값을 추가하면 가장 오래된 항목이 제거됩니다.
```
