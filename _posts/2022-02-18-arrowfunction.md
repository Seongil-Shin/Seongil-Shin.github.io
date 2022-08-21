---
title: Arrow function
author: 신성일
date: 2022-02-18 12:50:00 +0900
categories: [cs, javascript]
tags: [javascript]
---

## **화살표함수**

기존의 function 표현방식보다 간결하게 함수를 표현할 수 있다. 화살표함수는 항상 익명이며, 자신의 this, arguments, super, new.target을 바인딩하지 않는다. 따라서 생성자로는 사용할 수 없다.

- 화살표함수 도입 영향 : 짧은 함수, 상위 스코프 this

<br/>

### **짧은 함수**

```javascript
var materials = ["Hydrogen", "Helium", "Lithium", "Beryllium"];

materials.map(function (material) {
  return material.length;
}); // [8, 6, 7, 9]

materials.map((material) => {
  return material.length;
}); // [8, 6, 7, 9]

materials.map(({ length }) => length); // [8, 6, 7, 9]
```

### **상위스코프 this**

```javascript
function Person() {
  this.age = 0;

  setInterval(() => {
    this.age++; // |this|는 person 객체를 참조
  }, 1000);
}

var p = new Person();
```

일반 함수에서 this는 전역객체를 this로 정의한다. 하지만 화살표 함수 this는 상위 스코프의 this와 동일한 값을 갖는다. 따라서 위 예제에서 setInterval로 전달된 this는 Person의 this를 가리키며, Person 객체의 age에 접근한다.

<br/>

## **화살표함수를 사용하면 안되는 순간**

**메소드**

```js
const cat = {
  name: 'meow';
  callName: () => console.log(this.name);
}

cat.callName();	// undefined
```

이 경우 화살표함수의 상위 스코프는 전역 this가 된다. 따라서 메소드로 화살표함수를 사용하면 안된다.

메소드의 경우, 호출한 객체의 this로 바인딩되니 일반 함수를 사용하자

## 생성자

```js
const Foo = () => {};
const foo = new Foo()	// TypeError: Foo is not a constructor
```

화살표함수를 생성자로 사용할 수는 없다.

**addEventListener()의 콜백함수**

```js
const button = document.getElementById('myButton');

button.addEventListener('click', () => {
  console.log(this);	// Window
  this.innerHTML = 'clicked';
});

button.addEventListener('click', function() {
   console.log(this);	// button 엘리먼트
   this.innerHTML = 'clicked';
});
```

addEventListener의 콜백함수에서 this 는 이벤트리스너가 호출된 엘리먼트로 바인딩된다.

하지만 화살표함수를 사용할 경우 상위스코프로 바인딩되므로, 전역 this가 사용되게 된다.

<br/>

<br/>

## **출처**

https://github.com/JaeYeopHan/Interview_Question_for_Beginner/tree/master/JavaScript#promise

https://velog.io/@padoling/JavaScript-%ED%99%94%EC%82%B4%ED%91%9C-%ED%95%A8%EC%88%98%EC%99%80-this-%EB%B0%94%EC%9D%B8%EB%94%A9
