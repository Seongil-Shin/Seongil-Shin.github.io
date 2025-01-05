https://ko.react.dev/learn/you-might-not-need-an-effect

Effect에서 외부 시스템과 상호작용하지 않을 경우 Effect가 필요없을 수 있다. 불필요한 Effect를 제거하면 코드를 더 쉽게 따라갈 수 있고, 실행 속도가 빨리지며, 오류 발생 가능성이 줄어든다.



## 불필요한 Effect들

1. 렌더링을 위한 데이터를 변환하기만 하는 effect
	- effect 내에서 단순히 상태를 바꾸고 있기만 한다면, 최초 렌더링 -> effect 실행 -> 리렌더링으로 불필요한 리렌더링이 발생하게 된다.
	- 따라서 컴포넌트 최상위 레벨에서 모든 데이터를 변환하는 방식으로 구현해야한다.
1. 사용자 이벤트를 핸들링하는 effect는 필요없을 수 있음
	- 버튼클릭, input 등은 effect가 필요없는 사용자 이벤트

외부 시스템과 동기화하기 위한 목적이 아니라면 effect는 필요없을 수 있다.

어떤 코드가 effect가 있어야하는지 이벤트 핸들러에 있어야하는지 헷갈릴 때는 코드가 실행되는 이유에 따라 결정하라
- 컴포넌트가 렌더링 되었기에 실행되어야하는 코드 -> effect
- 사용자가 인터랙션 했기때문에 실행되어야하는 코드 -> 이벤트핸들러


### 예시

#### props 변경시 모든 state 초기화

```js
export default function ProfilePage({ userId }) {  
	const [comment, setComment] = useState('');  
	
	useEffect(() => {  
		setComment('');  
	}, [userId]);  
}
```

	- `ProfilePage`는 최초 값으로 렌더링 후, effect를 실행하고 리렌더링을 수행함
	- 다음과 같이 `key`를 사용할 수 있다.

```js
export default function ProfilePage({ userId }) {  
	return (  
		<Profile  
			userId={userId}  
			key={userId}  
		/>  
	); 
}  

function Profile({ userId }) {  
	// ✅ 이 state 및 아래의 다른 state는 key 변경 시 자동으로 재설정됩니다.  
	const [comment, setComment] = useState('');  
	// ...  
}
```

#### prop 이 변경될 때 일부 state 업데이트하기

prop이 변경되었을때 전체 state를 초기화하기보단 일부 state를 업데이트하고 싶을 수 있다. effect를 사용한 코드는 다음과 같다.

```js
function List({ items }) {  
	const [isReverse, setIsReverse] = useState(false);  
	const [selection, setSelection] = useState(null);  
	
	// 🔴 피하세요: Effect에서 prop 변경 시 state 조정하기  
	useEffect(() => {  
		setSelection(null);  
	}, [items]);  
	// ...  
}
```

```js
function List({ items }) {  
	const [isReverse, setIsReverse] = useState(false);  
	const [selection, setSelection] = useState(null);  
		
	// 더 좋습니다: 렌더링 중 state 조정  
	const [prevItems, setPrevItems] = useState(items);  
	if (items !== prevItems) {  
		setPrevItems(items);  
		setSelection(null);  
	}  
	// ...  

}
```

이전 렌더링의 정보를 저장하는 건 비효율적이라 생각할 수 있지만, Effect에서 동일한 state를 업데이트하는 것보다는 낫다고 한다.
하지만 어디까지나 effect를 사용하는 것보다 낫다는 거지 베스트는 아니다. props나 다른 state에 따라 state를 조정하면 데이터 흐름을 이해하고 디버깅하는 것이 어렵기 때문이다. 대신 key를 사용하여 모든 state를 초기화하거나 state를 사용하지 않는 방법이 좋다.
```js
function List({ items }) {  
	const [isReverse, setIsReverse] = useState(false);  
	const [selectedId, setSelectedId] = useState(null);  
	
	// ✅ 최고예요: 렌더링 중에 모든 것을 계산  
	const selection = items.find(item => item.id === selectedId) ?? null;  
	// ...  
}
```
이제 state를 업데이트할 필요없이 selectedId가 items에 있으면 그대로 남고 아니면 null이 저장되게 된다.


