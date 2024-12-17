---
title: You don't know JS Yet 2부 - Appendix
author: 신성일
date: 2024-12-15 19:00:00 +0900
categories: study, YDKJSY
tags:
  - "#study"
---
## A.1 암시적 스코프

스코프는 가끔 명확하지 않은 위치에 생성된다. 일반적으론 이러한 암시적 스코프가 작동에 영향을 주진않는데, 알고 있으면 유용하다.
- 매개변수 스코프
- 함수명 스코프

### 매개변수 스코프

함수 매개변수는 함수 스코프에서 지역으로 선언한 변수와 동일하다고 하였지만, 항상 그렇지는 않다.

아래와 같이 단순한 매개변수 형태는 함수 스코프와 매개변수 스코프가 같다.

```js
function getStudentName(studentId) {
}
```

하지만 기본값이 있는 매개변수, ...을 사용하는 나머지 매개변수, 비구조화 매개변수의 경우에는 매개변수 자체적으로 스코프를 형성하고, 함수 스코프는 그 아래 형성된다. (**단순하지 않은 매개변수는 자체 스코프를 형성한다**)

```js
function getStudentName(/* 파란색 버블 (스코프) */studentId = 0) {
 // 초록색 버블 (스코프)
}
```

그 이유는 단순하지 않은 매개변수 형태에서는 다양한 예외 케이스를 효과적으로 다루기 위해 자체 스코프를 형성한다.

- 왼쪽에서 오른쪽으로 연산이 수행되면서 `maxId`는 초기화되어있지 않으므로 TDZ 에러를 발생시킴.
```js
// TDZ 에러 발생
function getStudentName(studentId = maxId, maxId) {
}

// 문제 없음
function getStudentName(maxId, studentId = maxId) {
}
```

- 기본 매개변수에 함수 표현식이 있는 경우 매개변수 자체에 대한 클로저가 생길 수 있어 복잡성이 더 증가한다.

```js
// id를 재할당하여 5가 출력됨
function getStudentName(id, defaultId = () => id) {
	id = 5;
	console.log(defaultId())
}
getStudentName(3) // 5


// var id가 매개변수 스코프 내의 id를 가려서 defaultId의 클로저는 매개변수 스코프를 가리키고 있기에 3이 출력됨 
function getStudentName2(id, defaultId = () => id) {
	var id = 5;
	console.log(defaultId())
}
getStudentName2(3) // 3
```

- 아래와 같이 과거 코드와의 호환 문제로 상황이 더 복잡해진 경우가 있다.
```js
function getStudentName(id, defaultId = () => id) {
	var id;   // 호환 문제로 매개변수와 깉은 이름의 지역변수는 undefined로 초기화하지 않고, 매개변수 id를 매개변수 값으로 초기화함.
	console.log(id, defaultId())  

	id = 5;  // 지역변수를 5로 재할당하여 매개변수와 다른 스코프에 있음을 확인할 수 있음.
	console.log(id, defaultId())
}
getStudentName(3)
// 3, 3
// 5, 3
```

따라서 이러한 차이에 영향을 받지 않으려면 다음 사항을 유념해야한다
- 지역 변수로 매개변수를 섀도잉하지마라
- 기본 매개변수에서는 다른 매개변수를 참조하지마라

### 함수명 스코프

함수이름 스코프에서는 함수 자체의 스코프에 함수 표현식의 이름이 추가된다고 설명하였다. 
```js
var askQuestion = function ofTheTeacher() {
}
```

이때 ofTheTeacher가 askQuestion이 선언된 외부 스코프에 추가되지 않지만, 단순히 함수 내부 스코프에 추가되는 것도 아니다. 이 경우도 암시 스코프의 예외 케이스를 만든다

```js
var askQuestion = function ofTheTeacher() {
   let ofTheTeacher = "some name" // 중복 선언 오류를 발생하지 않는다
}
```

let은 재선언이 불가능하지만 함수명인 `ofTheTeacher`와 다른 스코프에 있기에 섀도잉으로 처리된다. 실무적으로 이러한 경우가 중요할때가 거의 없지만, 이슈를 피하기 위해 함수 이름과 동일한 이름의 변수를 함수 본문 내부에서 let으로 선언하는 것은 피하라.


