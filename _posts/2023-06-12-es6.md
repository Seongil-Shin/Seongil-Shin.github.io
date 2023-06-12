---
title: es6에서 도입된 문법
author: 신성일
date: 2023-06-12 23:00:00 +0900
categories: [study, javascript]
tags: [javascript]
---



## **let, const 키워드를 통한 변수선언**

기존 자바스크립트에서는 `var` 키워드로만 변수선언이 가능했다. 하지만, `let` `const` 키워드를 추가하여 보다 예측가능한 코드를 작성할 수 있게 됐다.

```js
// es6 이전.
// var는 재선언 가능
var a = "This is string";
var a = 1234;

// ES6 이후
// let은 재선언 불가능
let b = "This is string";
b = 1234;

// const는 재할당 불가능
const c = "This is string";
c = 1234; // <-- 에러 발생
```

`const`는 상수 키워드로 재할당, 재선언이 불가능하다. 변경할 수 없다로 이해하기 쉽지만, `let`과 `var`는 헷갈릴 수 있다. 차이는 다음과 같다.

|          | var            | let                                                          |
| -------- | -------------- | ------------------------------------------------------------ |
| 재선언   | 가능           | 불가능                                                       |
| 재할당   | 가능           | 가능                                                         |
| scope    | function-scope | block-scope                                                  |
| 호이스팅 | 일어남         | 안일어남<br />(정확히는 일어나지만 할당문을 만나기 전까지는 사용불가능) |

<br/>

## 템플릿리터럴

문자열 내에 변수사용을 쉽게 만들어주는 문법으로 다음과 같이 사용한다.

```js
const name = "user-1";
const count = 324;

// es6 이전
var str1 = name + " has pushed button " + count + " times."; 
// es6 이후
const str2 = `${name} has pushed button ${count} times.`;
```

<br/>

## 화살표함수

함수 표현식을 화살표함수로 표현하여 간결하게 함수를 작성할 수 있다.

```js
// es6 이전
function func(a, b) {
  return a + b;
}

// es6 이후
const func = (a, b) => {
  return a + b;
}
// return 생략
const func = (a, b) => a + b;
```

<br/>

## 구조분해할당

객체, 배열의 구조를 분해하여, 새로운 변수에 할당하는 과정이다. 객체/배열에서 값을 꺼낼때 가독성 좋게 가져올 수 있도록 한다.

```js
// es6 이전 (배열)
var arr = [1, 2, 3];
console.log(arr[0], arr[1], arr[2]);
// es6 이후 (배열)
const [first, second, third] = arr;
console.log(first, second, third);

// es6 이전 (객체)
var obj = {
  name : "user-1",
  count: 123
}
console.log(obj.name, obj.count);

// es6 이후 (객체)
const {name, count} = obj;
console.log(name, count);
```

<br/>

## Promise

ES6 이전에 자바스크립트의 비동기처리는 콜백함수로 이루어졌다. 하지만 비동기 함수가 많아지면서 콜백 지옥이 일어나기도 했다.

```js
// es6 이전
function callbackHell(callback) {
  setTimeout(function() {
    ...	// do something
   	setTimeout(function() {
      ... // do something
      setTimeout(function() {
        ... // do something
        callback();
      }, 1000)
    }, 1000)
  }, 1000)
}
```

이러한 콜백 지옥을 개션하기 위한 문법으로 다음과 같이 코드가 변경된다.

```js
function callbackFree(callback) {
  const promise1 =  new Promise((resolve, reject) => {
    setTimeout(() => {
      ... // do something
      resolve();
    }, 1000)
  })
  
  const promise2 = promise1.then(() => {
   return new Promise((resolve, reject) => {
       setTimeout(() => {
        ... // do something
        resolve();
      }, 1000)
   })
  });
  
  promise2.then(() => {
    setTimeout(() => {
	    ... // do something
      callback();
    }, 1000)
  })
}

```

<br/>

## Class

자바스크립트에서 객체 지향 프로그래밍이 가능하도록 `Class` 키워드가 도입되었다. 

```js
class Polygon {
  constructor(height, width) {
    this.name = 'Polygon';
    this.height = height;
    this.width = width;
  }
}

class Square extends Polygon {
  constructor(length) {
    super(length, length);
    this.name = 'Square';
  }
}
```

<br/>

## 모듈 시스템

자바스크립트에서 코드를 분리하여 관리할 수 있게하여 재사용성과 생산성을 높이기위해 모듈 시스템이 도입되었다.

- script 태그 사용 예시

  ```html
  <script type="module" src="lib.mjs"></script>
  ```

- es6에서 사용 에시

  ```js
  import compute from "commons/compute";
  
  const result = compute(1,2);
  
  export result;
  ```

- commonsJs에서 사용 예시
  ```js
  const compute = require("commons/compute");
  
  const result = compute(1,2);
  
  module.exports = { result };
  ```



>require와 import의 차이점
>
>- require/exports
>  - 구형브라우저나, 브라우저 밖에서 사용하는 `CommonJS` 문법
>  - 파일의 어디서나 호출 가능
>
>- import/export
>  - ES6에서 도입된 문법
>  - 파일의 시작부분에서만 실행가능(import 전용 비동기 문법으로 중간에 불러올 수도 있다.)
>  - 필요한 부분만 선택하여 로드 가능. 또한 require보다 성능이 우수함
>
>- 하나의 파일에서 두 키워드를 동시에 사용할 순 없

<br/>

## 출처

- https://hanamon.kr/javascript-es6-%EB%AC%B8%EB%B2%95/