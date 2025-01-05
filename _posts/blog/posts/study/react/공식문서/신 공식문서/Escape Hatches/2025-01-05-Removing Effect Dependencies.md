https://react-ko.dev/learn/removing-effect-dependencies

Effect에서 linter는 Effect의 의존성 목록에 Effect가 사용하는 모든 반응형 값을 포함했는지 확인한다. 이때 linter를 무시할 수 있지만 지키는 것이 좋다. 따라서 무시하기보단 불필요한 의존성을 없애는 것이 좋다.
불필요한 의존성으로 인해 Effect가 너무 자주 실행되거나 무한 루프에 빠질수도 있다.

## Dependencies should match the code

Effect는 [반응형 값](obsidian://open?vault=Obsidian%20Vault&file=react%2F%EC%8B%A0%20%EA%B3%B5%EC%8B%9D%EB%AC%B8%EC%84%9C%2FEscape%20Hatches%2FSeparating%20Events%20from%20Effects)(리렌더링 시 변경될 수 있는 값.)에 반응한다. 따라서 linter는 Effect에서 사용하는 반응형 값이 모두 의존성 목록에 포함되어있는지 확인한다. 만약 Effect에서 사용하는 값을 linter에 포함하지 않으면 linter는 주의를 줄 것이고 그것이 맞다. 

의존성을 제거하려면, Effect 코드 또는 반응형 값 선언 방식을 변경 후, 변경한 코드에 맞게 의존성을 조정해야한다.

- 이전
```js
const serverUrl = 'https://localhost:1234';  

function ChatRoom({ roomId }) {  
	useEffect(() => {  
		const connection = createConnection(serverUrl, roomId);  
		connection.connect();  
		return () => connection.disconnect();  
	}, []); // 🔴 React Hook useEffect has a missing dependency: 'roomId'  
	// ...  
}
```
- 이후
```js
const serverUrl = 'https://localhost:1234';    
const roomId = 'music'; // Not a reactive value anymore

function ChatRoom() {  
	useEffect(() => {  
		const connection = createConnection(serverUrl, roomId);  
		connection.connect();  
		return () => connection.disconnect();    
	}, []); // ✅ All dependencies declared
	// ...  
}
```

## Removing unnecessary dependencies

다음과 같은 질문을 Effect에서 던져보자
- 이벤트핸들러에 있어야하는 코드이지 않을까? [[Separating Events from Effects]]
- Effect 내에서 서로 관련없는 일을 하고 있는가? [관련링크](obsidian://open?vault=Obsidian%20Vault&file=react%2F%EC%8B%A0%20%EA%B3%B5%EC%8B%9D%EB%AC%B8%EC%84%9C%2FEscape%20Hatches%2FLifecycle%20of%20Reactive%20Effects)
	- 서로 독립적인 작업은 별도의 Effect에서 진행하여야한다.
- 다음 state를 만들기 위해, 현재 state를 읽고 있는가?
	- 현재 state를 읽지 말고 업데이터 함수를 사용하라 [링크](https://react-ko.dev/learn/queueing-a-series-of-state-updates)
- 값의 변경에 반응하지 않고 값을 읽고 싶은가?
	- 실험적인 `useEffectEvent`를 사용할 수 있다.
- 객체, 배열, 함수로인해 의도치않게 Effect가 자주 실행되고 있는가?
	- 컴포넌트 내에서 선언한 객체는 매렌더링마다 다시 생성되기에 항상 다른 객체로 인식된다.
	- 따라서 다음과 같은 방법을 채용할 수 있다.
		- 컴포넌트 외부 옮기기 
		- Effect 내부로 옮기기
		- 원시값 추출하기
		- useMemo, useCallback 사용
