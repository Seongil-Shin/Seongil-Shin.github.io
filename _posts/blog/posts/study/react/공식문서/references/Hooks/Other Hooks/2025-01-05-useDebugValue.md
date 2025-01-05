https://react-ko.dev/reference/react/useDebugValue
React 개발자 도구에서 커스텀 훅에 레이블을 추가해주는 React 훅

```js
useDebugValue(value, format?)
```

### Parameter
- `value` : 개발자 도구에서 표시하려는 값
- `format` (optional) 
	- 포매팅 함수. `value`로 포매팅 함수를 호출한 다음 반환된 포매팅 값을 표시한다.
	- 없으면 `value`가 그대로 표시됨.

## Usage

### Adding a lavel to a custom Hook
커스텀 훅에서 `useDebugValue`를 사용하면 레이블을 추가하여 콘솔에 보여준다.
```js
import { useDebugValue } from 'react';  

function useOnlineStatus() {  
	// ...  
	useDebugValue(isOnline ? 'Online' : 'Offline');  
	// ...  
}
```

```
OnlineStatus: "Online"
OnlineStatus: "Offline"
```
