---
title: You don't know JS Yet 2부 - 5장 변수의 비밀 생명주기
author: 신성일
date: 2024-11-30 19:00:00 +0900
categories: study
tags:
  - "#study"
---

변수 선언 위치에 따른 작동 방식의 차이와 사용 여부

## 5.1 변수 사용 가능 시점

모든 식별자는 컴파일 타임 때 각자의 스코프에 등록된다. 또 모든 식별자는 자신이 속한 스코프가 생성될 때 해당 스코프의 시작 부분에서 생성된다. 

이렇게 선언은 스코프 아래에 있더라도 스코프 시작부분에서 변수의 가시성이 확보되는 걸 **호이스팅**이라고 한다.

하지만 가시성을 넘어 함수의 경우 실제로 실행이 되기도 하는데, 이는 **함수 호이스팅** 때문이다. 함수 선언문으로 함수를 선언하면 함수 이름에 해당하는 식별자가 스코프 최상단에 등록되고, 함수 참조로 그 값이 자동으로 초기화된다. 따라서 함수는 스코프 어디서든 호출 가능해진다.

```js
hello();

function hello() {};
```

함수 호이스팅, var 호이스팅 모두 블록 스코프가 아닌 가장 가까운 함수 스코프에 등록된다. (함수가 없으면 전역으로)

### 선언문과 표현식에서의 호이스팅 차이

```js
hello(); // TypeError

var hello = function hello() {}
```

- var로 선언한 변수는 호이스팅이 되지만, 스코프가 시작될 때 undefined로 초기화됨. 코드상 할당문에 도달하기 전까지 undefined로 계속 남아있음. 따라서 `TypeError`가 난다. (참조는 가능하기에 `ReferenceError`는 나지 않음.)
- 함수 선언문의 경우 호이스팅이 되고 해당 함숫값으로 초기화되는 것과는 동작이 다르다

### 변수 호이스팅

```js
greeting = "hi";
console.log(greeting) // hi

var greeting = "hello";
```

- var는 호이스팅되며 undefined로 초기화되기에, 첫번째줄에서부터 값을 할당할 수 있다ㅏ

## 5.2 호이스팅 : 비유일 뿐입니다

호이스팅은 아래처럼 코드를 재정렬하는 것으로 생각하기가 쉽다.

```js
var greeting;        // 호이스팅됨
greeting = "hi";
console.log(greeting) // hi

greeting = "hello";
```

하지만 실제 JS 엔진은 코드를 재정렬하지 않는다. JS 엔진은 앞을 내다보고 선언문을 찾지 못한다. 프로그램의 모든 스코프 경계와 선언문을 정확히 찾는 것은 파싱 과정이다. 

물론 호이스팅을 코드 재정렬로 생각할 수 있다. 이는 이해에 도움을 주지만, 호이스팅은 런타임이 아닌 컴파일 단계에서 일어나는 일임을 기억해야한다

## 5.3 중복 선언 처리하기

- var 중복 선언 : 두번째 선언이 무시됨

```js
var name = "name"
console.log(name); // "name" 
var name;
console.log(name); // "name"
```

아래와 같이 이해할 수 있음. 

```js
var name;
var name;       // 의미 없는 작업

name = "name";
console.log(name); // "name" 

console.log(name); // "name"
```

이때 `var name;`과 `var name = undefined;`는 큰 차이가 있다. 만약 후자처럼 선언하였으면 다음과 같이 동작한다.

```js
var name;
var name;       // 의미 없는 작업

name = "name";
console.log(name); // "name" 
name = undefined;
console.log(name); // "name"
```

- 함수와 var의 중복선언 : 함수 호이스팅이 변수 호이스팅보다 우선순위가 높음.

```js
var hello;

function hello() {
	console.log("hello");
}

var hello;   // 의미 없는 작업

typeof hello; // "function"

var hello = "hi"

typeof hello; // "string"
```

- let 중복 선언 : SyntaxError 발생하며 실행이 안됨. (하나가 let이고 나머지가 var 인 경우도 안됨)
```js
let name = "name";
let name = "name2";
```