## A.2 익명 함수 vs 기명 함수

요약
- 이름 추론은 불완전하다
- 렉시컬 이름을 사용하면 자기 참조가 가능하다
- 이름은 설명을 제공하기에 유용하다
- 화살표함수에는 렉시컬 이름이 없다
- IIFE에도 이름이 필요하다

### A.2.1 명시적 혹은 추론된 이름

기명함수를 사용하여 함수의 이름을 명시적으로 남기면 스택트레이스에서도 함수 이름을 확인할 수 있다. 
이와 별개로 익명함수를 사용하더라도 이름이 추론되는 경우가 있다

```js
var notNamed = function() {}
var config = {
	cb: funcion() {}
}

notNamed.name; // notNamed
confib.cb.name; // cb
```

이와같이 JS는 이름을 추론하기에 추론된 이름을 사용해도 되지만 모든 문제를 해결할 수는 없다

### A.2.2 이름이 없다면?

이름 추론이 정상적으로 동작하지 않는 경우들이 있기에, 이름 추론에 의존하는 것은 불완전하다

- 콜백 인자로 전달 시 이름 추론 안됨
```js
function ajax(url, cb) {
	console.log(cb.name)
}

ajax("some.url", function(){})

// ""
```

- 함수 표현식을 단순 할당이 아닌 다른 형태로 하는 경우에는 대부분 추론에 실패함. 
```js
var config = {}
config.cb = function(){}
config.cb.name; // ""

var [noName] = [function(){}]
noName.name; // ""
```

### A.2.3 나는 누구일까요?

렉시컬 이름 식별자가 없으면 함수는 내부적으로 자기 자신을 참조할 방법이 없다. 자기 참조는 재귀나 이벤트 처리하는 작업에서 매우 중요하다

### A.2.4 이름은 설명입니다

함수에 이름을 넣지 않으면 코드를 이해하기가 어렵다

### A.2.5 화살표 함수

화살표함수는 이름 추론을 통해 이름이 붙더라도 항상 익명이다. 따라서 사용을 지양하는 것이 좋다.

화살표함수에는 목적이 있지만, 이는 타이핑 횟수를 줄이는 것이 아닌 화살표 함수 렉시컬 this 동작을 이해해야 제대로 쓸 수 있다.

간단히 말하면 화살표 함수의 this는 렉시컬 스코프에 의해 결정된다. 화살표 함수 자체는 this를 가지지 않으며 화살펴 함수 안에 this를 사용하면 다른 변수 참조처럼 작동한다. 즉 화살표 함수는 this를 다른 렉시컬 변수처럼 취급한다. 이러한 동작을 사용해야하는 경우는 다음과 같다
- `var self = this` 와 같은 기법을 사용하는 경우
- 내부 함수 표현식에 `.bind(this)`를 호출하는 것 대신에 사용하기 위해

### A.2.6 IIFE

IIFE 에도 이름을 붙이는 것이 좋다.

이와 별개로 IIFE는 보통 함수 표현식을 `()`로 감싸서 정의하는데, 이는 JS 파서가 function 키워드를 만났을때 함수 선언문으로 해석하지 않게하기 위함이다. 이를 위해서는 다음과 같은 방법도 활용가능하며 취향껏 선택할 수 있다
- `()`로 감싸기
- `!`, `+`, `~`, `void` 등의 단항 연산자를 function 앞에 놓아 표현식으로 만들기

```js
(function thisIsIIFE() {})()
!function thisIsIIFE() {}()
+function thisIsIIFE() {}()
~function thisIsIIFE() {}()
void function thisIsIIFE() {}()
```


## A.3 호이스팅 : 함수와 변수

함수, 변수 호이스팅이 왜 유용하고 배워야하는지

### A.3.1 함수 호이스팅

함수 선언문은 컴파일 단계에서 호이스팅 되기에 함수 식별자는 스코프 시작 부분에 함수 참조로 자동 초기화된다. 

이를 사용하면 함수 실행은 코드 위쪽에서하고 선언은 아래쪽에 배치할 수 있는데, 이러한 배치는 복잡한 함수 내부를 아래로 숨길 수 있기에 함수 세부 사항을 굳이 파악하지 않고도 코드의 작동을 파악하기 편하다

