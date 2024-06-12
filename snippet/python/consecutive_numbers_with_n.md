# 주어진 n 개의 연속되는 숫자의 존재 여부 판단하기

## 연속된 숫자의 갯수 세기

```python
def is_consecutive_numbers_1(numbers: list[int], n: int) -> bool:
    sorted_numbers = sorted(set(numbers))
    length = len(sorted_numbers)

    if length < n:
        return False

    count = 1
    result = False
    for i in range(length - 1):
        if sorted_numbers[i + 1] - sorted_numbers[i] != 1:
            count = 1
            continue

        count += 1
        if count == n:
            result = True
            break

    return result
```

## Sliding Window

연속되는 n 개의 정수의 처음과 끝의 값을 뺀 값은 n - 1 규칙을 활용한다.

```python
def is_consecutive_numbers_2(numbers: list[int], n: int) -> bool:
    sorted_number = sorted(set(numbers))

    if len(sorted_number) < n:
        return False

    for i in range(len(sorted_number) - n + 1):
        if sorted_number[i + n - 1] - sorted_number[i] == n - 1:
            return True

    return False
```

## Hash Set

각 배열의 원소에 대한 n 개의 연속적인 정수 배열을 생성 후, 기존의 배열에 존재하는지 판단한다.

```python
def is_consecutive_numbers_3(numbers: list[int], n: int) -> bool:
    sorted_number = sorted(set(numbers))

    if len(sorted_number) < n:
        return False

    for num in numbers:
        if all([num + i in sorted_number for i in range(n)]):
            return True

    return False
```
