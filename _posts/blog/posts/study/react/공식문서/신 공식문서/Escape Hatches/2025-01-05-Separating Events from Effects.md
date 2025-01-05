https://react-ko.dev/learn/separating-events-from-effects

이벤트핸들러는 같은 상호작용을 다시 수행할때만 다시 실행된다.
Effect는 props 또는 state의 변경 시 동기화를 위해 실행된다. 때때로 일부 값에 대한 응답으로 다시 실행되는 Effect와 그렇지 않은 Effect의 혼합이 필요할 때가 있다.
이 페이지는 이를 어떻게 수행할 수 있는지에 대한 글이다


## Choosing between event handlers and Effects

어떤 동작이 이벤트 핸들러에 있어야하는지 effect에 있어야하는지 헷갈릴때가 있다. 이때는 코드가 실행되어야하는 이유에 대해 생각해보자

이벤트 핸들러
- 특정 상호작용에 대한 응답으로 실행됨.(유저가 클릭했을때, 입력했을때)
- 상호작용 없이 동작이 이루어지면 안됨.

Effect
- state, prop 등 값이 변경되었을때 실행함.
- 상호작용이 없어도 값이 변경되면 재실행해야함.


## Reactive values and reactive logic

직관적으로 봤을때 이벤트 핸들러는 수동으로 실행되고, Effect는 자동으로 실행된다는 인식을 가질 수 있다. 좀 더 정확한 표현은 이벤트 핸들러는 `반응형 로직`이 아니고, Effect는 `반응형 로직`이라는 것이다.

반응형 값 (Reactive values)
- 컴포넌트 본문 내부에 선언된 props, state, 변수
- 렌더링 데이터 흐름에 참여하며, 리렌더링으로 인해 변경될 수 있다.

이벤트 핸들러 (Non-reactive logic)
- 반응형 값이 변경되었다고 다시 실행되지 않아야한다.

Effect (Reactive logic)
- 의존하는 반응형 값이 변경되면 다시 실행되어야한다.


## Extracting non-reactive logic out of Effects

반응형 로직과 비반응형으로 동작해야하는 로직이 함께 Effect에 있는 경우가 있다.

예제 : 사용자가 채팅에 연결할때 알림을 표시하는데, props에서 현재 테마를 읽어 올바른 색상으로 알림을 표시.

```js
function ChatRoom({ roomId, theme }) {  
	useEffect(() => {  
		const connection = createConnection(serverUrl, roomId);  
		connection.on('connected', () => {  
			showNotification('Connected!', theme);  
		});  
		connection.connect();  
		
		return () => {  
			connection.disconnect()  
		};  
	}, [roomId, theme]); // ✅ All dependencies declared

```

위 예제에서 `theme`는 반응형 값이기에 의존성에 추가했다. 하지만 이러면 `theme`가 변경될때마다 재연결이 이루어진다.

이 문제를 해결하는 가장 쉬운 방법은 `theme`를 의존성배열에서 제거하는 것이지만, 리액트에서 실험중인 `useEffectEvent`를 사용할 수 있다.

## Declaring an Effect Event

[useEffectEvent](https://react-ko.dev/reference/react/experimental_useEffectEvent)라는 실험적인 API로 위 문제를 해결할 수 있다.

```js
function ChatRoom({ roomId, theme }) {    
	const onConnected = useEffectEvent(() => {  
		showNotification('Connected!', theme);  
	});
	
	useEffect(() => {  
		const connection = createConnection(serverUrl, roomId);  
		connection.on('connected', () => {  
			onConnected();  
		});  
		connection.connect();  
		
		return () => {  
			connection.disconnect()  
		};  
	}, [roomId]); // ✅ All dependencies declared

```

Effect Event는 반응형 이벤트가 아니기에 의존성에서 생략해도 된다. 

이벤트 핸들러 vs Effect Event
- 이벤트 핸들러 : 사용자 상호작용에 대한 응답으로 실행
- Effect Event : Effect에서 사용자가 촉발하는 별도의 이벤트

Effect Event는 사용하는 반응형 값이 변경시 자동으로 새로운 값으로 반영한다. 하지만 Effect를 다시 실행시키지 않는다.

## Limitations of Effect Events

- Effect 내부에서만 호출할 수 있음
- 다른 컴포넌트나 Hook에 전달하면 안됨.

Effect Event는 Effect 코드의 비반응형 조각이기에, Effect Event는 이를 사용하는 Effect 옆에 있어야한다.