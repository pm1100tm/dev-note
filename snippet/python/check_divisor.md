# 약수 판별

```python
def count_divisors(n):
    if n <= 0:
        raise ValueError("n must be a positive integer")

    count = 0
    sqrt_n = int(n ** 0.5)

    for i in range(1, sqrt_n + 1):
        if n % i == 0:
            count += 1
            if i != n // i:
                count += 1

    return count
```
