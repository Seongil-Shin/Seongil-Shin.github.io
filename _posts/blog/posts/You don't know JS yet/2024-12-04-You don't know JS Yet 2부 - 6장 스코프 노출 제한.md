---
title: You don't know JS Yet 2부 - 6장 스코프 노출 제한
author: 신성일
date: 2024-12-02 19:00:00 +0900
categories: study, YDKJSY
tags:
  - "#study"
---
어떻게, 왜 함수와 블록을 사용해 프로그램 내 변수를 다양한 스코프로 구성해야하는지

## 6.1 최소 노출의 원칙 (POLE)

정보보안분야의 최소 권한의 원칙(POLP)처럼, 변수를 최소한의 블록에만 노출해야한다는 최소 노출의 원칙(POLE)을 적용할 수 있다.

만약 모든 변수를 전역 스코프에 등록하게 되면 아래와 같은 문제가 발생한다.
- 이름 충돌 
- 예기치 않은 동작 : 어떤 변수가 쓰이면 안되는 곳에서 쓰여져 의도치 않게 값을 변경하거나 사용하게 될 수 있음. 개인 정보를 담고 있는 변수나 함수가 제한을 뚫고 사용될 수도 있음.
- 의도하지 않은 종속성 : 변수/함수가 의도하지 않은 곳에서 사용되면 그 곳에서의 의존성이 생김. 추후 유지보수 시 그곳도 신경을 써야함

따라서 아래와 같이 가능한 좁은 범위로 스코프를 제한 시키는 것이 좋음

```js
function diff(x, y) {
	if(x > y) {
		let tmp = x;   // var 대신 let을 사용해 if문 안에서만 쓰이도록 제한함
		x = y;
		y = tmp;
	}
	return y - x;
}
```

## 6.2 일반(함수) 스코프에 숨기기

var 나 함수 선언을 숨기기 위해서는 함수 스코프로 감싸는 방법이 있다.
- 숨기기 전 : cache라는 변수를 어디에서든 접근할 수 있음
```js
var cache = {}
function factorial(x) {
	if(x < 2) return 1;
	if(!(x in cache)) {
		cache[x] = x * factorial(x - 1)
	}
	return cache[x];
}
```

- 숨긴 후 : 선언된 함수 스코프 안에서만 접근 가능함
```js
function hideTheCache() {
	var cache = {}
	return factorial;
	
	function factorial(x) {
		if(x < 2) return 1;
		if(!(x in cache)) {
			cache[x] = x * factorial(x - 1)
		}
		return cache[x];
	}
}
```

함수 표현식으로 감싸서 `hideTheCache`라는 이름도 재사용되도록 할 수 있다.

```js
var factorial = (function hideTheCache() {
	var cache = {}
	return factorial;
	
	function factorial(x) {
		if(x < 2) return 1;
		if(!(x in cache)) {
			cache[x] = x * factorial(x - 1)
		}
		return cache[x];
	}
})()
```

### 함수 표현식 즉시 호출하기

IIFE : 즉시 실행 함수 표현식 
- function 표현식 옆에 `()`을 사용하여 표현된 함수를 바로 실행하는 방법
- `hideTheCache`와 같이 이름을 붙일 수 있고, 익명으로 지정할수도 있다 (익명이 훨씬 흔하다)
- function 표현식을 단독으로 쓸 수도 있고, 무언가를 반환하여 다른 구문의 일부가 될 수 있지만, 단독으로 쓰일때는 function 표현식을 `()`로 감싸야한다. (일관성을 위해 항상 감싸는 것도 좋은 방법이다)

IIFE는 온전한 함수이기에, IIFE를 사용하면 함수 경계가 변경되는데 이는 문이나 구조의 동작을 바꿀 수 있다.
- 화살표함수가 아닌 일반 함수 형태로 IIFE를 사용하면 this의 바인딩이 바꾸니다.
- break, continue는 함수 경계를 넘지 않기에 제어문 안에서 IIFE를 사용하여 IIFE에서 break, continue를 사용한다고 IIFE 밖의 제어문을 통제할 수 있지 않다.


## 6.3 블록으로 스코프 지정

블록은 let, const 같은 블록 스코프 선언을 포함해야할 필요가 있을때만 스코프로 작용한다. 
```js
{
	...
	// 아직 스코프 생성이 필요하지 않음
	...

	// 이제 스코프가 필요하다고 인지함
	let name = "123";

	for(let i = 0; i<5; i++) {
		// let i를 위해 스코프를 생성함

		if(i % 2 === 0) {
			// 여기는 스코프가 아니라 블록일뿐이다.
		}
	}
	
}
```

