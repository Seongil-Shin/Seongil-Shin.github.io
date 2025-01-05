https://react-ko.dev/reference/react/useReducer

```js
import { useReducer } from 'react';  

function reducer(state, action) {  
	// ...  
}  

function createInitial(initialArg) {
	// ...
}

function MyComponent() {  
	const [state, dispatch] = useReducer(reducer, { age: 42 });  
	const [state2, dispatch2] = useReducer(reducer, { age: 42}, createInitial);
	// ...
```

### parameters

- `reducer` : 현재 state, action 객체를 받는 reducer 함수
- `initialArg` : 초기값
- (optional) `init` : 초기 state 계산 방법을 지정하는 초기화 함수. `init(initialArg)` 로 실행된다.
	- 다음과 같이 하면 초기값 반영은 최초 렌더링에만 되지만, 함수 호출 자체는 매 렌더링마다 되어 비싼 계산을 수행하는 경우 성능 낭비가 된다.
	  `useReducer(reducer, createInitialState(initialArg))`
	- 다음과 같이 초기화 함수를 사용하면 매 렌더링마다 함수호출이 일어나지 않는다. 인자가 필요없을 경우엔 `null`을 넣으면 된다.
	  `useReducer(reducer, initialArg, createInitialState)` 