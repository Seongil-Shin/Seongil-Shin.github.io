https://ko.react.dev/reference/react/startTransition

UI를 차단하지 않고 state를 업데이트할 수 있게해줌
```jsx
import { startTransition } from 'react';  

function TabContainer() {  
	const [tab, setTab] = useState('about');  
	
	function selectTab(nextTab) {  
		startTransition(() => {  
			setTab(nextTab);  
		});  
	}  
	// ...  
}
```

### Caveats
- [useTransition](obsidian://open?vault=Obsidian%20Vault&file=react%2Freferences%2FHooks%2FPerformance%20Hook%2FuseTransition)에서 `startTransition`만 가져온 것으로, pending 인지 추적하는 방법을 제공하지 않는다.
- `set` 함수에 접근할 수 있는 경우에만 업데이트를 트랜지션으로 감쌀 수 있고, 일부 prop이나 커스텀 훅 값에 대한 응답으로 트랜지션을 시작하려면 `useDeferredValue`를 사용해야한다
- 전달하는 함수는 동기식이어야하고 React는 이를 바로 실행한다. 실행동안 발생한 모든 업데이트를 트랜지션으로 표시한다.
- 트랜지션 업데이트는 텍스트 입력을 제어하는데 사용할 수 없다.
- 