
```js
const value = useContext(SomeContext)
```



### Parameters
- `SomeContext`
	- `createContext`로 생성된 어떠한 context

### Returns
- 가장 가까운 `SomeContext.Provider`에 전달된 `value`값을 반환
- `Provider`가 없는 경우 `defaultValue`가 된다. 
- 반환되는 값은 항상 최신이며, context가 변경되면 context를 읽는 컴포넌트를 자동으로 리렌더링한다.



## Usage

### 객체 및 함수 전달 시 리렌더링 최적화

context로 객체 및 함수를 포함한 모든 값을 전달할 수 있는데, 이때 객체 및 함수는 `MyApp`이 리렌더링 될때마다 다시 생성된다. 따라서 `useContext`로 이 context를 가져오는 컴포넌트들도 실제로 값이 변경되지 않았어도 리렌더링된다.

```js
function MyApp() {  
	const [currentUser, setCurrentUser] = useState(null);  
	
	function login(response) {  
		storeCredentials(response.credentials);  
		setCurrentUser(response.user);  
	}  
	
	return (  
		<AuthContext.Provider value={{ currentUser, login }}>  
			<Page />  
		</AuthContext.Provider>  
	);  
}
```

이때는 다음과 같이 `useMemo` 또는 `useCallback`을 사용할 수 있다.

```js
function MyApp() {  
	const [currentUser, setCurrentUser] = useState(null);  
		  
	const login = useCallback((response) => {  
		storeCredentials(response.credentials);  
		setCurrentUser(response.user);  
	}, []);  
	
	const contextValue = useMemo(() => ({  
		currentUser,  
		login  
	}), [currentUser, login]);
	
	return (  
		<AuthContext.Provider value={contextValue}>  
			<Page />  
		</AuthContext.Provider>  
	);  
}
```