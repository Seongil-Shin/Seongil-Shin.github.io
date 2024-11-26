---
title: You don't know JS Yet - 1권 부록
author: 신성일
date: 2024-11-24 19:00:00 +0900
categories: study, YDKJSY
tags:
  - "#study"
---
## Appendix

### A.1 값 vs 참조

다른 언어에서는 값 자체를 할당, 전달할지 값의 참조를 할당, 전달할지 선택할 수 있게 하는 경우가 많다. 하지만 JS에서는 이 과정이 오로지 값의 타입으로만 결정된다.

- 원시타입 : 값이 복사됨
- 객체 (객체, 배열, 함수 등) : 참조로 처리됨


이러한 작동방식을 변경할 수 있는 방법은 없다

### A.2 다양한 형태의 함수

다음과 같은 함수 표현식은 익명 함수 표현식이라고 한다.
```js
var someFunction = function() {
}

console.log(someFunction.name) // "someFunction"
```
- JS는 익명함수라 하더라도 자체적으로 이름을 추론한다. 위 예시처럼 `.name` 에 접근하면 추론된 이름을 볼 수 있다. 이를 통해 오류 발생 시 스택 트레이스에서 함수 이름을 통해 오류 위치를 찾을 수 있다
- 하지만 JS의 이름 추론은 `=`을 통해 할당한 경우에만 발생하고, 인수를 통해 전달된 익명함수는 추론이 발생하지 않는다. 이때는 name은 빈문자열이 되고, 오류 발생 시 (anonymous function)이라고 뜬다.
- 또한 추론된 이름은 어디까지나 메타데이터일뿐이며 실제 해당 함수를 참조하는 식별자가 아니다. 따라서 익명 함수는 본문 안에서 자신을 참조할 수 있는 식별자를 갖지 못한다. 이는 재귀를 사용하거나, 등록된 이벤트를 해제할때 문제가 된다.

아래는 기명 함수 표현식이다
```js
var someFunction = function anyFunction() {
}

console.log(someFunction.name) // "anyFunction"
```
- 컴파일 중에 식별자 `anyFunction`과 함수 표현식에 직접적인 연관관계가 생긴다. `someFunction`과의 연관관계는 런타임까지 발생하지 않는다.

대부분 익명함수표현식을 거부감없이 사용한다. 더 짧기 때문이다. 하지만 함수에 의미있는 이름을 붙이는 것이 가독성이 훨씬 더 좋아진다.

함수 선언방식에는 다른 함수 선언 방식도 있다.

### A.3 강제 조건부 비교

조건에 맞는지 아닌지를 판단하기위해 강제 변환이 선행되어야하는 조건부 표현식에 대한 절

if, 삼항연산자, while/for의 반복문 조건절은 암시적으로 값을 비교한다. 여기에는 타입이 같은지 확인하는 엄격비교가 있고, 강제로 타입을 전환해 비교하는 경우도 있다. 강제 조건부 비교는 이 둘다를 따른다.

만약 아래와 같은 코드가 있으면

```js
var x = "hi";

if(x) {}

if(x == true){}
```

우리가 생각해야할 멘탈모델은 다음과 같다

```js
var x = "hi"

if(Boolean(x) == true) {}

if(Boolean(x) == true)
```

중요한 것은 비교하기 전에 강제 변환이 일어나고, x 는 타입에 상관없이 불리언 값이 된다.


### A.4 프로토타입 클래스

3장에서 다룬 프로토타입과 프로토타입 체인을 사용해 객체를 연결하는 방법을, 프로토타입 클래스라고 부른다. ES6에서 클래스 시스템이 등장하기 전까지 프로토타입 클래스는 객체를 연결하는 역할을 했다.

```js
var Classromm = {
	welcome() {
		console.log("hi")
	}
}

var mathClass = Object.create(Classroom);
mathClass.welcome() // "hi"
```

프로토타입 클래스 패턴에서는 이러한 방식을 상속이라고 부른다. 이러한 상속은 다음과 같이 정의할 수도 있다

```js
function Classroom() {}

Classroom.prototype.welcome = function hello() {
	console.log("hello");
}

var mathClass = new Classroom();
mathClass.welcome() // "hello"
```

모든 함수는 기본적으로 prototype 이라는 프로퍼티를 통해 빈 객체를 참조한다. 이 프로퍼티는 함수의 프로토타입(프로토타입을 통해 함수와 연결된 객체)과는 다르다. 이 프로퍼티 prototype은 new를 사용해 함수를 호출해 객체를 만들었을때 새롭게 만든 객체의 프로토타입을 설정할 수 있게 한다.

하지만 이러한 프로토타입 클래스 패턴보다는 ES6의 클래스 매커니즘이 클래스 지향 디자인 패턴에서는 훨씬 더 잘맞는다. (근본에는 동일한 프로토타입 연결장치로 설정되어있음.)

```js
class Classroom {
	constructor() {
	}

	welcome() {
		console.log("hello")
	}
}

var mathClass = new Classroom();
mathClass.welcome() // "hello"
```