#### 이벤트핸들러 간 로직 공유

이벤트 핸들러 간에 반복적인 코드를 줄이기 위해 effect를 사용하는 경우가 있다. 예를들어 구매 버튼을 클릭하거나 장바구니 버튼을 클릭했을 때 알람을 띄우는 코드가 같다고 해보자.
effect를 사용한 코드는 다음과 같다.

```js
function ProductPage({ product, addToCart }) {  
	// 🔴 피하세요: Effect 내부의 이벤트별 로직  
	useEffect(() => {  
		if (product.isInCart) {  
			showNotification(`Added ${product.name} to the shopping cart!`);  
		}  
	}, [product]);  
	
	function handleBuyClick() {  
		addToCart(product);  
	}  
	
	function handleCheckoutClick() {  
		addToCart(product);  
		navigateTo('/checkout');  
	}  
	// ...  
}
```

이때 effect는 불필요하고, 버그를 일으킬 수 있다. (새로고침 시 state를 저장한다면, 새로고침 후 `isInCart`가 `true`가 되어 바로 알람이 띄워질 수 있음)
따라서 다음과 같이 알림을 띄우는 코드를 함수로 빼서 effect 없이 다루는 것이 좋다.
```js
function ProductPage({ product, addToCart }) {  
	// ✅ 좋습니다: 이벤트 핸들러에서 이벤트별 로직이 호출됩니다.  
	function buyProduct() {  
		addToCart(product);  
		showNotification(`Added ${product.name} to the shopping cart!`);  
	}  

	function handleBuyClick() {  
		buyProduct();  
	}  
	
	function handleCheckoutClick() {  
		buyProduct(product);  
		navigateTo('/checkout');  
	}  
	// ...  
}
```


#### effect vs 이벤트핸들러 

어떤 코드가 effect가 있어야하는지 이벤트 핸들러에 있어야하는지 헷갈릴 때는 코드가 실행되는 이유에 따라 결정하라

```js
function Form() {  
	const [firstName, setFirstName] = useState('');  
	const [lastName, setLastName] = useState('');  
	
	// ✅ 좋습니다: 컴포넌트가 표시되었으므로 이 로직이 실행되어야 합니다.  
	useEffect(() => {  
		post('/analytics/event', { eventName: 'visit_form' });  
	}, []);  
	
	
	// 🔴 피하세요: Effect 내부의 이벤트별 로직  
	const [jsonToSubmit, setJsonToSubmit] = useState(null);  
	
	useEffect(() => {  
		if (jsonToSubmit !== null) {  
			post('/api/register', jsonToSubmit);  
		}  
	}, [jsonToSubmit]);  
	
	function handleSubmit(e) {  
		e.preventDefault();  
		setJsonToSubmit({ firstName, lastName });  
	}  
	// ...  
}
```

- analytics api 는 컴포넌트가 렌더링 되었기에 호출되어야하므로 effect에 두어야한다.
- register api는 사용자가 제출해야 실행하기에 이벤트 핸들러에 두어야한다.


#### 애플리케이션 초기화 

일부 로직은 앱이 로드될 때 한번만 실행되어야한다. 이때 다음과 같이 할 수 있다.
```js
function App() {  
	// 🔴 피하세요: 한 번만 실행되어야 하는 로직이 포함된 Effect  
	useEffect(() => {  
		loadDataFromLocalStorage();  
		checkAuthToken();  
	}, []);  
	// ...  
}
```

하지만 이 함수들은 개발 모드에서는 두번 실행된다. 함수가 한번만 실행되도록 설계되어있다면 이러한 방식은 이슈가 생길 수 있다. 일반적으로 컴포넌트는 다시 마운트될 때 탄력이 있어야하며 여기에는 최상위 App 컴포넌트도 포함된다.

