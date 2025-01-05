컴포넌트가 마운트 되기 전에 실행되는 버전의 useEffect
> `useLayoutEffect`는 브라우저가 화면을 다시 그리는 것을 차단하기에 과도하게 사용하면 앱이 느려질 수 있다. 가급적 useEffect를 사용하는 것이 좋다


## 동작 방식

1. 컴포넌트 렌더링
2. `useLayoutEffect` 셋업 실행 -> state 업데이트 촉발
3. 업데이트 된 state로 컴포넌트 리렌더링
4. `3`에서 렌더링된 컴포넌트를 화면에 반영 (다시 `useLayoutEffect`를 실행하지는 않음) 


## Usage
### Measuring layout before the browser repaints the screen
툴팁과 같은 경우 화면 상의 위치와 크기에따라 노출되는 위치가 다를 수 있다. 이처럼 최종적으로 렌더링 될 컴포넌트가 레이아웃 정보를 알아야하고, 재조정되는 과정을 사용자에게 노출시키고 싶지 않을때 `useLayoutEffect`를 사용할 수 있다.

```js
function Tooltip() {  
	const ref = useRef(null);  
	const [tooltipHeight, setTooltipHeight] = useState(0); // You don't know real height yet  
	// 아직 실제 height 값을 모릅니다.  
	
	useLayoutEffect(() => {  
		const { height } = ref.current.getBoundingClientRect();  
		setTooltipHeight(height); // Re-render now that you know the real height  
		// 실제 높이를 알았으니 이제 리렌더링 합니다.  
	}, []);  
	
	// ...use tooltipHeight in the rendering logic below...  
	// ...아래에 작성될 렌더링 로직에 tooltipHeight를 사용합니다...  
}
```
1. `Tooltip`은 초기 `tooltipHeight = 0`으로 렌더링된다 (임의로 지정된 높이)
2. React는 이를 DOM에 배치하고 `useLayoutEffect`를 실행한다. 
3. `useLayoutEffect`는 툴팁의 높이를 측정하고 리렌더링을 촉발한다.
4. `Tooltip`이 실제 `tooltipHeight`으로 리렌더링된다.
5. React가 이를 DOM에 반영하면 브라우저에 툴팁이 표시된다.




### 실험

- `useLayoutEffect`에서 의존하고 있는 state를 업데이트할 경우, 화면에 반영이 끝나고 다시 `useLayoutEffect`가 실행된다. (화면 렌더링을 막지 않음)
```js
export default function App() {
	const [height, setHeight] = useState(0);
	
	useLayoutEffect(() => {
		setHeight(document.querySelector(".App").clientHeight);
		console.log(
			"Current rendered height is : " +
			document.querySelector(".height").innerHTML
		)
		console.log("Executed useLayoutEffect with : " + height);
	}, [height]);
	
	return (
	<div className="App">
		<h1>Hello CodeSandbox</h1>
		<h2>Start editing to see some magic happen!</h2>
		<span className="height">{height}</span>
	</div>
	);
}
```

```
Current rendered height is : 0
Executed useLayoutEffect with : 0
Current rendered height is : 128
Executed useLayoutEffect with : 128
```



## 의문

- 화면에 반영되지 전이면 useLayoutEffect에서 document.querySelector 로 접근해서 얻는 정보는 업데이트 전 정보인가?
	- 아래 실험은 동일하게 나옴
```js
useLayoutEffect(() => {
	console.log(document.querySelector(".count").innerHTML);
	console.log(countRef.current.innerHTML);
}, [count]);

return (
	<button
		ref={countRef}
		onClick={() => setCount((prev) => prev + 1)}
		className="count">		
		{count} + 1
	</button>
);
```


