---
title: useState 원리
author: 신성일
date: 2023-05-06  18:00:00 +0900
categories: [study, react]
tags: [react]
---

## State 변경 시 어떤 일이 벌어질까?

리액트의 함수형 컴포넌트는 최초에 한번 실행이 되면서 초기값으로 설정해놓은 상태를 기억한다.

```js
const [state, setState] = useState(0);
```

이후 `setState`가 호출되어 상태가 변경된다면 다시 함수형 컴포넌트가 실행되고 virtual DOM을 리턴한다. 그리고 이전에 리턴했던 virtual DOM과 비교해서 state 값이 달라졌다면 달라진 부분에 해당하는 DOM만 업데이트한다.

## useState와 Closure

클래스형 컴포넌트는 `render()` 메서드를 통해 상태 변경을 감지할 수 있다. 반면 함수형 컴포넌트는 렌더링이 발생하면 함수 자체가 다시 호출된다. 그래서 상태 관리를 위해선 함수가 다시 호출되었을 때 이전 상태를 기억하고 있어야한다.

useState는 Closure를 통해 이 문제를 해결한다.

> Closure는 내부함수에서 상위 함수 스코프의 변수에 접근할 수 있는 개념

다음은 클로저의 특징을 이용한 `MyReact` 모듈이다

```react
const MyReact = (function() {
  let _val // hold our state in module scope
  return {
    render(Component) {
      const Comp = Component()
      Comp.render()
      return Comp
    },
    useState(initialValue) {
      _val = _val || initialValue
      function setState(newVal) {
        _val = newVal
      }
      return [_val, setState]
    }
  }
})()
```

- MyReact는 익명함수로부터 두개의 Closure를 반환받아 저장한다.
- `_val`은 익명함수 scope 안에서 정의된다. 하지만 반환되는 함수의 Closure에서 사용되기에 익명함수가 종료되더라도 메모리에 유지된다. 

이 `MyReact` 함수는 다음과 같이 사용할 수 있다.

```react
function Counter() {
  const [count, setCount] = MyReact.useState(0)
  return {
    click: () => setCount(count + 1),
    render: () => console.log('render:', { count })
  }
}
let App
App = MyReact.render(Counter) // render: { count: 0 }
App.click()
App = MyReact.render(Counter) // render: { count: 1 }
```

Counter가 일종의 컴포넌트라고 생각해보자. 

- MyReact의 render를 이용해 첫 Counter를 렌더링한다. 
- 렌더링하면 MyReact의 `useState`가 실행된다. `_val`에 아무것도 없기에 초기값으로 설정된다.
- `useState`는 반환값으로 상태와 setter를 낸다. 그 후 Counter는 `click`과 `render`가 들어있는 객체를 반환해 App에 저장한다. 
- App.click을 통해 상태를 업데이트한다. 이러면 리렌더링이 되는데, 현재 MyReact에는 이러한 로직이 구현되어있지 않다. 실제 React에서는 setter가 실행될 경우 컴포넌트를 리렌더링한다. 여기서는 `click` 이후 `render`를 실행함으로써 리렌더링을 보여준다.
- Counter가 다시 실행되는데 이번엔 메모리에 저장된 `_val`에 값이 있기에 초기값을 설정하는 대신 업데이트된 값을 계속 유지한다.
- 새롭게 반환된 객체는 `count`가 1인 상태로 동작한다. 

하지만 위와 같은 useState엔 문제점이 있다. 바로 여러 useState를 사용할 경우 다 같은 `_val`을 바라본다는 점이다. 이를 해결하기 위해선 다음과 같이 할 수 있다. 

```react
let state = [];
let setters = [];
let cursor = 0;
let firstrun = true;

const createSetter = (cursor) => {
  return (newValue) => {
    state[cursor] = newValue;
  };
};

const useState = (initialValue) => {
  if (firstrun) {
    state.push(initialValue);
    setters.push(createSetter(cursor));
    firstrun = false;
  }

  const resState = state[cursor];
  const resSetter = setters[cursor];
  cursor++;

  return [resState, resSetter];
};
```

- useState는 초깃값을 받는다. 그리고 최초 실행일 경우 초깃값을 `state` 배열에 삽입한다. 마찬가지로 `setter`도 추가한다. 이때 `state` 배열과 `setters` 배열은 useState 함수 외부에 위치한다.
- 추가된 위치의 state와 setter를 반환하면, state 값은 useState 함수 외부에 위치하므로 closure가 적용되어 useState가 종료되더라도 값이 유지된다.

<br/>

## Hook 규칙