이때는 최상위 변수를 추가하여 이미 실행되었는지 기록하는 변수를 추가할 수 있다.
```js
let didInit = false;  

function App() {  
	useEffect(() => {  
		if (!didInit) {  
			didInit = true;  
			// ✅ 앱 로드당 한 번만 실행  
			loadDataFromLocalStorage();  
			checkAuthToken();  
		}  
	}, []);  
	// ...  
}
```

또는 모듈 초기화 중이거나 앱이 렌더링 되기 전에 실행할 수 있다.

```js
if (typeof window !== 'undefined') { // 브라우저에서 실행 중인지 확인합니다.  
	// ✅ 앱 로드당 한 번만 실행  
	checkAuthToken();  
	loadDataFromLocalStorage();  
}  

function App() {  
	// ...  
}
```

컴포넌트를 import 할 때 최상위 레벨의 코드는 렌더링 되지 않더라도 한 번 실행된다. 


#### 부모에게 데이터 전달하기

자식 컴포넌트의 effect에서 부모 컴포넌트의 state를 업데이트하면 데이터 흐름을 추적하기 어려워진다.

```js
function Parent() {  
	const [data, setData] = useState(null);  
	// ...  
	return <Child onFetched={setData} />;  
}  

function Child({ onFetched }) {  
	const data = useSomeAPI();  
	// 🔴 피하세요: Effect에서 부모에게 데이터 전달하기  
	useEffect(() => {  
		if (data) {  
			onFetched(data);  
		}  
	}, [onFetched, data]);  
	// ...  
}
```
자식과 부모 모두 동일한 데이터가 필요하므로 부모 컴포넌트가 해당 데이터를 가져와서 자식에게 내려주는 것이 바람직하다.
```js
function Parent() {  
	const data = useSomeAPI();  
	// ...  
	// ✅ 좋습니다: 자식에서 데이터를 전달  
	return <Child data={data} />;  
}  

	function Child({ data }) {  
	// ...  
}
```


#### 외부 저장소 구독하기

서드파티 라이브러리나 브라우저 API에서 데이터를 가져올 수 있다. 이 작업은 effect 내에서 수행하는 경우가 많다.

```js
function useOnlineStatus() {  
	// 이상적이지 않습니다: Effect에서 저장소를 수동으로 구독  
	const [isOnline, setIsOnline] = useState(true);  
	useEffect(() => {  
		function updateState() {  
			setIsOnline(navigator.onLine);  
		}  
		  
		updateState();  
		  
		window.addEventListener('online', updateState);  
		window.addEventListener('offline', updateState);  
		
		return () => {  
			window.removeEventListener('online', updateState);  
			window.removeEventListener('offline', updateState);  
		};  
	}, []);  
	return isOnline;  
}  

function ChatIndicator() {  
	const isOnline = useOnlineStatus();  
	// ...  
}
```

하지만 React에는 외부 저장소를 구독하기 위해 특별히 제작된 Hook인 `useSyncExternalStore`가 있다.

```js
function subscribe(callback) {  
	window.addEventListener('online', callback);  
	window.addEventListener('offline', callback);  
	
	return () => {  
		window.removeEventListener('online', callback);  
		window.removeEventListener('offline', callback);  
	};  
}  

function useOnlineStatus() {  
	// ✅ 좋습니다: 내장 Hook으로 외부 스토어 구독하기  
	return useSyncExternalStore(  
		subscribe, // 동일한 함수를 전달하는 한 React는 다시 구독하지 않습니다.  
		() => navigator.onLine, // 클라이언트에서 값을 얻는 방법  
		() => true // 서버에서 값을 얻는 방법  
	);  
}  

function ChatIndicator() {  
	const isOnline = useOnlineStatus();  
	// ...  
}
```