```js
getStudents()


function getStudents() {
	var whatever = doSomething();

	return whatever;

	function doSomething() {
		// ...
	}
}
```

### A.3.2 변수 호이스팅

함수 호이스팅과 다르게 변수 호이스팅은 가독성에 도움을 주지 않는다. 따라서 유용한 경우는 적은데 저자의 생각으로는 CommonJS에서 비공개 변수를 하단에 배치하는 경우에는 도움이 된다고 한다.

```js
var publicAPI = Object.assign(module.exports, {
	getStudents
})

// 비공개 구현
var cache = {}
var otherData = [];

function getStudents() {
	// ...
}
```

## A.4 var에 대한 변론

변수 호이스팅과 관련하여 JS 개발 중 발생하는 여러 이슈의 원인은 주로 var와 관련된다. 여기서 살펴볼 것은 다음과 같다
- var에 문제가 있는 것이 아니다
- let은 우리들의 친구이다
- const의 유용성은 제한적이다
- var와 let을 사용하는 최선의 방법은 둘의 장점을 모두 취하는 것이다


### A.4.1 var를 버리지 마세요.

var 에는 문제가 없고 var 만의 장점이 있다. 따라서 무조건 쓰지말자고 하는 것은 장점을 버리는 일이다. 

### A.4.2 혼란스러운 const

const 를 사용하면 얻을 수 있는 약간의 이점은 있지만, const를 사용하면서 겪는 혼란보다는 크지 않다.

const는 변경할 수 없는 값을 만드는 것이 아니라, 재할당을 할 수 없도록 하는 것이다. 따라서 객체, 배열과 같이 변경할 수 있는 값에 const를 사용하면 나중에 수정이 발생할 수 있다

```js
const studentIds = [12,41,51]
studentsIds.push(55) // 동작함
```

물론 `41`, `"name"`과 같은 원시값을 사용할 때는 변경할 수 없지만, 이런 이점을 취할 일은 거의 없다. const는 블록 스코프이고 블록은 짧아야한기에 실제로 const가 영향을 끼치는 범위는 작다.


### A.4.3 var와 let

var를 사용할 때
- 함수의 최상위 스코프에서 사용 (처음, 중간, 끝 관계없이). 전역에서는 되도록 사용하지 않음
- 반복문 내에서 선언한 변수를 조건문에서도 보고 싶을때
- 의도하지 않은 블록 내에 선언이 있을때. (ex : try...catch)

```js
function commitAction() {
	do {
		let result = commit();
		var done = result && result.code == 1;
	} while(!done)
}

function getStudents() {
	try {
		var records = fromCache("학생")
	} catch(e) {
		var records = []
	}
	return records
}
```

let을 사용할 때
- 블록 안에서만 사용하는 경우


이처럼 var의 함수스코프라는 특성과 여러번 안전하게 재선언이 가능하다는 특징은 잘 활용하면 유용하다.


## A.5 TDZ

TDZ가 필요한 이유에 대한 절

### A.5.1 모든 일이 시작된 곳, const

TC39에서는 ES6 개발 초기에 const와 let을 호이스팅 해야할 지 여부를 결정해야했다. 결과적으로 아래와 코드에서 블록 내부에서 일관된 변수만 사용하도록 하기위해 호이스팅 하도록 결정하였다. 

```js
let greeting = "hi"
{
	console.log(greeting)
	let greeting = "hello";
}
```

만약 호이스팅 되지 않으면 블록 전반부에는 상위 스코프의 변수가, 후반은 새로 선언된 변수가 적용되는 혼란스러운 동작이 발생했을 것이다.

let과 const가 var 처럼 undefined로 자동 초기화 되지 않는 이유 또한 이러한 호이스팅과 관련있다. 다음과 같은 경우, undefined로 초기화된다면 const는 상위에서는 undefined인데 아래에서는 값이 할당되고 그 이후에는 값이 재할당안되는 비직관적인 동작이 발생했을 것이다. (const는 상수라 변하면 안되는데 변경됨)

```js
{
	console.log(studentName)
	const studentName = "kevin"
}
```

이런 이유 때문에 let, const는 호이스팅 되지만 값이 없는 시간대가 있고 이 시간대에 변수에 접근할 경우 TDZ 오류를 발생시키자고 하였다

