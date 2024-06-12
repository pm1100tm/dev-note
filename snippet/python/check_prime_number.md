# 소수 판별

```python
import math

def is_prime(n):
    # 1 이하의 수는 소수가 아님
    if n <= 1:
        return False

    # 2와 3은 소수임
    if n <= 3:
        return True

    # 2와 3으로 나누어 떨어지는 경우 소수가 아님
    if n % 2 == 0 or n % 3 == 0:
        return False

    # 5부터 제곱근 n까지 6k ± 1 형태의 수로 나누어 떨어지는지 확인
    i = 5
    while i * i <= n:
        if n % i == 0 or n % (i + 2) == 0:
            return False
        i += 6

    return True
```
