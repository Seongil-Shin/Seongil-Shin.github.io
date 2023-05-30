---
title: javascript 이벤트루프
author: 신성일
date: 2022-02-08 11:20:00 +0900
categories: [study, javascript]
tags: [javascript]
---

## **이벤트 루프**

javascript를 공부하다보면 아래와 같은 말을 종종 듣는다.

> 싱글스레드 기반으로 동작하는 자바스크립트
>
> 이벤트 루프를 기반으로 하는 싱글스레드 Node.js

정말 싱글 스레드인가? 어떻게 싱글 스레드인가? 이벤트 루프는 무엇인가? 를 간단히 알아보기 위해 자바스크립트가 동작하는 환경과 엔진에 대해 알아보자.

<br/>

### **Javascript Engine**

javascript로 작성한 코드를 해석하고 실행하는 인터프리터. 주로 웹브라우저에서 사용되지만, node.js에서는 V8 엔진이 사용된다.

구글에서 개발한 V8 엔진을 비롯해, 대부분의 자바스크립트 엔진은 다음 세 영역으로 나뉜다.

- Call Stack
- Task Queue(Event Queue)
- Heap

추가적으로 `Event Loop`가 존재하여 Task queue에 들어가는 task들을 관리하게 된다.

![image-20220209004707394](/assets/img/2022-02-08-javascript 질문/image-20220209004707394.png)

<br/>

#### **Call stack**

자바스크립트는 단 하나의 call stack을 사용한다. 따라서 자바스크립트에서는 하나의 함수가 실행하면, 다른 어떤 task도 수행될 수가 없다.

자바스크립트에서는 요청이 들어올때마다 해당 요청을 순차적으로 call stack에 담아 처리한다. 메소드가 실행될 때 call stack에 새로운 프레임이 생기고, push 되고, 메소드의 실행이 끝나면 해당 프레임은 pop되는 원리이다.

예시)

```javascript
function foo(b) {
  var a = 10;
  return a + b;
}
function bar(x) {
  var y = 2;
  return foo((x = y));
}
console.log(bar(1));
```

1. bar 가 스택에 들어간다.
2. foo 가 스택에 들어간다.
3. foo 의 실행이 완료되고, pop된다.
4. bar의 실행이 완료되고, pop된다.

<br/>

#### **Heap**

동적으로 생성된 객체(인스턴스)는 힙에 할당된다. 대부분 구조화되지 않는 더미 같은 메모리 영역을 heap이라고 표현한다.

<br/>

#### **Task Queue(Event Queue)**

자바스크립트의 런타임 환경에서 처리해야하는 task들을 임시 저장하는 대기 큐. call stack이 비어졌을 때 먼저 대기열에 들어온 순서대로 수행된다.

```javascript
function test1() {
  setTimeout(function () {
    console.log("first");
  }, 0);
  test2();
}
function test2() {
  console.log("second");
}
test1();
```

위 코드는 다음 결과를 낸다.

```javascript
second;
first;
```

자바스크립트에서 비동기로 호출되는 함수들은 call stack에 쌓이지 않고, task queue에 들어가기 때문이다. 따라서 위 작업은 다음 순서로 실행된다.

1. setTimeout가 call stack에 쌓인다.
2. setTimeout은 web api에서 실행될 수 있다. 따라서, setTimeout의 실행을 web API로 요청하고 setTimeout은 종료된다.
3. 0초이기에 Web API는 콜백함수를 바로 task queue에 등록한다.
4. 3번과 동시에, test2()가 call stack 안에 들어간다.
5. test2()가 종료되어 pop되고, test1()도 종료된 다음 pop된다.
6. task queue에 있던 익명함수가 빠져나와 실행된다.

이때 task queue 안에 들어있는 이벤트를 꺼내거나, 넣는 작업을 하는 것이 `이벤트 루프` 이다.

<br/>

다음과 같은 의문이 들 수 있다.

- 이벤트 루프는 백그라운드 스레드가 존재해서, call stack을 polling 하면서 비어있는지 확인하는 건가?
- task queue에도 event가 있는지 확인해야할 것 같은데, 이때는 polling 으로 검사하는 건가?
- event loop에 의해서 event queue에 있던 하나의 이벤트가 call stack에 들어간 다음에는 그 이벤트가 끝나기 전까지, 이벤트 루프는 event queue에서 이벤트를 꺼내지 않는가?
- call stack에서 이벤트가 진행 중일 때도 이벤트루프는 어떻게 확인하나?

위 질문에 대해 MDN은 다음의 가상 코드로 답을 한다.

```javascript
while (queue.waitForMessage()) {
  queue.processNextMessage();
}
```

이런 식으로 이벤트루프는 현재 실행중인 태스크가 없는지와 태스크 큐에 이벤트가 있는지 반복적으로 확인한다.

queue에 이벤트가 존재하면 while-loop 안으로 들어가서 해당하는 이벤트를 처리하거나 작업을 수행한다. 그리고 다시 queue로 돌아와 새로운 이벤트가 존재하는지 파악하는 것이다. 따라서 task queue에 있는 작업들은 한번에 하나씩 call stack으로 호출되어 처리된다.

<br/>

이때 브라우저에서 task queue는 다음과 같이 세가지 큐로 나뉜다.

![browser structure](/assets/img/2022-02-08-javascript 질문/browser-structure.png)

- 비동기로 처리되는 작업은 task, microtask, animationFrame으로 구분된다.
- microtask는 task보다 먼저 작업이 처리된다.
  - microtask : MutationObserver, Promise 가 해당
- microtask가 처리된 이후, requestAnimationFrame이 호출되고 이후 브라우저 렌더링이 발생한다.

## **출처**

https://asfirstalways.tistory.com/362

https://sculove.github.io/post/javascriptflow/