이와 같이 모든 중괄호 쌍이 블록스코프를 생성하는 것은 아니다
- 객체 리터럴 {}, class 정의 시 {}, switch 문의 {} 로는 블록이나 스코프를 정의할 수 없다
- function의 본문을 감싸는 {}는 블록이 아니라 함수스코프를 생성한다.

블록 스코프를 지원하는 언어에서 명시적으로 블록스코프를 만드는 것은 POLE을 위해 일반적인 패턴이다. 기본 마인드셋으로 POLE을 따르면 의도치않은 버그를 줄이고, TDZ 위험을 최소화할 수 있다.

```js
if(somethingistrue) {
	// ...
	
	{
		let msg = "어떤 메세지"
		console.log(msg);
	}
	
	// ...
}
```

### var와 let

- 함수 전체에 걸쳐 필요한 변수는 의도에따라 명확하게 var로 선언해야한다. 
- var을 블록 내에서 선언할수도 있지만, 예외적인 상황을 제외하면 함수 전체에 필요한 변수는 함수 최상위 스코프에서 var로 선언하는 것이 좋다.

var와 let은 함수/블록 스코프라는 목적에 맞게 사용하는 것이 좋다는게 저자의 의견

### let의 위치

최상위 함수 스코프에서만 var를 사용하라는 조언을 따르면 다른 선언문에서는 let을 사용해야만 한다. 구체적으로 어디에 let을 사용해야하는지는 스스로 '이 변수를 최소한으로 노출시키면서 요구조건을 만족하는 위치는 어디지'하고 질문해보는 것이다.

처음에는 블록스코프에 선언하였다가 나중에 함수 스코프에 선언해야한다고 알게되면 그때 선언 위치와 키워드를 바꾸면 된다.


### catch와 스코프

catch 절에서 선언된 변수는 catch 절 밖에서 사용할 수 없다는 규칙이 있다.

```js
try {
   ...
} catch(e) {
	var outerVar = "123";
	let onlyHere = true;
}

console.log(outerVar); // "123"
console.log(e); // ReferenceError: 'e' is not defined
```

- catch 절로 선언한 e 변수는 해당 블록으로 블록 스코프가 지정된다. 
- ES2019 부터는 catch 절 선언시 매개변수를 선언하지 않아도 된다. 이때는 블록 스코프를 형성하지 않고 블록으로 처리된다. 이 패턴을 사용하면 불필요한 스코프를 줄여 약간의 성능 향상을 꾀할 수 있다

```js
try {
	...
} catch {
	console.log("some err");
}
```


## 6.4 블록 내 함수 선언 (FiB)

블록 내 함수를 선언하는 패턴은 환경마다 다르게 동작한다.

```js
if(false) {
	function ask() {
		console.log("run");
	}
}

ask();
```

- JS 명세서 : ask 는 블록 스코프 안이므로 외부/전역에서는 사용할 수 없어 ReferenceError 예외 발생
- 대부분의 브라우저 : ask 식별자가 존재하지만 if 구문이 실행되지 않아 정의가 되지 않았으므로 실행가능한 상태가 아니기에 TypeError 발생

이는 브라우저에서 과거에 사용하던 패턴을 지원하기 위해 만든 예외이다. 아래와 같은 패턴이 예시이다.

```js
if (typeof Array.isArray != "undefined") {
	function isArray(a) {
		return Array.isArray(a);
	}
} else {
	function isArray(a) {
		return Object.prototype.toString.call(a) == "[object Array]";
	}
}
```

하지만 이러한 FiB 패턴은 환경에따라 동작이 다르기에 사용하지 않는 것이 좋다. 함수 선언은 항당 함수의 최상위 스코프에서 하라.

또한 성능이 약간 떨어지더라도 아래와 같은 패턴을 사용하는 것이 좋다.

```js
function isArray(a) {
	if (typeof Array.isArray != "undefined") {
			return Array.isArray(a);
	} else {
			return Object.prototype.toString.call(a) == "[object Array]";
	}
}
```

아니면 아래와 같이 함수 표현식을 블록안에 배치할 수 있다.

```js
var isArray = function isArray(a) {
	return Array.isArray(a);
}

if (typeof Array.isArray == "undefined") {
	isArray = function isArray(a) {
		return Object.prototype.toString.call(a) == "[object Array]";
	}
}
```

