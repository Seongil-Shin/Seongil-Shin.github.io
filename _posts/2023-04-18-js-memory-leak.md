---
title: Javscript 메모리 누수 방지 및 성능 개선
author: 신성일
date: 2023-04-18 22:00:00 +0900
categories: [study, javascript]
tags: []
---



# Javscript 메모리 관리 이해하기

## Garbage collector

자바스크립트 엔진은 더이상 사용하지 않을 메모리를 놓아주기 위해 `garbage collector`를 사용한다. `garbage collector(GC)`는 앱에서 더이상 사용하지 않을 객체를 찾아내고 삭제한다. 따라서 GC는 앱의 object와 변수를 계속 모니터링하고 어떤 것이 여전히 referenced 되고 있는지 트랙킹한다.  그러다 Object가 더이상 쓰이지 않으면 마킹하고 삭제하여 메모리를 놓아준다.

GC가 사용하는 이 기법은 `mark and sweep`이다. 

1. 아직 사용중인 모든 Object를 마킹함 (mark)
2. heap을 검사하고, 마킹되지 않은 object는 삭제함. (sweep)

이 작업은 주기적으로 수행되며, heap 의 사이즈가 작더라도 수행된다.



## Stack vs Heap

자바스크립트의 메모리를 얘기하면 **stack**과 **heap**이 주역이다.

stack은 함수 실행 동안 필요한 데이터를 저장한다. 빠르고 효율적이지만 공간은 한정적이다. 함수가 실행되면 자바스크립트 엔진이 그 함수의 변수와 파라미터를 스택에 push하고, 함수가 return 되면 pop 한다. 이렇게 stack은 빠르고 효율적으로 메모리를 관리한다.

반면에 heap은 앱의 생명주기 동안 데이터를 저장하는데 사용한다. stack에 비해 다소 느리고 덜 정리되어있지만 공간은 크다. Heap은 object, array 등 복잡한 데이터 구조를 여러번 접근하기 위해 사용된다.

<br/>

# Common causes of Memory Leaks

## 순환참조(Circular reference)

가장 흔한 케이스 중 하나다. 2개 이상의 object가 서로를 참조하고 있을 때 발생한다.

```js
let object1 = {};
let object2 = {};

// create a circular reference between object1 and object2
object1.next = object2;
object2.prev = object1;

// do something with object1 and object2
// ...

// set object1 and object2 to null to break the circular reference
object1 = null;
object2 = null;
```

위 예제에서는 마지막에 `object1`과 `object2`를 null로 설정하여 순환 참조를 깨려고 했지만, `{next: refenceToObject2}`, `{ prev: refenceToObject1}`는 각각 메모리에 남아있다. 따라서  GC는 순환 참조를 꺨 수 없기에 메모리가 그대로 남아있게 된다.

이러한 경우에는 `delete` 키워드를 사용한  `manual memory management`가 필요하다.  `delete`를 사용하여 순환참조를 만들어내는 property를 삭제하는 것이다.

```js
delete object1.next;
delete object2.prev;
```

또다른 방법은 `WeakMap`, `WeakSet`을 사용하는 방법이다. 이 두개는 object와 variable에 대한 약한 참조를 만들 수 있게 한다. 

## Event Listeners

엘리먼트에 Event Listener를 붙이면, listener function의 reference가 생긴다. 그리고 이 reference는 GC가 메모리를 풀어주는걸 막는다. 이는 앨리먼트가 더이상 필요없어져도 listener function이 제거되지 않으므로 memory leak의 원인이 된다.

```javascript
let button = document.getElementById("my-button");

// attach an event listener to the button
button.addEventListener("click", function() {
  console.log("Button was clicked!");
});

// do something with the button
// ...

// remove the button from the DOM
button.parentNode.removeChild(button);
```

이를 해결하기 위해선 필요없어진 event listener는 삭제해야한다.

```javascript
button.removeEventListener("click", function() {
  console.log("Button was clicked!");
});

// 전부 삭제
button.removeAllListeners();
```

## Global variables

전역 변수를 생성하면, 그 변수는 모든 코드에서 접근이 가능하다. 따라서 더 이상 필요한지 아닌지 판단하기가 어렵다. 이는 memory leak로 이어진다.

```javascript
// create a global variable
let myData = {
  largeArray: new Array(1000000).fill("some data"),
  id: 1
};

// do something with myData
// ...

// set myData to null to break the reference
myData = null;
```

위 코드의 마지막에 `myData`에 `null`을 할당하였지만, 전역 변수이기에 모든 코드에서 접근 가능하다. 따라서 더이상 필요한지 아닌지 판단하기 어렵고, 이는 memory leak로 이어진다. 

