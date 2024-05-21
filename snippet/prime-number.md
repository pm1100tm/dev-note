# Prime Number(소수 판단)

## n이 소수(prime number)인지 판별

- 1보다 작거나 같은 수는 소수가 아니다.
- 1보다 큰 수 n에 대해서 2부터 n-1 까지 나누어지는지 확인한다.

```js
function isPrime(n: number) {
  if (n <= 1) {
    return false;
  }

  // 2부터 n-1까지의 수로 나눈다.
  for (let i = 2; i < n; i += 1) {
    if (n % i === 0) {
      return false;
    }
  }

  return true;
}
```

### 코드 개선 1

위의 코드는 성능적으로 개선할 부분이 있다.

- 1보다 작거나 같은 수는 소수가 아니다.
- 나누는 범위를 2부터 √n까지로 줄인다.

```js
function isPrime(n: number) {
  if (n <= 1) {
    return false;
  }

  // 2부터 n의 제곱근까지의 수로 나눈다.
  for (let i = 2; i <= Math.sqrt(n); i++) {
    if (n % i === 0) {
      return false;
    }
  }

  return true;
}
```

### 코드 개선 2

아래와 같이 나열된 소수 배열에서 규칙을 찾을 수 있다.

```plain
2,3,5,7,11,13,17,19,23,29,31,37,41,43,47,53...
```

- 0과 1은 소수가 아니다.
- 2와 3은 소수이다.
- 2또는 3의 배수는 제외한다.
- 2와 3을 제외한 모든 소수는 6K ± 1 의 규칙을 갖는다.
- 반복문을 n의 제곱근까지만 확인한다.

```js
function isPrime(n: number) {
  if (n <= 1) return false; // 0 and 1 are not prime numbers
  if (n <= 3) return true; // 2 and 3 are prime numbers

  // Check if n is divisible by 2 or 3
  if (n % 2 === 0 || n % 3 === 0) return false;

  // Check divisibility by numbers of the form 6k ± 1 up to √n
  for (let i = 5; i * i <= n; i += 6) {
    if (n % i === 0 || n % (i + 2) === 0) {
      return false;
    }
  }

  return true;
}
```