1. 함수형 컴포넌트의 최상위에서만 hook을 호출해야한다.
   - state는 컴포넌트의 실행 순서대로 배열에 저장된다. 따라서 반복문, 조건문 혹은 중첩함수에서 hook을 호출하면, 컴포넌트의 실행 순서가 달라질 수 있다.
2. 오직 React 함수 내에서 hook을 호출해야한다.
   - Hook을 일반적인 JS 함수에서 호출하면 안되고, 함수형 컴포넌트 또는 커스텀 훅 내에서만 호출할 수 있다.

<br/>

## useState 모듈 분석

```react
export function useState<S>(
  initialState: (() => S) | S,
): [S, Dispatch<BasicStateAction<S>>] {
  const dispatcher = resolveDispatcher();
  return dispatcher.useState(initialState);
}
```

```react
function resolveDispatcher() {
  const dispatcher = ReactCurrentDispatcher.current;

  if (__DEV__) {
    if (dispatcher === null) {
      console.error('Some error msg...');
    }
  }

  return ((dispatcher: any): Dispatcher);
}
```

```react
const ReactCurrentDispatcher = {
  /**
   * @internal
   * @type {ReactComponent}
   */
  current: (null: null | Dispatcher),
};
```

`ReactCurrentDispatcher`은 전역에 선언된 객체의 프로퍼티이다. useState 리턴 값의 출처가 전역에서 온다는 것이다.
리액트가 실제로 클로저를 활용해 함수 외부의 값에 접근하는 사실을 알 수 있다.

<br/>

## useState 배치 프로세스

다음 예제에서 `increase1`의 결과는 `1`이고 `increase2`의 결과는 `3`이다.

```js
const Counter = () => {
  const [count, setCount] = useState(0);

  const increase1 = () => {
    setCount(count + 1);
    setCount(count + 1);
    setCount(count + 1);
  }

  const increase2 = () => {
    setCount((count) => count + 1);
    setCount((count) => count + 1);
    setCount((count) => count + 1);
  }
}

export default Counter;
```

그 이유는  새로운 상태가 바로 이전의 상태를 통해 계산되어야하면 함수를 인자로 넣어야하기 때문인데, 리액트는 퍼포먼스 향상을 위해 특별한 배치 프로세스를 사용하기 때문이다. 여러 setState 업데이트를 한 번에 묶어서 처리한 후 마지막 값을 통해 state를 결정한다.

```js
{
  memoizedState: 0, // first hook
  baseState: 0,
  queue: { /* ... */ },
  baseUpdate: null,
  next: { // second hook
    memoizedState: false,
    baseState: false,
    queue: { /* ... */ },
    baseUpdate: null,
    next: { // third hook
      memoizedState: {
        tag: 192,
        create: () => {},
        destory: undefined,
        deps: [0, false],
        next: { /* ... */ }
      },
      baseState: null,
      queue: null,
      baseUpdate: null,
      next: null
    }
  }
}
```

- 실제 hook을 변수에 할당하여 출력했을 때 나타나는 결과
- next는 연결리스트의 일종으로 한 컴포넌트 안에서 여러번 실행되는 hook들을 연결해주는 역할

```js
{
  memoizedState: 0,
  baseState: 0,
  queue: {
   last: {
      expirationTime: 1073741823,
      suspenseConfig: null,
      action: 1, // setCount를 통해 설정한 값
      eagerReducer: basicStateReducer(state, action),
      eagerState: 1, // 상태 업데이트를 마치고 실제 렌더링되는 값
      next: { /* ... */ },
      priority: 98
    },
    dispatch: dispatchAction.bind(bull, currenctlyRenderingFiber$1, queue),
    lastRenderedReducer: basicStateReducer(state, action),
    lastRenderedState: 0,
  },
  baseUpdate: null,
  next: null
}
```

리액트의 `배치 프로세스`는 이렇게 묶인 hook들을 한 번에 처리한 뒤 last를 생성한다. 여기서 최종 반환될 `eagerState`를 계산하는 함수가 Reducer이다.

```js
function basicStateReducer(state, action) {
  return typeof action === 'function' ? action(state) : action;
}
```

이 Reducer에 넘기는 action 타입이 함수일 때 이전 상태를 인자로 받는다. 그래서 기존 상태를 기반으로 새로운 상태를 업데이트할 수 있게 된다.

<br/>

## 출처

https://thinkforthink.tistory.com/339
https://seokzin.tistory.com/entry/React-useState%EC%9D%98-%EB%8F%99%EC%9E%91-%EC%9B%90%EB%A6%AC%EC%99%80-%ED%81%B4%EB%A1%9C%EC%A0%80
https://hengxi.tistory.com/22