이를 해결하기 위해선 `Function Scoping`을 사용해야한다. 함수를 생성하고 그 안에서 지역변수로 변수를 생성하는 방법이다. 그러면 그 변수는 오직 그 함수에서만 접근 가능하고, 더이상 필요한지 아닌지 판단하기 쉽다. 즉 필요없어지면 GC에 의해 청소된다.

```javascript
function myFunction() {
  let myData = {
    largeArray: new Array(1000000).fill("some data"),
    id: 1
  };

  // do something with myData
  // ...
}
myFunction();
```

다른 방법은 `var` 대신에 `let`, `const`를 사용하는 방법이다. `let`, `const`는 block-scoped이므로 선언된 block 외에선 사용이 불가능하니 GC에 의해 제거될수 있다.

```javascript
{
  let myData = {
    largeArray: new Array(1000000).fill("some data"),
    id: 1
  };

  // do something with myData
  // ...
}
```

<br/>

## Best Practices for Manual Memory Management

## Using weak references

`WeakMap`, `WeakSet` 을 사용하여 object와 variable에 대한 weak reference를 만들 수 있다. **Weak reference**는 일반적인 reference와는 달리 GC가 object에 의해 사용되는 메모리를 free up 하는 걸 막지 않는다. 이러한 특성은 `WeakMap`과 `WeakSet`을 Circular reference를 해결하는데 사용할 수 있도록 한다.

```javascript
let object1 = {};
let object2 = {};

// create a WeakMap
let weakMap = new WeakMap();

// create a circular reference by adding object1 to the WeakMap
// and then adding the WeakMap to object1
weakMap.set(object1, "some data");
object1.weakMap = weakMap;

// create a WeakSet and add object2 to it
let weakSet = new WeakSet();
weakSet.add(object2);

// in this case, the garbage collector will be able to free up the memory
// used by object1 and object2, since the references to them are weak
```

위에서 순환참조 때 봤던 예시와 비슷하게 서로 참조하고 있더라도, 이번엔 WeakMap을 사용하여 순환참조가 일어나도 메모리 free up을 막지 않는다.

## Using Garbage Collector API

GC API를 사용하는 방법도 있다. GC를 수동적으로 동작하게 하고, 현재 heap의 상태를 알 수 있다. 따라서 memory leak나 성능 이슈를 디버깅하는데도 도움을 준다.

```javascript
let object1 = {};
let object2 = {};

// create a circular reference between object1 and object2
object1.next = object2;
object2.prev = object1;

// manually trigger garbage collection
gc();
```

`gc()`를 실행하여 GC를 수동적으로 실행하면 설령 아직 참조되고 있더라도 object에 의해 사용되고 있던 메모리를 비운다. 

`gc()`는 모든 자바스크립트 엔진에서 지원되지는 않는다. 그리고 엔진에 따라 동작이 다르다. 또한 성능에 영향을 줄 수 있기에 신중하게 필요할 떄만 사용해야한다.

## Using `heap snapshots` and `profilers`

Javascripts는 `heap snapshot`과 `profilers`를 제공하여 어플리케이션이 어떻게 메모리를 사용하고 있는지 알려준다. 

`Heap Snapshot`은 현재 heap 상태의 스냅샷을 찍어주고, 그것을 분석하여 어떤 object가 가장 많은 메모리를 사용하고 있는지 알 수 있게 해준다.

```javascript
// Start a heap snapshot
let snapshot1 = performance.heapSnapshot();

// Do some actions that might cause memory leaks
for (let i = 0; i < 100000; i++) {
  myArray.push({
    largeData: new Array(1000000).fill("some data"), 
    id: i
  });
}

// Take another heap snapshot
let snapshot2 = performance.heapSnapshot();

// Compare the two snapshots to see which objects were created
let diff = snapshot2.compare(snapshot1);

// Analyze the diff to see which objects are using the most memory
diff.forEach(function(item) {
  if (item.size > 1000000) {
    console.log(item.name);
  }
});
```

위 예시에서는 루프 실행 전후로 스냅샷을 찍고, 두 스냅샷을 비교하여 어떤 object가 메모리를 많이 차지하는지 알 수 있게 해준다.

`Profiler`는 어플리케이션의 성능을 트랙킹하고, 메모리 사용이 많은 area를 인식한다.

```javascript
let profiler = new Profiler();

profiler.start();

// do some actions that might cause memory leaks
for (let i = 0; i < 100000; i++) {
  myArray.push({
    largeData: new Array(1000000).fill("some data"), 
    id: i
  });
}

profiler.stop();

let report = profiler.report();

// analyze the report to identify areas where memory usage is high
for (let func of report) {
  if (func.memory > 1000000) {
    console.log(func.name);
  }
}
```

`heap snapshot`과 `profilers`은 모든 엔진과 브라우저에서 지원되는 것은 아니다.

<br/>

## 출처

https://itnext.io/javascript-memory-management-how-to-avoid-common-memory-leaks-and-improve-performance-c018dbbca954#9f04

