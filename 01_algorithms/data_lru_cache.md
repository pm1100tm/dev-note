# LRU 캐시(Least Recently Used)

캐시는 원본 데이터보다 빠르게 접근할 수 있는 저장소에 자주 사용하는 데이터나 연산 결과를 보관하는 기술입니다. 응답 시간을
줄이고 원본 저장소, 외부 API, CPU 등에 가해지는 부하를 낮출 수 있습니다.

캐시는 성능 최적화 수단이지 원본 데이터의 대체물이 아닙니다. 만료, 무효화, 일관성, 장애 시 동작을 함께 설계해야 합니다.

## 캐시 교체 정책

저장 공간이 가득 찼을 때 제거할 항목을 결정하는 규칙을 캐시 교체 정책이라고 합니다.

- **LRU(Least Recently Used)**: 가장 오래 사용되지 않은 항목을 제거합니다.
- **LFU(Least Frequently Used)**: 사용 빈도가 가장 낮은 항목을 제거합니다.
- **MRU(Most Recently Used)**: 가장 최근에 사용된 항목을 제거합니다.

LRU는 최근 사용된 데이터가 다시 사용될 가능성이 높다는 **시간 지역성(temporal locality)**을 활용합니다. 접근 패턴이
순차 스캔에 가깝거나 과거 사용 시점이 미래 사용을 잘 예측하지 못한다면 효과가 떨어질 수 있습니다.

## 주요 용어

- **Cache hit**: 요청한 데이터가 캐시에 있어 즉시 반환하는 경우
- **Cache miss**: 요청한 데이터가 캐시에 없어 원본에서 조회해야 하는 경우
- **Eviction**: 용량 확보를 위해 캐시 항목을 제거하는 작업
- **Hit ratio**: 전체 조회 중 캐시 적중 비율
- **TTL(Time To Live)**: 캐시 항목이 유효한 시간

## 동작 예시

아래에서는 왼쪽이 가장 최근에 사용된 위치이고 캐시 용량은 3입니다.

```text
접근 1: [1]
접근 2: [2, 1]
접근 3: [3, 2, 1]
접근 4: [4, 3, 2]  # 가장 오래된 1 제거
접근 2: [2, 4, 3]  # 2를 최신 위치로 이동
접근 4: [4, 2, 3]
```

## Python의 `functools.lru_cache`

`lru_cache`는 함수의 인수별 반환값을 저장하는 메모이제이션 데코레이터입니다. 인수는 딕셔너리 키로 사용되므로 해시 가능한
값이어야 합니다.

```python
from functools import lru_cache


@lru_cache(maxsize=128)
def fibonacci(n: int) -> int:
    if n < 2:
        return n
    return fibonacci(n - 1) + fibonacci(n - 2)


print(fibonacci(30))
print(fibonacci.cache_info())
fibonacci.cache_clear()
```

- `maxsize`는 저장할 최대 항목 수이며 기본값은 128입니다.
- `maxsize=None`이면 항목을 제거하지 않아 메모리가 계속 증가할 수 있습니다.
- `cache_info()`로 적중·실패 횟수와 현재 크기를 확인할 수 있습니다.
- `cache_clear()`로 저장된 결과를 비울 수 있습니다.

입력은 같지만 외부 상태에 따라 결과가 바뀌는 함수, 부수 효과가 있는 함수, 매번 새 객체를 반환해야 하는 함수에는 결과 캐싱이
적합하지 않을 수 있습니다.

## `OrderedDict`로 구현하기

해시 맵만 사용하면 키 조회는 빠르지만 사용 순서를 효율적으로 갱신하기 어렵습니다. Python의 `OrderedDict`는 키 조회와
순서 변경을 평균 `O(1)`에 지원하므로 간단한 LRU 캐시를 구현할 수 있습니다.

```python
from collections import OrderedDict
from typing import Generic, TypeVar


K = TypeVar("K")
V = TypeVar("V")


class LRUCache(Generic[K, V]):
    def __init__(self, capacity: int) -> None:
        if capacity <= 0:
            raise ValueError("capacity는 1 이상이어야 합니다.")
        self.capacity = capacity
        self.cache: OrderedDict[K, V] = OrderedDict()

    def get(self, key: K) -> V | None:
        if key not in self.cache:
            return None

        self.cache.move_to_end(key)
        return self.cache[key]

    def put(self, key: K, value: V) -> None:
        if key in self.cache:
            self.cache.move_to_end(key)
        self.cache[key] = value

        if len(self.cache) > self.capacity:
            self.cache.popitem(last=False)


cache = LRUCache[str, str](capacity=3)
cache.put("a", "apple")
cache.put("b", "banana")
cache.put("c", "cherry")

assert cache.get("a") == "apple"
cache.put("d", "durian")
assert cache.get("b") is None
```

`get`과 `put`의 평균 시간 복잡도는 모두 `O(1)`이고, 공간 복잡도는 용량을 `c`라 할 때 `O(c)`입니다. 다른 언어에서는
보통 해시 맵과 이중 연결 리스트를 조합해 같은 복잡도를 구현합니다.

> 저장할 값 자체가 `None`일 수 있다면 cache miss와 구분할 수 없으므로 별도의 sentinel 객체나 `KeyError`를 사용하는 편이 안전합니다.

## 운영 환경에서 고려할 점

- TTL과 명시적 무효화 전략 없이 오래된 데이터를 제공할 위험
- 여러 인스턴스의 로컬 캐시 간 데이터 불일치
- 동일한 키의 만료 직후 요청이 원본으로 몰리는 cache stampede
- 키와 값이 계속 증가해 발생하는 메모리 압박
- 민감 정보의 캐시 저장 및 테넌트 간 키 충돌

운영에서는 적중률뿐 아니라 miss 지연 시간, eviction 수, 메모리 사용량도 함께 관찰해야 합니다.
