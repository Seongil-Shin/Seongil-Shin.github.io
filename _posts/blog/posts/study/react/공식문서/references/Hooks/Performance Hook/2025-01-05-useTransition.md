https://react-ko.dev/reference/react/useTransition

UI를 차단하지 않고 state를 업데이트할 수 있는 React 훅
```js
const [isPending, startTransition] = useTransition();
```

### Parameter
- 매개변수를 받지 않음

### Returns
- 값이 두개 들어있는 배열 반환
	- `isPending` : 진행 중인 트랜지션이 있는지 여부를 알려주는 플래그
	- `startTransition` : state 업데이트를 트랜지션으로 표시할 수 있는 함수


### `startTransition` 함수

`startTransition` 안에서 상태 업데이트를 수행하면 state 업데이트를 transition으로 mark 할 수 있다.
```js
function selectTab(nextTab) {  
	startTransition(() => {  
		setTab(nextTab);  
	});  
}
```

- Parameter
	- `scope`
		- 하나 이상의 set 함수를 호출해서 일부 state를 업데이트하는 함수
		- React는 매개변수 없이 scope를 즉시 호출하고, 호출 중에 동기적으로 예약된 state 업데이트를 트랜지션으로 mark 한다. 
			- 직접적으로 set 함수를 호출하지 않고, 다른 함수을 호출하는데 그 함수 안에서 set 함수를 호출하는 등 간접적인 경우에도 트랜지션으로 mark 된다.
		- 이 과정은 논블로킹이고, 원치않는 로딩을 표시하지 않는다.

### Caveats
- `useTransiton`은 훅이므로 컴포넌트나 커스텀 훅 내부에서만 호출 가능하다. 다른 곳에서 호출하고 싶으면 독립형 [startTransition](obsidian://open?vault=Obsidian%20Vault&file=react%2Freferences%2FAPIs%2FstartTransition)을 사용하라
- 해당 state의 set 함수에 접근 가능할 때만 트랜지션으로 감쌀 수 있다. prop이나 커스텀 훅 값에 대한 응답으로 트랜지션을 시작하려면 [useDeferredValue](obsidian://open?vault=Obsidian%20Vault&file=react%2Freferences%2FHooks%2FPerformance%20Hook%2FuseDeferredValue)를 사용하라
- `startTransition`에 전달되는 함수는 동기적이어야한다.
- **트랜지션으로 표시된 state 업데이트는 다른 state 업데이트에 의해 중단된다.**
	- ex) 트랜지션 내에서 차트 컴포넌트를 업데이트한 다음, 차트가 다시 렌더링되는 도중에 입력을 시작하면 React는 입력 업데이트를 처리한 후 차트 컴포넌트에서 렌더링 작업을 다시 시작한다
- 트랜지션 업데이트로 텍스트 입력을 제어하는데 사용할 수 없다
- 진행중인 트랜지션이 여러 개일 경우, React는 현재 트랜지션을 함께 일괄 처리한다. (이는 이후 릴리즈에서 제거될 수도 있다)

## Usage
### Marking a state update as a non-bloking transition
- 상태 업데이트를 트랜지션으로 mark 하면 리렌더링 도중에도 UI가 반응성을 유지한다
	- ex) 사용자가 탭을 클릭했다가 다시 바꿀 경우 첫번재 리렌더링이 완료될 때까지 기다리지 않고도 다른 탭을 클릭할 수 있다

### Updating the parent component in a transition
다음과 같이 prop으로 넘어온 함수를 `startTransition`에서 호출해도 `onClick` 내부의 상태 업데이트가 트랜지션으로 mark 된다
```js
export default function TabButton({ children, isActive, onClick }) {
  const [isPending, startTransition] = useTransition();
  
  return (
    <button onClick={() => {
      startTransition(() => {
        onClick();
      });
    }}/>
  )
}
```

### Displaying a pending visual state during the transition

`isPending`은 진행 중인 트랜지션이 있을때 true가 되므로, 다음과 같이 트랜지션이 진행되고 있음을 사용자에게 알릴 수 있다
```js
export default function TabButton({ children, isActive, onClick }) {
  const [isPending, startTransition] = useTransition();
  
  if (isPending) {
    return <b className="pending">{children}</b>;
  }
  ...
```

### Preventing unwanted loading indicators
`Suspense`를 사용하면 컴포넌트 중단 시 가장 가까운 로딩 폴백이 나타난다. 하지만 이러한 폴백을 막고 싶을 때가 있는데, 이럴때 `startTransition`을 사용하면 폴백을 안띄울 수 있다.

### Building a Suspense-enabled router
React 프레임워크나 라우터를 구축하는 경우 페이지 네이게이션을 트랜지션으로 mark 하는 것이 좋다
- 트랜지션을 중단가능하기에 사용자는 다시 렌더링이 완료될 때까지 기다리지 않고 바로 클릭할 수 있다
- 트랜지션은 원치않는 fallback을 방지하여 사용자가 네이게이션 시 갑자기 이동하는 것을 방지할 수 있다

```js
  function navigate(url) {
    startTransition(() => {
      setPage(url);
    });
  }
```



## Troubleshooting

### Updating an input in a transition doesn't work
input을 제어하는 state 변수에는 트랜지션을 사용할 수 없는데, 트랜지션은 논블로킹이지만, 변경 이벤트에 대한 응답으로 input을 업데이트하는 것은 동기적으로 이루어져야하기 때문이다.
```js
function handleChange(e) {  
	// ❌ Can't use transitions for controlled input state  
	startTransition(() => {  
		setText(e.target.value);  
	});  
}
```
트랜지션을 사용하고 싶으면 다음 두가지 방법이 있다
- input state와 트랜지션 실행시 업데이트할 state 변수를 각각 선언할 수 있다.
- `useDeferredValue`를 사용할 수 있다.

### startTransition에 전달되는 함수는 동기적이어야한다.

다음은 안되지만
```js
startTransition(() => {  
	// ❌ Setting state *after* startTransition call  
	setTimeout(() => {  
		setPage('/about');  
	}, 1000);  
});
```

다음은 된다
```js
setTimeout(() => {  
	startTransition(() => {  
		// ✅ Setting state *during* startTransition call  
		setPage('/about');  
	});  
}, 1000);
```