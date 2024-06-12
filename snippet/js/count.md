# 특정 문자 개수 세기

## forEach 활용

```javascript
const char = 'a';
const str = 'aaabbbccc';

let count = 0;
[...str].forEach((ele) => {
  if (ele === char) {
    count++;
  }
});
console.log(count); // 3
```

---

## reduce 함수 활용

```javascript
const char = 'a';
const str = 'aaabbbccc';

const answer = [...str].reduce((acc, cur) => (cur === char ? acc + 1 : acc), 0);
console.log(answer); // 3
```

---

## match 함수와 정규식 활용

```javascript
const char = 'a';
const str = 'aaabbbccc';
const regex = new RegExp(char, 'g');
console.log((str.match(regex) || []).length); // 3
```
