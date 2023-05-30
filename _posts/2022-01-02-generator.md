---
title: generator 문법
author: 신성일
date: 2022-01-02 18:19:26 +0900
categories: [study, javascript]
tags: [javascript]
---

## Generator 문법

- 자바스크립트의 기능이며 Redux-saga의 핵심 기능이다.
- 함수를 특정 구간에 멈춰놓고, 원할때 다시 돌아가게 할 수 있음. 또 반환을 여러번 할수 있다.

### 예시

```javascript
function* generate() {
  yield 1;
  yield 2;
  yield 3;
  return 4;
}
const generator = generate();
```

- 제너레이터 함수를 호출 하면, 제너레이터 객체를 반환한다.
- 이 객체에 있는 메서드인 .next()를 호출하면 yield 한 값을 반환하고 코드의 흐름을 멈춤.

```console
generator.next()		-> {value : 1, done :false}
generator.next()		-> {value : 2, done :false}
generator.next()		-> {value : 3, done :false}
generator.next() 		-> {value : 4, done :true}
generator.next()		-> {value : undefined, done :false}
```

- 아래와 같이 값을 중간에 받을 수도 있다.

```javascript
function* sumGenerator() {
  let a = yield;
  let b = yield;
  yield a + b;
}
```

```console
generator.next()		-> {value : undefined, done :false}
generator.next(1)		-> {value : undefined, done :false}
generator.next(2)		-> {value : undefiend, done :false}
generator.next()		-> {value : 3, done :true}
```

- 아래와 같이 next에 넣은 값을 통해 모니터링이 이루어 질수도 있음

```javascript
function* watchGenerator() {
  while (true) {
    const action = yield;
    if (action.type === "type1") {
      console.log("type 1");
    }
    if (action.type === "type2") {
      console.log("type 2");
    }
  }
}
```

```javascript
generator.next()					-> {value : undefined, done : false}
generator.next({type:'type1'})		-> "type 1"
```
