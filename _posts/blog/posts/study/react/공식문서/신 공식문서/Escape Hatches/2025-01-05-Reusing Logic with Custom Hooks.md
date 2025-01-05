https://react-ko.dev/learn/reusing-logic-with-custom-hooks

## Custom Hooks : Sharing logic between components

여러 컴포넌트에서 반복되는 로직을 custom hook으로 뽑을 수 있다. 


## Hook names always start with `use`

1. React 컴포넌트 이름은 `StatusBar` 와 같이 항상 대문자로 시작해야한다. 
2. hook의 이름은 `use`로 시작하고 그 다음 첫글자는 대문자여야한다. 또한 hook은 임의의 값을 반환할 수 있다.

linter가 react 용으로 설정되어있다면, 이 명명 규칙을 사용한다. 


## Custom Hooks let you share stateful logic, not state itself

커스텀 훅에서 state를 반환한다고 그 커스텀 훅을 사용하는 컴포넌트들이 모두 같은 값을 공유하는 것은 아니다.
마치 `useState`가 그러한 것처럼 아래 두 컴포넌트는 서로 각자의 state를 가지고 있다.
```js
function StatusBar() {  
	const isOnline = useOnlineStatus();  
	// ...  
}  

function SaveButton() {  
	const isOnline = useOnlineStatus();  
	// ...  
}
```

## Passing reactive values between Hooks

컴포넌트를 다시 렌더링할때마다 커스텀 훅 내부의 코드가 다시 실행된다. 따라서 커스텀 훅은 순수해야한다.
커스텀 훅은 컴포넌트와 함께 리렌더링되기에 항상 최신 props와 state를 받는다.

 ```js
export function useChatRoom({ serverUrl, roomId }) {  
	useEffect(() => {  
		const options = {  
			serverUrl: serverUrl,  
			roomId: roomId  
		};  
		
		const connection = createConnection(options);  
		connection.connect();  
		connection.on('message', (msg) => {  
			showNotification('New message: ' + msg);  
		});  
		
		return () => connection.disconnect();  
	}, [roomId, serverUrl]);  
}
```

```js
export default function ChatRoom({ roomId }) {  
	const [serverUrl, setServerUrl] = useState('https://localhost:1234');  

	useChatRoom({  
		roomId: roomId,  
		serverUrl: serverUrl  
	});  
	
	return (  
		<>  
		<label>  
			Server URL:  
			<input value={serverUrl} onChange={e => setServerUrl(e.target.value)} />  
		</label>  
		<h1>Welcome to the {roomId} room!</h1>  
		</>  
	);  
}
```


## Passing event handlers to custom Hooks

다음과 같이 커스텀 훅 내부에서 사용할 함수를 외부에서 받으면 좀 더 다양한 케이스에 대응이 가능하다.
이때 `useEffectEvent`를 사용하여 의존성을 줄일 수 있다.

```js
export function useChatRoom({ serverUrl, roomId, onReceiveMessage }) {    
	const onMessage = useEffectEvent(onReceiveMessage);
	
	useEffect(() => {  
		const options = {  
			serverUrl: serverUrl,  
			roomId: roomId  
		};  
		
		const connection = createConnection(options);  
		connection.connect();  
		connection.on('message', (msg) => {  
			onMessage('New message: ' + msg);  
		});  
		
		return () => connection.disconnect();  
	}, [roomId, serverUrl]);  
}

```


## When to use custom Hooks

모든 중복에 커스텀 훅이 필요하지는 않다. 약간의 중복은 괜찮다. 하지만 Effect를 사용할 때는 항상 커스텀 훅으로 뽑아내는게 더 명확하지 않을지 생각하라.

장점
- Effect는 외부 시스템과의 동기화이므로 Effect를 커스텀 훅으로 감싸면 의도와 데이터 흐름방식을 명확하게 전달할 수 있다.
- 다른 개발자로 하여금 불필요한 의존성을 추가하는 것을 막을 수 있다. 이상적으로는 앱의 Effect 대부분이 커스텀 훅에 포함될 것이다.
- React에 새로운 기능이 추가될 때 컴포넌트를 변경하지 않고 Effect를 제거할 수 있다.

## 커스텀 훅은 구체적인 고수준 사용 사례에 집중하라

`useEffect` API 자체에 대한 대안 및 편의 래퍼 역할을 하는 커스텀 생명주기 훅을 생성하거나 사용하지마라
- useMount(fn)
```js
// 🔴 Avoid: creating custom "lifecycle" Hooks  
function useMount(fn) {  
	useEffect(() => {  
		fn();  
	}, []); // 🔴 React Hook useEffect has a missing dependency: 'fn'  
}
```
- useEffectOnce(fn)
- useUpdateEffect(fn)

이와같은 커스텀 생명주기 훅은 React 패러다임에 맞지 않는다. linter는 `useEffect` 호출만 확인하기에 `useMount` 내부의 의존성 누락을 경고하지 않는다.

차라리 직접 `useEffect`를 사용하면 좀 더 고수준 커스텀 훅을 추출할 수 있다. (꼭 추출하지 않아도 된다.)

좋은 커스텀 훅은 호출 코드가 수행하는 작업을 제한하여 보다 선언적으로 만든다. 커스텀 훅 API의 사용사례를 제한하지 않고 추상적으로 만들경우 장기적으로 더 많은 문제를 야기한다.
- `useChatRoom(options)` : 채팅방에만 연결할 수 있다.
- `useImpressionLog(eventName, extraData)` :  애널리틱스 노출 로그만 전송가능

## Custom Hooks help you migrate to better patterns

리액트는 계속 업데이트 되고 있고, 리액트팀의 목표는 더 나은 솔루션을 제공하여 Effect의 수를 줄이는 것이다. 이때 Effect가 커스텀 훅으로 감싸져있다면 이러한 업데이트에 더 쉽게 대응할 수 있다.
