# Prime Factorization(소인수분해)

## 주어진 n이라는 정수 이하의 수에서 모든 소수 찾기

```js
function solution(n: number) {
  const factors = [];
  let divisor = 2;
  let _n = n;
  while (_n > 1) {
    while (_n % divisor === 0) {
      factors.push(divisor);
      _n /= divisor;
    }
    divisor++;
  }
}
```

### 개선된 snippet

```js
function primeFactors(n: number) {
  // Print the number of 2s that divide n
  while (n % 2 == 0) {
    console.log(2);
    n = n / 2;
  }

  // n must be odd at this point. So we can skip one element (Note i = i +2)
  for (var i = 3; i * i <= n; i = i + 2) {
    // While i divides n, print i and divide n
    while (n % i == 0) {
      console.log(i);
      n = n / i;
    }
  }
  // This condition is to handle the case when n is a prime number
  // greater than 2
  if (n > 2) {
    console.log(n);
  }
}
primeFactors(10); // prints '5' and '2'
```
