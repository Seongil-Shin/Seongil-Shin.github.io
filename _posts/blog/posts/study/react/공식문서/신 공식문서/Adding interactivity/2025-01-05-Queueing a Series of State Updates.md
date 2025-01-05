https://react.dev/learn/queueing-a-series-of-state-updates

### 요약 🔍  
React는 상태 업데이트를 배치 처리하여 여러 상태 업데이트를 효율적으로 처리하며, 상태 업데이트를 여러 번 연속으로 수행하는 방법에 대해 배웁니다.  
  
### 사실 📚  
- React는 상태 업데이트를 배치 처리하여 여러 업데이트를 효율적으로 처리합니다.  
- 이로 인해 이벤트 핸들러의 코드가 모두 실행된 후에 상태 업데이트가 처리된다.
- 배치 처리는 관련된 상태 업데이트를 모아 한번에 업데이트 하기에 리렌더링 횟수를 줄인다.
- React는 의도적인 이벤트(ex : 클릭)들 간에는 배치처리 하지않는다. 따라서 버튼을 클릭하여 버튼을 disable 했을 경우, 두번째 클릭은 버튼 클릭 이벤트를 발생시키지 않는다.
  
### 동일한 상태를 여러 번 업데이트하는 경우  
- 다음 렌더링 전에 동일한 상태 변수를 여러 번 업데이트하려는 경우, `setNumber(n => n + 1)`과 같은 함수를 사용할 수 있습니다.  
- `n => n + 1`와 같은 함수를 updater function 이라고 한다.
- 동작방식
	1. 이벤트 핸들러 내 모든 코드가 실행될 때까지 `setState` 처리를 queuing 함
	2. 리렌더링 동안 리액트는 그 queue를 수행하고, 최종 업데이트 된 상태를 계산함.
- ex)
```js
setNumber(n => n + 1);  
setNumber(n => n + 1);  
setNumber(n => n + 1);
```
	- 리액트는 각 `n => n + 1`을 queue에 추가함.
	- 이후 queue에 든 것들을 하나씩 수행함. 
	- 첫번쨰 updater function이 받는 n은 이벤트 핸들러 수행시의 state 값. 
	- 이후 updater funciton 들은 이전 updater function에서 업데이트 된 state를 n으로 받는다.

### 상태 대체 후 업데이트하는 경우  
- 다음과 같은 경우에서도 배치처리는 이루어지며, 리액트는 각 `setState`를 queue에 담아 순차적으로 처리한다.
```js
setNumber(number + 5);
setNumber(n => n + 1);

// 최종결과 : 6  (number 최초값 0일떄) 
```
```js
setNumber(number + 5);  
setNumber(n => n + 1);  
setNumber(42);

// 최종결과 : 42
```
- `setState(5)` 은 `setState(n => 5)`와 같다.


### naming conventions
- 다음과 같이 연관된 상태의 첫번째 문자로 updater function의 argument 이름을 짓거나 풀네임을 쓰는 것이 흔하다.
```js
setEnabled(e => !e);
setEnabled(enabled => !enabled);
setEnabled(prevEnabled => !prevEnabled);
```