---
title: You don't know JS Yet - 3장 자바스크립트 뿌리 파헤치기
author: 신성일
date: 2024-11-17 19:00:00 +0900
categories: study, YDKJSY
tags:
  - "#study"
---

## 3.1 이터레이션

이터레이터 패턴 : 데이터를 일정 단위로 쪼개고, 이 조각들을 차례로 순회하며 점진적으로 처리하는 표준화된 방법
- 처리할 데이터를 참조하는 데이터 구조인 이터레이터가 정의되어야함
- `next()` 메서드 : 데이터 조각을 차례로 반환하는 메서드
	- 매 호출마다 value, done 프로퍼티를 제공함. 
	- 반복작업이 종료되었을 경우 done 은 true로 세팅됨


이터레이터 소비자 (consumer)
- `next()` 메서드 호출
- `for...of` 반복문 : ES6에서 추가된 문법
- `...` : spread syntax, rest parameter 에 사용됨


이터러블
- 이터레이터 소비 프로토콜은 순회 가능한 값인 이터러블을 소비하는 기술적인 방법이라고 정의 가능. 이터러블에서 이터레이터 인스턴스를 생성하고, 이 인스턴스를 소비해 연산한다.
- ES6 에서 이터러블은 문자열, 배열, 맵, 셋 등이 있음
- 기본적으로 `for...of`로 값을 하나씩 가져와서 소비할수 있으나, JS 내장 인터러블로 조금 다르게 동작할수도 있음
	- 맵은 `for...of` 시 키와 값이 같이 나옴 
		- `for (let [btn, btnName] of buttonNames)`
	- `keys()`, `values()` 로 키 또는 값만 대상으로 소비할 수 있음 
		- `for (btn of buttonNames.keys())`
	- 배열과 `entries()`를 함께 사용하면 인덱스와 값을 순회할 수 있음 
		- `for (let [idx, val] of arr.entries())`
	- 그 외 이터레이션 프로토콜을 준수하는 자료구조를 직접 만들어 `...`, `for...of` 문을 직접만든 자료구조에 사용하게 만들 수 있다


## 3.2 클로저

클로저 : 함수가 정의된 스코프가 아닌 다른 스코프에서 함수가 실행되더라도, 스코프 밖에 있는 변수를 기억하고 이 외부 변수에 계속 접근할 수 있는 경우
- 함수의 타코난 특징이다
- 클로저를 직접 보고 싶으면 함수를 해당 함수가 정의된 스코프가 아닌 다른 스코프에서 실행해야한다

```js
function greeting(msg) {
	return function who(name) {
		console.log(`${name} 님, ${msg}!`)
	}
}

var hello = greeting("안녕하세요")
var howdy = greeting("잘 지내시나요")
hello("카일")
howdy("호진")
```

- 스코프는 인스턴스마다 생성된다 -> 위 예시에서는 hello, howdy에 각각의 클로저가 있음
- 클로저는 변수 자체와 직접적인 관계를 맺어 변수가 업데이트 되는 것을 관찰하고 최신 값을 가져온다.

클로저의 외부 스코프는 항상 함수여야하는 것은 아니다. 보통 내부 함수에서 하나이상의 외부 스코프 변수를 접근하려할 때 클로저를 볼 수 있다

```js
for (let [idx, btn] of buttons.entries()) {
	btn.addEventListener("click", () => console.log(`${idx}`번째 ))
}
```

## 3.3 this 키워드

this는 현재 함수의 실행 컨텍스트를 참조하는 키워드이다. 스코프와 달리 실행컨텍스트는 동적이며, 함수를 정의한 위치나 호출위치에 관계없이 호출 방식에 따라 호출할 때마다 결정된다. 

엄격모드에서 실행하지 않으면 컨텍스트를 지정하지 않고 함수를 실행할 경우, 기본 컨텍스트는 전역 객체 (브라우저에서는 window)가 된다. 

다음과 같이 객체의 프로퍼티로 함수를 정의하면 해당 함수의 `this`는 해당 객체가 된다.
```js
var homework = {
	topic: "JS",
	assignment: function () {
		console.log(this.topic)
	}
}
```

또는 다음처럼 `.call` 메서드와 함께 사용하여 컨텍스트를 직접 주입해줄수도 있다

```js
var homework = {
	topic: "JS"
}

function assignment() {
   console.log(this.topic)
}
assignment.call(homework);
```


## 3.4 프로토타입

프로토타입은 두 객체를 연결하는 연결장치로, 객체가 생성될 때 만들어지고 이 연결장치를 통해 새롭게 생성된 객체는 기존에 존재하는 다른 객체에 연결된다. 프로토타입을 통해 연결된 일련의 객체는 `프로토타입 체인`이라고 한다.

```js
var homework = {
	topic: "JS"
}
```

- `homework` 객체에는 `topic`이라는 프로퍼티만 존재하지만, `Object.prototype` 객체를 연결하므로 해당 객체 내에 있는 `toString()` 등의 메서드를 사용할수가 있다.


### 객체 연결장치

객체 프로토타입 연결 장치를 직접 정의하고 싶을 때는 `Object.create()`를 사용해 객체를 만들면 된다.

```js
var homework = {
	topic: "JS"
}
var otherHomework = Object.create(homework)
otherHomework.topic; // "JS"
```
- 첫번째 인수에는 새롭게 생성할 객체를 어떤 객체와 연결할지 명시한다. 그러면 이 객체와 연결된 새로운 객체가 반환된다
- 위 관계에서는 `otherHomework` -> `homework` -> `Object.prototype` 으로 프로토타입이 연결되어있다. (`otherHomework.prototype`이 `homework`)
- 이 상태에서 객체에 새로운 프로퍼티를 생성하면, 프로토타입에 관계 없이 바로 그 객체에 프로퍼티가 추가된다
```js
homework.topic; // "JS"
otherHomework.topic; // "JS"

otherHomework.topic = "수학" // otherHomework 에 `topic` 프로퍼티가 생성됨
otherHomework.topic; // "수학"

homework.topic; // "JS"
```

### this 다시 보기

함수를 호출할 때 프로토타입을 통해 발생하는 위임과 this를 함께 다루면 진가가 드러난다

```js
var homework = {
	study() {
		console.log(this.topic)
	}
}

var jsHomework = Object.create(homework)
jsHomework.topic = "JS"
jsHomework.study(); // JS

var mathHomework = Object.create(homework)
mathHomework.topic = "수학"
mathHomework.study(); // 수학
```

- `homework`가 각 객체와 연결되면서, 각 객체는 `study()` 메서드를 사용할 수 있음.
- 각 객체에서 `topic`을 설정해주어 각자의 실행컨텍스트에 `topic`을 추가함.
- `this`가 `homework` 였다면 위와 같은 코드는 불가능하였을 것이다. 상당수의 다른 언어들에서는 실제로 그렇다. 하지만 JS에서는 실행컨텍스트가 동적으로 결정되고, 이는 프로토타입을 통한 위임을 가능하게 만드는 중요한 요소이다.