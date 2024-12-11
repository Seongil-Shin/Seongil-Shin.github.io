---
title: You don't know JS Yet 2부 - 7장 클로저 사용법
author: 신성일
date: 2024-12-09 19:00:00 +0900
categories: study, YDKJSY
tags:
  - "#study"
---
클로저는 최소 노출의 원칙 (POLE)을 기반으로 한다. 변수를 오랫동안 유지해야하는 경우 클로저를 사용하면 변수를 외부 스코프에 두는 대신 더 제한된 스코프로 캡슐화 할 수 있다.

함수 내부에서 함수 밖 해당 변수에 계속 접근할 수 있어 변수를 더 넓은 범위에서 사용할 수 있는 이점이 있다.


## 7.1 클로저 관찰하기

- 클로저는 함수에서만 일어나는 함수의 동작이다
- 클로저를 관찰하려면 함수를 반드시 호출해야한다
- 호출한 함수는 해당 함수를 정의한 스코프 체인이 아닌 다른 분기에서 호출되어야한다
- 클로저는 함수 인스턴스에 따라 다르게 생성된다

```js
function seeCloser(name) {
	var random = Math.random();

	return function printRandom() {
		console.log(`${name}님, ${random}입니다.`)
	}
}

const print = seeCloser("홍철");
print(); // 홍철님, 0.123213입니다.
const print2 = seeCloser("지후");
print2(); // 지후님, 0.1242131입니다.
```

만약 클로저가 없었다면 printRandom은 해당 함수 내 스코프에 있는 변수만 접근가능하여 `print` 호출 시 ReferenceError가 발생했을 것임. 하지만 클로저가 있기에 에러가 발생하지 않음.

이렇게 내부 함수가 외부 스코프에 있는 변수를 참조하는 것을 클로저라고 한다.

이러한 클로저는 렉시컬 스코프에 기반을 두고 있고 컴파일 시 처리되긴 하지만, 실제로 클로저 동작은 실행 시점에 함수 인스턴스에 따라 달라지는 특성을 가진다.

### 화살표 함수의 스코프

화살표 함수에서도 마찬가지로 스코프가 형성되며 클로저가 만들어진다.

```js
function seeCloser(name) {
	var random = Math.random();

	return () => console.log(`${name}님, ${random}입니다.`);
}

const print = seeCloser("홍철");
print(); // 홍철님, 0.123213입니다.
```

### 스냅샷이 아닌 라이브 링크

클로저는 값을 저장하는 스냅샷이 아닌 실시간으로 변수에 언제든 접근 가능하도록 관계를 맺어주는 라이브링크이다. 따라서 값을 읽을 뿐 아니라 수정도 할 수 있다.

```js
function counter() {
	var count = 0;

	return function getCurrent() {
		return count++;
	}
}

var hits = counter();
hits(); // 0
hits(); // 1
```

클로저는 감싼 변수와 연결을 맺어주는 것이므로 아래와 같은 동작이 일어날 수 있다. 

```js
var name = "지수";

function hello() {
	return function greeting(gret) {
		console.log(`${name}님, ${gret}`)
	}
}

name = "한주"

var hi = hello();
hi("hi") // 한주님, hi
```

아래와 같은 의도치 않은 동작도 일어날 수도 있다. 원래 의도는 각각 0,1,2 를 반환하는 것이었지만 var가 한번만 선언했기에 keeps에 생성된 모든 인스턴스들은 해당 변수와 연결된 것이다.

```js
var keeps = [];

for(var i = 0; i<3; i++) {
	keeps[i] = function keepI() {
		return i;
	}
}

keeps[0](); // 3
keeps[1](); // 3
keeps[2](); // 3
```

이러한 의도치 않은 동작은 setTimeout 등 비동기 함수를 실행할 때 자주 일어난다

### 쉽게 관찰할 수 있는 클로저 : ajax 와 이벤트

이벤트 등 비동기 함수를 등록할때 보통 콜백과 함께 등록하는데, 이때 클로저를 자주 관찰할 수 있다.

```js
function listeanForClicks(btn, label) {
	btn.addEventListener("click", function onClick() {
		console.log(`${label} 버튼을 클릭했습니다`)
	})
}
```


### 관찰 가능성 관점에서 클로저의 정의

> 클로저는 함수가 외부 스코프의 변수를 사용하면서, 그 변수에 접근하지 않은 다른 스코프에서 실행될 때 관찰된다.

## 7.2 클로저 생명주기와 가비지 컬렉션

클로저는 본질적으로 함수의 인스턴스와 연결되므로, 이 함수를 참조하는 함수가 있는한 변수에 대한 클로저는 지속된다. 함수 참조가 삭제되면 변수에 대한 클로저가 사라지고 변수는 GC 처리된다.

이러한 특성은 의도치 않은 메모리 사용을 급증시키는 요인이 될 수 있다. 따라서 더이상 필요하지 않은 함수참조를 삭제하는 것은 중요하다 (이벤트핸들러 제거 등)

### 변수 혹은 스코프

