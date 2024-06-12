# Regex(정규표현식)

## RegExp 객체 기본 코드

```js
const regex = new RegExp();
```

RegExp 생성자가 받는 매개변수에는 필수 매개변수인 정규 표현식과 선택 매개변수가 있다.

### 선택 매개변수

- i: 대소문자 구분하지 않고 일치하는 문자열 검색
- g: 전역적으로 일치하는 문자열 검색하며, 일치하는 모든 문자열 검색한다.
- m: 다중열 문자열에 대해서도 일치하는 문자열 검색

### 함수

- search()

  - 정규식과 일치하는 문자열을 찾아 첫번 째 일치하는 항목의 인덱스를 반환.
  - 항목이 없는 경우 -1

- match()
  - 매개변수를 전달하지 않으면 빈문자열이 있는 Array 반환
  - 일치하는 문자열이 있다면 해당 문자열을 포함한 Array 반환
  - 일치하지 않는다면 null 반환

> 📚 자바스크립트 String 객체에는 정규 표현식과 관련된 두가지 함수가 있다.
>
> - exec(): 문자열 내에 일치하는 문자열을 찾고, 일치하는 첫번째 문자열 반환
> - test(): 문자열 내에 일치하는 문자열을 찾고, true, false 반환

### 정규표현식

| regex    | desc                                |
| -------- | ----------------------------------- |
| ^        | 문자열 시작                         |
| \d       | 모든 숫자                           |
| [abc]    | 괄호 내의 모든 문자                 |
| [\^abc]  | 괄호 내의 문자들을 제외한 모든 문자 |
| [0-9]    | 괄호 내의 모든 숫자를 찾음          |
| [\^0-9]  | 괄호 내의 숫자를 제외한 모든 숫자   |
| (x \| y) | x 또는 y 를 찾음                    |

---

#### 특정 문자 시퀀스를 제외

- ^
  - 문자열 클래스 내부 ^는 문자 클래스를 무효화한다.
  - 즉, 나열된 문자를 제외한 모든 문자와 일치하는 문자를 찾는다.
- \+
  - 앞의 문자나 문자 클래스가 하나 이상 나타나는 것과 일치하는지 검사한다.
  - 즉, 나열된 문자열 a 에서 c 까지의 문자열 시퀀스를 찾는다.

```js
const regex = new RegExp(/[^a-c]+/, 'g');

const a = 'abc';
console.log(a.match(regex)); // null

const b = 'abcd';
console.log(b.match(regex)); // ['d']

const c = 'cba';
console.log(c.match(regex)); // null

const d = 'ca';
console.log(d.match(regex)); // null

const e = 'cad';
console.log(d.match(regex)); // ['d']
```

---

#### 문자열을 n 개씩 잘라서 배열로

```js
const str = 'abc1Addfggg4556b';
const n = 6;

const ret = str.match(new RegExp(`.{1,${n}}`, 'g'));
console.log(ret); // [ 'abc1Ad', 'dfggg4', '556b' ]
```

- .: 개행 문자를 제외한 모든 문자와 일치
- {1,${n}}: 앞의 .(모든 문자)의 1~`n` 발생과 일치하는 수량자를 지정
- g: 전역. 정규식이 첫 번째 일치 이후에 중지되지 않고 문자열에서 모든 일치를 찾도록

---

#### 문자열에서 특정 문자를 특정 문자로 변경

주어진 문자열에서 특정 문자들을 특정 문자들로 변경

```javascript
const obj: { [key: string]: string } = {
  1: 'a',
  2: 'b',
  3: 'c',
};

const str = '12344';
const r = str.replace(/1|2|3/g, (ele) => obj[ele]);

console.log(r); // abc44
```

---

#### 문자열에서 그룹 문자열 제거

아래의 코드는 문자열에서 모음을 제거한다.

```js
function solution(myString: string) {
  const reqex = new RegExp('[aeiou]', 'g');
  return myString.replace(reqex, '');
}
console.log(solution('bus'));
console.log(solution('nice to meet you'));
```

---

#### 문자열에서 숫자 문자만 찾기

```js
function extractNumericStringA(input: string) {
  const regex = /\d+/g;
  const numericStrings = input.match(regex) || [];
  return numericStrings.join('');
}

function extractNumericStringB(input: string) {
  const regex = /\d+/g;
  const result = [...input].reduce(
    (acc, cur: string) => (cur.match(regex) ? (acc += cur) : (acc += '')),
    '',
  );
  return result;
}
```

두 코드는 같은 결과를 반환한다.

- 첫번째 함수

  - input.match(regex)를 통해 정규 표현식을 사용하여 문자열에서 숫자를 찾는다
  - 이 작업은 문자열을 한 번만 반복하므로 문자열의 길이에 비례하여 작동한다.
  - 즉, 문자열의 길이가 N이면 시간 복잡도는 O(N)
  - 그 후, numericStrings.join('')를 사용하여 배열을 문자열로 변환하는데, join 메서드는
    배열의 모든 요소를 연결하기 때문에 시간 복잡도는 배열의 길이에 비례한다.
    이 경우에는 숫자의 개수에 비례한다.
  - 따라서 이 방법의 총 시간 복잡도는 O(N + M)이며, 여기서 N은 입력 문자열의 길이이고, M은 숫자의
    총 개수이다.

- 두번째 함수
  - 이 함수는 문자열을 반복하여 각 문자를 검사한다. 각 문자를 검사할 때마다 cur.match(regex)를
    사용하여 해당 문자가 숫자인지 확인한다.따라서 문자열의 길이에 비례하여 작동하므로 시간 복잡도는
    O(N)이다.
  - result 문자열에 숫자를 추가하는 작업은 각 문자마다 한 번씩 수행되므로 문자열의 길이에 비례한다.
    이 방법의 총 시간 복잡도는 O(N)이다.
