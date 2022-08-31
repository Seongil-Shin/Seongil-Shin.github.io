---
title: this에 대해서
author: 신성일
date: 2022-02-18 12:10:00 +0900
categories: [study, javascript]
tags: [javascript]
---

## **this에 대해서**

자바스크립트에서 모든 함수는 실행될때마다 함수 내부에 this라는 객체가 추가된다. `arguments`라는 유사배열 객체와 함께 함수 내부로 암묵적으로 전달되는 것이다. 그렇기 때문에 자바스크립트에서의 this는 함수가 호출된 상황에 따라 그 모습을 달리한다.

<br/>

### **상황 1. 객체의 메서드를 호출할때**

객체의 프로퍼티가 함수일 경우 메서드라고 부른다. this는 함수를 실행할 때 함수를 소유하고 있는 객체를 참조한다. 즉, 해당 메서드를 호출한 객체로 바인딩 된다. `A.B`일때, `B`함수 내부에서의 this는 `A`를 가리키는 것이다.

```javascript
var myObject = {
  name: "foo",
  sayName: function () {
    console.log(this);
  },
};
myObject.sayName();
// console> Object {name: "foo", sayName: sayName()}
```

<br/>

### **상황 2. 함수를 호출할때**

특정 객체의 메서드가 아니라 함수를 호출하면, 해당 함수 내부코드에서 사용된 this는 전역 객체에 바인딩 된다. `A.B`라고 생각해보면 A가 전역객체가 되므로, B 함수 내부에서의 this는 전역 객체에 바인딩된다. 즉, A에 해당하는 특정 객체가 없다면 B에서의 this는 전역객체에 바인딩된다는 뜻이다.

```javascript
var value = 100;
var myObj = {
  value: 1,
  func1: function () {
    console.log(`func1's this.value: ${this.value}`);

    var func2 = function () {
      console.log(`func2's this.value ${this.value}`);
    };
    func2();
  },
};
myObj.func1();
// console> func1's this.value: 1
// console> func2's this.value: 100
```

<br/>

### **상황 3. 생성자 함수를 통해 객체를 생성할 때**

new 키워드를 통해 생성자함수를 호출하여 객체를 만들었을때는, new 키워드를 통해 호출된 함수 내부에서의 this는 객체 자신이 된다. 이는 생성자 함수가 동작하는 방식을 통해 이해할 수 있다.

new 연산자를 통해 함수를 생성자로 호출하게 되면, 일단 빈 객체가 생성되고 this가 바인딩된다. 이 객체는 함수를 통해 생성된 객체이며, 자신의 부모인 프로토타입 객체와 연결되어있다. 그리고 return 문이 명시되어있지 않은 경우에는 this로 바인딩 된 새로 생성한 객체가 리턴된다.

```javascript
var Person = function (name) {
  console.log(this);
  this.name = name;
};

var foo = new Person("foo"); // Person
console.log(foo.name); // foo
```

<br/>

### **상황 4. apply, call, bind를 통한 호출**

상황 2에서, `func2`를 호출할때, `func1`에서의 this를 주입하기 위해서는 위 세가지 메소드를 사용할 수 있다.

- bind : 함수를 생성할때 연결함. 매개변수는 하나씩 콤마로 구분하여 넣어줌

```javascript
var value = 100;
var myObj = {
  value: 1,
  func1: function () {
    console.log(`func1's this.value: ${this.value}`);

    var func2 = function (val1, val2) {
      console.log(`func2's this.value ${this.value} and ${val1} and ${val2}`);
    }.bind(this, `param1`, `param2`);
    func2();
  },
};

myObj.func1();
// console> func1's this.value: 1
// console> func2's this.value: 1 and param1 and param2
```

- call : 호출할때 연결함. 매개변수는 하나씩 콤마로 구분하여 넣어줌

```javascript
var value = 100;
var myObj = {
  value: 1,
  func1: function () {
    console.log(`func1's this.value: ${this.value}`);

    var func2 = function (val1, val2) {
      console.log(`func2's this.value ${this.value} and ${val1} and ${val2}`);
    };
    func2.call(this, `param1`, `param2`);
  },
};

myObj.func1();
// console> func1's this.value: 1
// console> func2's this.value: 1 and param1 and param2
```

- apply : 호출할때 연결함. 매개변수는 배열로 넣어줌

```javascript
var value = 100;
var myObj = {
  value: 1,
  func1: function () {
    console.log(`func1's this.value: ${this.value}`);

    var func2 = function (val1, val2) {
      console.log(`func2's this.value ${this.value} and ${val1} and ${val2}`);
    };
    func2.apply(this, [`param1`, `param2`]);
  },
};

myObj.func1();
// console> func1's this.value: 1
// console> func2's this.value: 1 and param1 and param2
```

## **출처**

https://github.com/JaeYeopHan/Interview_Question_for_Beginner/tree/master/JavaScript#this-%EC%97%90-%EB%8C%80%ED%95%B4%EC%84%9C