let 중복 선언은 기술적으로는 가능하지만 TC39에서 스타일 상 막아놓은 것이다.


### const 재선언

const도 let과 마찬가지로 동일한 스코프 내 동일 식별자를 사용할 수 없다. 하지만 const는 let과 달리 재선언을 기술적인 이유로 막아놓은 것이다

const는 변수를 선언할 때 할당도 필요하다. 만약 선언 시 할당을 빼놓으면 다음과 같이 `SyntaxError`가 발생한다. 또한 재할당도 불가능하다
```js
const empty; // SyntaxError
```

따라서 만약 const를 재선언하게 되면 어떤 형식이든 SyntaxError 가 발생한다

```js
const name = "name1";

const name; // SyntaxError
const name = "name2"; // SyntaxError
name = "name3" // TypeError
```

> SyntaxError vs TypeError
> SyntaxError 는 프로그램 실행전에 발생하여 프로그램을 실행시키지 않지만, TypeError는 프로그럼 실행 중 발생한다


### 반복문

스코프 규칙 (let으로 생성한 변수 재선언 등)은 각 스코프 인스턴스마다 적용된다. 

반복문에서는 새로운 반복이 시작될 때마다 자체적인 새 스코프가 생성된다. 따라서 다음과 같은 케이스는 let 재선언이 일어나지 않고, 매번 새로운 let을 선언하는 것이고 에러가 발생하지 않는다. (let은 블록 스코프이기에)

```js
var keepGoing = true;
while(keepGoing) {
	let value = Math.random();
	if(value > 0.5) {
		keepGoing = false;
	}
}
```

하지만 만약 var로 선언한 경우, var는 함수 스코프이기에 전역스코프에 연결이 되어 재선언은 일어나지 않지만, 재할당이 매번 일어난다.

```js
var keepGoing = true;
while(keepGoing) {
	var value = Math.random();
	if(value > 0.5) {
		keepGoing = false;
	}
}
```

for문의 경우도 마찬가지인데, for 조건문 안에 있는 변수들도 매번 새로운 스코프에 연결되므로 에러가 발생하지 않는다.

```js
for(let i = 0; i < 3; i++) {  // 조건문 속 let은 매번 새로운 인스턴스와 연결됨.
	let value = i * 30;
}
```

const의 경우도 마찬가지인데, 다만 특정 형태의 for 문에서는 문제가 발생한다. 

```js
var keepGoing = true;
while(keepGoing) {
	const value = Math.random();   // 문제 없음
	if(value > 0.5) {
		keepGoing = false;
	}
}

for(const student of students) {  // 조건문 속 const는 매번 새로운 스코프와 연결됨
}

for(const i = 0; i < 3; i++) {  // TypeError 발생
	let value = i * 30;
}
```

이는 반복문 종료 이후, const 로 선언한 `i`에 `i++`라는 재할당이 발생하기에 에러가 발생하는 것이다.


## 5.4 초기화되지 않은 변수와 TDZ

TDZ : 변수는 존재하지만 초기화되지 않아 어떤 방식으로도 해당 변수에 접근할 수 없는 시간대. 초기화가 이루어지면 TDZ는 종료되고 스코프 내에서 변수를 자유롭게 사용할 수 있다.

아래와 같은 코드는 프로그램 실행시 ReferenceError가 발생한다.
```js
console.log(name);

let name = "eil";
```

let, const로 선언된 변수는 호이스팅은 되지만, 스코프 맨 위에서 초기화가 발생하지 않기에 에러가 발생하는 것이다. 컴파일러는 프로그램 중간에서 해당 선언을 초기화하는 명령을 내리기에, 이 초기화가 발생하기 전까지는 TDZ에 걸려 변수를 사용할 수 없다.  (엄밀히는 var도 TDZ가 있지만 길이가 0이다.)

let, const로 선언된 변수도 호이스팅은 되지만, 자동 초기화가 이뤄지지 않는다는 것만 다르다. 호이스팅 되는 것은 아래 코드가 에러를 발생시킨다는 것으로 알 수 있다.

```js
var studentName = "카일"
{
	console.log(studentName);  // TDZ 에러발생
	let studentName = "지수";
}
```