개념만 따졌을때 클로저는 변수를 기준으로 동작하기에 클로저를 생성하는 함수 내에서 사용하지 않는 변수는 클로저에 포함되지 않는다.

```js
function manageStudentGrades(studentRecords) {
	var grades = studentRecords.map(getGrades)

	return addGrade;

	function getGrade(record) {
		return record.grade;
	}

	function sortAndTrimGradesList() {
		grades.sort(function desc(g1, g2) {
			return g2 - g1;
		})
		grades = grades.slice(0, 10);
	}

	function addGrade(newGrade) {
		grades.push(newGrades);
		sortAndTrimGradesList();
		return grades;
	}
}

var addNextGrade = manageStudentGrades([
	{grade:65}, {grade:12}, {grade:54}
])

addNextGrade(92);
```

- `sortAndTrimGradesList` : `addGrade` 에서 사용함으로 클로저에 포함됨
- `grades` : `addGrade` 에서 사용하여 클로저에 포함됨
- `studentRecords`, `getGrade` : `addGrade`, `sortAndTrimGradesList`  모두에서 사용하지 않아 클로저에 포함되지 않음

하지만 아래 코드와 같이 `eval`을 사용하는 경우는 다르다. 

```js
function storeStudentInfo(id, name, grade) {
	return function getInfo(witchValue) {
		var val = eval(whichValue);
		return val
	}
}

var info = storeStudentInfo(72, "지희", 82);
info("name")
```

디버깅을 해보면 id, name, grade 모두 클로저에 포함된다. 이 이유는 아래와 같다

- 클로저는 스코프를 기준으로 형성된다.
- 하지만 현대 JS 엔진 상당수는 최적화를 위해 함수에서 실제로 참조하지 않는 변수는 클로저에서 제외한다. 
- `eval`을 사용하는 경우에는 어떤 변수에 접근할지 결정할 수 없기에 이러한 최적화가 동작하지 않는다.


이러한 최적화는 명세서에 언급도니 동작이 아니다. 따라서 최적화가 항상 동작할거라고 과신하면 안된다. 만약 어떤 큰 값이 클로저 스코프에 포함된다면 수동으로 값을 버리는 것이 안전하다.


## 7.3 다른 관점

현재 관점에서 스코프는 함수의 일급값이라는 특성을 기반으로, 함수가 어디로 이동하든 해당 함수를 외부 스코프에 있는 변수와 연결해주는 링크 역할을 한다고 이해할 수 있다

하지만 여기서 JS는 함수를 참조에 의해 저장되고 참조/복사를 통해 할당/전달된다는 점을 강조하면 이해도가 높아진다.
![](/assets/images/Pasted%20image%2020241211190534.png)


- 함수 인스턴스는 자신이 생성된 위치에 그대로 있음. 반환된 것은 함수의 인스턴스의 참조.
- 반환된 함수 인스턴스 참조를 통해 다른 곳에 있는 함수를 호출한다고 생각하면, 단순히 렉시컬 스코프만으로 설명할 수 있음.
- 이 관점에서 클로저를 정의하면 클로저는 프로그램의 다른 부분에서 해당 함수 인스턴스 참조가 살아있으면, 그 함수 인스턴스와 전체 스코프 환경 및 스코프 체인을 살아있게 유지하는 마법이라 볼 수 있다.


기존 모델은 학문적인 관점에서 만들어진 모델이고, 새로운 관점은 실제로 구현할때 어떻게할지에 초점을 맞춘 관점이다. 어떤 모델을 선택하든 관찰가능한 결과는 같다.


## 7.4 클로저를 사용하는 이유

클로저를 기반으로 하는 함수형 프로그래밍 패러다임의 대표적인 기법 두 가지는 "부분 적용"과 "커링"이다. 간략히는 여러 입력이 필요한 함수의 모양일 바꿔서 미리 입력하거나 나중에 입력하는 기법이다. 초기 입력은 클로저를 통해 기억된다.

클로저를 통해 내부에 정보를 캡슐화하는 함수 인스턴스를 만들면 나중에 입력을 다시 제공할 필요 없이, 정보를 저장한 함수를 다시 사용할 수 있다.


## 7.5 정리

클로저를 이해하기 위한 두가지 모델
- 관찰 관점 : 클로저는 함수가 다른 스코프로 전달되거나 호출될 때에도 외부 변수를 기억하는 함수 인스턴스이다
- 구현 관점 : 클로저는 다른 스코프에서 참조가 전달되고 호출되는 동안 함수 인스턴스와 해당 스코프 환경을 제자리에 보존한다.

클로저를 사용했을때의 이점
- 함수 인스턴스가 매번 계산할 필요없이 이전에 결정된 정보를 기억해내어 함수의 효율성을 높인다
- 함수 인스턴스 안에 변수를 캡슐화해 코드 가독성을 개선하고 스코프 노출을 제한하는 동시에 나중에 변수에 있는 정보를 사용할 수 있도록 보장한다. 함수를 호출할때마다 정보를 전달할 필요가 없다.