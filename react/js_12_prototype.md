# Prototype과 프로토타입 체인

JavaScript 객체는 다른 객체를 프로토타입으로 참조할 수 있다. 속성을 찾을 때 자신의 속성에 없으면
프로토타입 체인을 따라 위쪽 객체에서 찾는다.

```js
const animal = {
  move() {
    return 'move';
  },
};

const dog = Object.create(animal);
dog.name = 'Bori';

console.log(dog.move()); // move
console.log(Object.getPrototypeOf(dog) === animal); // true
```

`__proto__`는 오래된 접근 방식이므로 `Object.getPrototypeOf`와 `Object.create`를 사용한다.
클래스 문법도 내부적으로는 프로토타입 기반 상속 위에서 동작한다.

## 속성 디스크립터와 불변성

속성은 값 외에 변경 가능 여부(`writable`), 열거 여부(`enumerable`), 재정의 가능 여부
(`configurable`)를 가진다.

```js
const user = { name: 'Jin' };
Object.defineProperty(user, 'id', { value: 1, writable: false });
```

`Object.freeze`는 얕게 동결한다. 중첩 객체까지 동결하려면 별도 깊은 동결 로직이 필요하며,
대부분의 애플리케이션에서는 필요한 경로만 불변 업데이트하는 방식이 더 명확하다.
