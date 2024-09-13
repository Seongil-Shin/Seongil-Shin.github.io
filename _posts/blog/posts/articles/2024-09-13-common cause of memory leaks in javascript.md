---
title: common cause of memory leaks in javascript
author: 신성일
date: 2024-09-12
categories: study, article
tags:
  - "#study"
---
https://www.trevorlasn.com/blog/common-causes-of-memory-leaks-in-javascript

## Understanding Memory Usage In Node.js (V8)

Node.js(V8)에서 사용하는 메모리 타입들 (`process.memoryUsage()` 을 통해 확인 가능)

| Memory Type             | Description                                                                                                                              |
| ----------------------- | ---------------------------------------------------------------------------------------------------------------------------------------- |
| RSS (Resident Set Size) | Node.js 프로세스에 할당된 모든 메모리(heap, stack, code 등등)                                                                                           |
| Heap Total              | Javascript 객체를 위해 할당된 메모리. heap 에 할당된 메모리의 사이즈이다                                                                                         |
| Heap Used               | Javascript 객체에 의해 실제로 사용된 메모리. 현재 얼마나 heap이 사용중인지 나타냄                                                                                    |
| External                | Javascript 객체와 연결된 C++ 객체에의해 사용중인 메모리. V8 heap 영역 외부에서 관리되는 메모리. Javascript가 native code와 상호작용하는 연결에 의해 사용된다. Javascript heap 영역 바깥에 할당됨 |
| Array Buffers           | ArrayBuffer 객체를 위해 할당된 메모리이며, raw binary data를 저장하기 위해 사용된다                                                                              |


## Common Causes of Memory Leaks in Javascript

### 1. Improperly Managed Variables

에를들어 임시로 선언한 변수를 잊어버리고 지우지 않으면 계속 메모리를 가지고 있는다

```js
let cache = {};

function storeData(key, value) {
    cache[key] = value;
}

// Simulating the function being called multiple times
storeData('item1', new Array(1000000).fill('A'));
storeData('item2', new Array(1000000).fill('B'));

// Memory leak: data stored in 'cache' is never released
```

- cache가 글로벌 객체에 추가되어있기에 더이상 필요없어져도 메모리에 계속 남아있는다
- global scope 에 저장되면 애플리케이션 라이프사이클 내내 살아있기에 더욱 문제가 된다.

### 2. Persistent Global Objects

전역 객체는 더이상 필요가 없어져도 계속 메모리를 차지한다. 따라서 다음과 같이 전역 객체에 선언하는 것보다는

```js
global.config = {
    settings: new Array(1000000).fill('Configuration')
};
```

아래처럼 지역적으로 선언하는 것이 좋다

```js
function createConfig() {
    return {
        settings: new Array(1000000).fill('Configuration')
    };
}

// Use config only when needed, and let it be garbage collected afterwards
function processConfig() {
    const config = createConfig();
    // Perform operations with config
    console.log(config.settings[0]);

    // Config will be cleared from memory once it's no longer referenced
}

processConfig();
```

### 3. Event Listeners Not Removed

이벤트리스너는 넘겨진 함수와 거기서 사용하는 변수의 참조를 가지고 있다. 이는 GC가 메모리를 정리하는걸 막는다. 따라서 만약 사용하지 않는 이벤트 리스너를 지우지 않고 유지하면 메모리 누수가 발생하게 된다

### 4. Closures Capturing Variable

clousre 에 의도치않게 변수가 필요가 없음에도 남아 있는 경우가 있다. 
```js
function createClosure() {
    let capturedVar = new Array(1000000).fill('Data');

    return function() {
        console.log(capturedVar[0]);
    };
}

const closure = createClosure();
```

누수를 예방하기 위해서는 closure가 불필요하게 큰 변수를 가지지 않도록하고, 필요가 없어지면 삭제해야한다

```js
function createClosure() {
    let capturedVar = new Array(1000000).fill('Data');

    return function() {
        console.log(capturedVar[0]);
        capturedVar = null; // Release memory when no longer needed
    };
}

const closure = createClosure();
closure(); // 'capturedVar' is released after use.
```

### 5. Unmanaged Callbacks 

콜백이 불필요한 객체를 가지고 있는 경우가 있다. 

```js
function fetchData(callback) {
    let data = new Array(1000000).fill('Data');

    setTimeout(() => {
        callback(data);
    }, 1000);
}

function handleData(data) {
    console.log(data[0]);
}

fetchData(handleData); // The 'data' array remains in memory.
```

JS GC는 더이상 필요하지 않는 메모리를 효과적으로 정리하기에, 특수한 시나리오가 아니면 수동으로 메모리를 정리해주지 않아도 된다. 

**Avoiding Unnecessary Complexity**

일반적으로 비동기 콜백에 수동 메모리 정리는 하지 않아도 된다. 아래 코드가 불필요하게 메모리를 정리한 예시다

```js
function fetchData(callback) {
  let data = new Array(1000000).fill('Data');

  setTimeout(() => {
      callback(data);
      data = null; // Release the reference
      global.gc(); // Explicitly trigger garbage collection
  }, 1000);
}

function handleData(data) {
  console.log(data[0]);
  data = null; // Clear reference after handling
}

console.log('Initial Memory Usage:', process.memoryUsage());

fetchData(handleData);

setTimeout(() => {
  console.log('Final Memory Usage:', process.memoryUsage());
}, 2000); // Give some time for garbage collection
```


### 6. Incorrect Use of bind()

`bind()`는 `this`를 세팅하면서 새로운 함수를 생성한다. 그런데 만약 조심스럽게 사용하지 않으면 메모리 누수로 이어질 수 있다.

```js
function MyClass() {
    this.largeData = new Array(1000000).fill('leak');

    window.addEventListener('click', this.handleClick.bind(this));
}

MyClass.prototype.handleClick = function() {
    console.log('Clicked');
};
```

- `bind()`를 사용하면, 새로운 함수는 기존 함수와 기존함수의 `this`를 기억한다. 만약 함수를 지우더라도 메모리에 계속 함아있는것이다.

### 7. Circular References

순환 참조가 발생하면 아래처럼 그 부모의 연결을 끊어도 메모리에 계속 남아있을 수 있다.

```js
function CircularReference() {
    this.reference = this; // Circular reference
}

let obj = new CircularReference();
obj = null; // Setting obj to null may not free the memory
```


**순환 참조 예방법**

1. Break the Loop : 순환참조가 발생하는 부분을 직접 끊어야한다.
```js
obj.reference = null; 
```

2. Use Weak References : WeakMap, WeakSet, WeakRef를 사용하여 references가 있어도 GC가 정리할 수 있도록 설정가능하다

```js
let weakMap = new WeakMap();

function CircularReference() {
    let obj = {};
    weakMap.set(obj, "This is a weak reference");
    return obj;
}

let obj = CircularReference();
```


**Quick Note**

WeakMap 등은 누수를 막기위한 좋은 도구지만, 매번 사용할 필요는 없다. 그보다 더 좋은 예시가 있다

- Profiling Memory Usage : 메모리 누수를 찾기위해 메모리가 얼마나 사용되는지 테스팅과정에 추가할 수 있다. 메모리 사용량을 감시하는 코드를 추가하고, 함수에 스트레스 테스트를 하는 것이다.
- Pay particular attention to 
	- High CPU usage functions
	- Memory-intensive functions
	- Event Loop and Garbage Collection : GC가 가장 시간을 많이 사용한 퍼센트가 얼마나 되는지 확인하라. 이는 어플리케이션이 얼마나 메모리 정리에 애를 먹고 있는지 알려준다.