### A.5.2 곁다리 let

TC39 는 let 에 const와 마찬가지로 TDZ를 도입하자고 결정하였다. 이는 개발자들로 하여금 변수를 신중하게 사용하고, 변수를 그 변수를 사용하는 코드의 상단에 선언하도록 하여 가독성을 높여 도움이 된다. 

다만 저자의 생각은 다른 것 같다. let은 var와 마찬가지로 동작하는 것이 직관적이라고 생각하는 것 같다

## A.6 동기 콜백도 여전히 클로저일까?

### A.6.1 콜백이란, A.6.2 동기 콜백

콜백에는 비동기콜백, 동기 콜백이 있다. 
- 비동기 콜백 : 미래의 어느 시점에 되돌아와 다시 호출하는 함수.
- 동기 콜백 : `Array.map`의 인자와 같이 당장 호출되는 함수로, 되돌아올 곳이 없음. 콜백보다는 DI, IoC라는 용어가 적합함.

### A.6.3 동기 클로저

동기 콜백의 경우 콜백이 실행되는 위치가 자신이 선언된 스코프이다.

```js
function printLabels(labels) {
	var list = document.getElementById("labelsList");
	labels.forEach(
		function renderLabel(label) {
			var li = document.createElement("li");
			li.innerText = lebel;
			list.appendChild(li);
		}
	)
}
```

따라서 동기 콜백에서 외부 스코프의 변수를 참조하더라도 이는 자신의 스코프에서 참조한 것이지 클로저를 통해 접근한 것이 아니다. 따라서 동기 콜백의 경우는 클로저가 형성된다고 보기 어렵다

### A.6.4 클로저 지연

커링을 수동으로 적용하면 클로저를 사용할 수 있다

```js
function printLabels(labels) {
	var list = document.getElementById("labelsList");
	var renderLabel = renderTo(list);

	labels.forEach(renderLabel)

	function renderTo(list) {
		return function createLabel(label) {
			var li = document.createElement("li");
			li.innerText = lebel;
			list.appendChild(li);
		}
	}
}
```


## A.7 클래식 모듈 변형

클래식 모듈 패턴 예시

```js
var StudentList = (function defineModule(Student){
	var elems = [];

	var publicAPI = {
		renderList() {
		}
	}

	return publicAPI;
})(Student)
```

이와 같은 패턴은 나중에 변형해서 사용할 수 있다. 어떤 모듈 로더를 사용하더라도 클래식 모듈의 변형일 뿐이다.

### A.7.1 내 API는 어디에 있나요?

위 예시에서 `publicAPI`는 따로 객체로 생성하지 않고 return 문에 직접 선언할 수도 있고 이러한 방식이 더 많이 쓰인다. 하지만 저자는 따로 객체로 생성하는걸 선호하는데 이유를 다음과 같이 들었다.

- publicAPI 라는 이름 자체가 객체의 목적을 명확하게 해주므로 가독성을 높인다
- publicAPI를 따로 두면 모듈이 살아있는 동안 API에 접근하거나 수정할때 유용하다

### A.7.2 AMD (비동기 모듈 정의)

RequireJS 를 쓰면 쉽게 구현 가능한 모듈 패턴으로, 몇 년전 인기를 끈 스타일의 모듈이다

```js
define(["./Students.js"], function StudentList(Student) {
	var elems = [];

	return {
		renderList() {
			// ...
		}
	}
})
```

### A.7.3 UMD

UMD 는 구체적인 패턴이라기보다 비슷한 형식들의 모음에 가깝다. UMD는 브라우저, AMD 스타일 로더, Node.js 에서 모듈을 읽어올때, 별도의 변환도구 없이 더 나은 상호 운용성을 위해 설계되었다

```js
(function UMD(name, context, definition){
	// AMD 스타일일 경우
	if(typeof define === "function" && define.amd) {
		define(definition);
	} 
	// Node.js 일 경우
	else if(typeof module !== "undefined" && module.exports) {
		module.exports = definition(name, context);
	}
	// 독립형 브라우저 스크립트일 경우
	else {
		context[nama] = definition(name, context);
	}
})("StudentName", this, function DEF(name, context) {
	var elems = [];

	return {
		renderList() {
			// ...
		}
	}
})
```

