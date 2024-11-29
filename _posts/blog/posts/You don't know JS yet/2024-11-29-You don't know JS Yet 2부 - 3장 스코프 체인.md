---
title: You don't know JS Yet 2부 - 3장 스코프 체인
author: 신성일
date: 2024-11-29 19:00:00 +0900
categories: study, YDKJSY
tags:
  - "#study"
---
스코프 체인 : 스코프와 중첩 스코프 사이에 맺어진 연결. 변수 접근 시 사용할 경로가 스코프 체인을 통해 결정된다. 이때 체인은 변수 탐색 경로가 위 혹은 바깥으로만 향하도록 지시 받는다.


## 3.1 탐색의 진실

우리는 어떤 변수가 어느 스코프에 있는지 상위스코프로 탐색해나간다고 생각하지만, 실제 동작방식은 다르다. 실제로는 컴파일 초기에 이미 어떤 스코프에 있는 변수인지 결정된다. 렉시컬 스코프가 컴파일 초기에 확정되기 때문이다.

따라서 프로그램 실행시 JS 엔진은 변수를 탐색할 필요가 없고, 이미 저장된 정보를 꺼내어 쓰면 되기에 최적화 측면에서 좋다.

하지만 컴파일 중에 스코프가 결정되지 않는 경우가 있다. 만약 현재 파일에서 접근 가능한 렉시컬 스코프에서 참조가 불가능한 변수가 있을때, 런타임에 다른 파일이 해당 변수를 전역에 선언하였을 가능성도 있다. 따라서 접근 가능한 스코프에 원하는 변수가 선언되었는지 여부는 런타임에 완전히 확정된다.

선언하지 않은 변수에 대한 참조는 해당 파일을 컴파일 하는 동안에는 스코프가 결정되지 않는다. 런타임에서 결정되는데, 이러한 과정은 변수당 최대 한번만 발생하며 이후 변경되지 않는다.

이러한 과정을 거친 후에도 스코프가 결정되지 않을 경우 `ReferenceError`가 발생한다


## 3.2 변수 섀도잉

같은 스코프에 이름이 같은 두 개 이상의 변수가 있으면 안된다. 만약 변수들이 속한 스코프가 다르고 이름이 같은 경우 발생하는 것이 `섀도잉`이다. 섀도잉이 발생하면 하위 스코프의 변수가 같은 이름의 상위 스코프의 변수를 가릴수 있다.

### 전역 언섀도잉

(버그, 가독성 저하를 불러일으키는 코드이기에 사용을 권장하지 않는 방법)

전역 스코프의 변수를 가린 스코프에서 가려진 전역 변수를 접근할 수 있는 방법이 있다. 렉시컬 식별자 참조가 아닌 다른 방법을 통하면 가능하다.

전역 스코프에서는 **var로 선언된 변수와 function 키워드로 선언한 함수**는 전역 객체의 프로퍼티를 통해 접근할 수 있다. 전역 객체는 본질적으로 전역 스코프를 객체로 나타낸 것으로 볼 수 있다. (ex : 브라우저에서 window 객체) (완전히 참인 정보는 아님.)

따라서 아래와 같이 가려진 전역 스코프의 변수는 아래처럼 전역 객체를 통해 접근할 수 있다
```js
var studentName = "name";

function printStudent(studentName) {
	console.log(studentName);          // "name2"
	console.log(window.studentName);   // "name"
}

printStudent("name2")
```

전역스코프가 아니면 이러한 방법은 통하지 않는다. (+ var, function으로 선언되지 않았으면)

### 복사와 접근은 다르다


```js
function lookingFor(special) {
	var another = {
		special: special
	};
	
	function keepLooking() {
		var special = 3.14;
		console.log(special)
		console.log(another.special)

		another.special = 3.16;
		console.log(another.special)
	}

	keepLooking();

	console.log(special)
}

lookingFor(1234)
// 3.14
// 1234
// 3.16
// 1234
```

- `keepLooking()` 내부에서 참조한 `another.special`은 매개변수 `special`의 복사본이다. 따라서 `keepLooking()`에서 접근한 `special`은 매개변수 `special`이 아닌 복사된 변수 `special`이다.
- 따라서 `keepLooking()`에서는 매개변수 `special`을 재할당할 방법이 없고, 이는 `lookingFor()`을 호출할때 객체나 배열 등 참조를 넣어도 마찬가지이다. (재할당이 안되고, 객체 프로퍼티를 통해 변경은 가능)

### 금지된 섀도잉

모든 선언 조합이 섀도잉을 만들지않는다. let은 var을 가릴수 있지만 var는 let을 가릴 수 없다.

```js
function something() {
	var special = "자바스크립트"
	{
		let special = 42  // 문제 없음
	}
}

function another() {
	let special = "자바스크립트"
	{
		var special = 42  // SyntaxError 발생 
	}
}
```

- syntaxError가 발생한 이유는 var가 같은 이름을 사용해 let으로 선언한 변수의 경계를 가로지르려고 했기 떄문이다. 

경계 뛰어넘기 금지는 함수 경계를 만났을때는 효과를 발휘하지 못하기에 다음 코드는 정상 동작한다

```js
function another() {
	let special = "자바스크립트"
	function () {
		var special = 42  // 섀도잉 됨
		console.log(special) // 42
	}
}
```

## 3.3 함수 이름 스코프

- 함수 선언문 (function declaration) : 함수가 호이스팅 됨
```js
function askQuestion() {}
```

- 함수 표현식 (function expression) : 함수 자체가 호이스팅 되지 않음
```js
var askQuestion = function ofTheTeacher() {}
```

함수 선언문과 함수 표현식의 두드러지는 차이점은 함수 이름 식별자 작동 방식이다. 다음은 기명 함수 표현식이다. 
```js
var askQuestion = function ofTheTeacher() {
    "use strict";
    
	console.log(ofTheTeacher)
	ofTheTeacher = 42; // TypeError
}

askQuestion(); // function ofTheTeacher() ...
console.log(ofTheTeacher)  // ReferenceError
```

- askQuestion은 외부 스코프이다
- ofTheTeacher는 함수 안에 식별자 그 자체로 선언된다. (외부에서는 접근이 안된다)
- ofTheTeacher는 읽기 전용으로 선언된다 (엄격모드에서는 에러를 발생 시킴)

다만 이는 기명함수표현식일때만이고, 익명 함수 표현식에서는 식별자가 애초에 없다.

## 3.4 화살표 함수

화살표함수는 렉시컬 스코프 관점에서 익명으로 취급된다. 화살표함수는 함수를 참조하는 식별자와 직접 연결되어있지 않는다. 이름을 추론할 수 이지만 기명함수와 똑같이 작동하는 것은 아니다

```js
var askQuestion = () => {}

askQuesion.name; // askQuestion
```

스코프 관점에서 화살표함수는 렉시컬 스코프 측면에서 일반함수와 동일하게 동작한다. 익명이라는 특성 외에는 function으로 선언한 함수와 동일한 렉시컬 스코프 규칙을 적용받는다. 

함수 본문을 감싸는 대괄호가 있든 없든 화살표 함수는 별도의 내부 중첩 스코프를 형성하고, 이 중첩 스코프에 선언된 변수들은 일반 함수의 본문 내에 선언한 변수 스코프와 동일하게 작동한다.