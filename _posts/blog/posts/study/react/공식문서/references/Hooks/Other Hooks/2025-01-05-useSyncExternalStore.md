https://react-ko.dev/reference/react/useSyncExternalStore#usesyncexternalstore
외부 스토어를 구독할 수 있는 React 훅

```js
const snapshot = useSyncExternalStore(subscribe, getSnapshot, getServerSnapshot?) 
```


### Parameters
- `subscribe`
	- 하나의 callback 인수를 받아 스토어를 구독하는 함수. 스토어가 변경되면 인수로 넘어온 callback을 호출해야함. 이로써 `getSnapshot`을 실행시키도록 한다.
	- `subscribe`는 구독을 해제하는 함수를 반환해야한다.
- `getSnapshot`
	- 컴포넌트에 필요한 스토어 데이터의 스냅샷을 반환하는 함수
	- 스토어가 변경되지 않았으면 같은 값을 반환하고, 변경되어 다른 값을 반환하면(`Object.is`) React는 컴포넌트를 리렌더링한다.
- `getServerSnapshot`
	- 스토어에 있는 데이터의 초기 스냅샷을 반환하는 함수
	- `서버에서 렌더링할 때`와 `클라이언트에서 hydrate할 떄`만 호출된다.
	- 서버 스냅샷은 클라이언트와 서버간에 동일해야하며 일반적으로 서버에서 serialize하여 클라이언트로 전달한다.
	- 이 함수가 없으면 서버에서 컴포넌트를 렌더링할 때 오류가 발생한다.

### Returns
- 스토어의 현재 스냅샷

### Caveats
- `getSnapshot`이 반환하는 스토어 스냅샷은 불변해야한다. 데이터에 변이 발생 시, 새로운 불변 스냅샷을 반환하도록 해야한다.
- 리렌더리시에 다른 `subscribe` 함수가 전달되면 React는 새로 전달된 `subscribe` 함수를 사용하여 스토어를 구독한다. 컴포넌트 외부에서 `subscribe`를 선언하거나 `useCallback`을 사용하여 이를 방지할 수 있다


## Usage
### Subscribing to an external store
보통 컴포넌트는 props, state, context를 읽지만 외부 저장소에서 데이터를 읽어야하는 경우도 있다
- React 외부에 state를 보관중인 서드파티 라이브러리
- 변이 가능한 값을 노출하는 브라우저 API와 그 변이사항을 구독하는 이벤트

```js
// App.js
export default function TodosApp() {
  const todos = useSyncExternalStore(todosStore.subscribe, todosStore.getSnapshot);
  return (
    <>
      <button onClick={() => todosStore.addTodo()}>Add todo</button>
      <hr />
      <ul>
        {todos.map(todo => (
          <li key={todo.id}>{todo.text}</li>
        ))}
      </ul>
    </>
  );
}
```

```js
// todoStore.js

let nextId = 0;
let todos = [{ id: nextId++, text: 'Todo #1' }];
let listeners = [];

export const todosStore = {
  addTodo() {
    todos = [...todos, { id: nextId++, text: 'Todo #' + nextId }]
    emitChange();
  },
  subscribe(listener) {
    listeners = [...listeners, listener];
    return () => {
      listeners = listeners.filter(l => l !== listener);
    };
  },
  getSnapshot() {
    return todos;
  }
};

function emitChange() {
  for (let listener of listeners) {
    listener();
  }
}
```

1. `subscribe` 함수에서 `listener`를 받아 저장한다.
2. `button`을 눌러 외부 저장소의 값을 업데이트한다.
3. 외부저장소 업데이트 함수(`addTodo`)는 구독중인 모든 listener를 호출하여 `getSnapshot`을 호출하도록 한다.
4. `getSnapshot`을 통해 새로운 값을 받아, `Object.is`로 비교하여 값이 달라졌으면 업데이트한다.


### Subscribing to a browser API
```js
import { useSyncExternalStore } from 'react';

export function useOnlineStatus() {
  const isOnline = useSyncExternalStore(subscribe, getSnapshot);
  return isOnline;
}

function getSnapshot() {
  return navigator.onLine;
}

function subscribe(callback) {
  window.addEventListener('online', callback);
  window.addEventListener('offline', callback);
  return () => {
    window.removeEventListener('online', callback);
    window.removeEventListener('offline', callback);
  };
}
```


### Adding support for server rendering
세번쨰 인자로 서버사이드에서 최초렌더링 시 클라이언트와 싱크를 맞추기 위한 함수를 전달할 수 있다.
```js
export function useOnlineStatus() {  
	const isOnline = useSyncExternalStore(subscribe, getSnapshot, getServerSnapshot);  
	return isOnline;  
}  

function getSnapshot() {  
	return navigator.onLine;  
}  

function getServerSnapshot() {  
	return true; // Always show "Online" for server-generated HTML  
}  

function subscribe(callback) {  
	// ...  
}
```