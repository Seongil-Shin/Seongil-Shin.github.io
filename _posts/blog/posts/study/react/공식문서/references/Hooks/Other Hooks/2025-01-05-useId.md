https://react-ko.dev/reference/react/useId
accessibility 속성에 전달할 수 있는 고유 ID를 생성하기 위한 훅
```js
const id = useId()
```


### Returns
- 특정 컴포넌트 내 특정 `useId`와 관련된 고유 ID 문자열을 반환한다. 이 값은 리렌더링되어도 항상 동일하다.


### Caveats
- `key`를 생성 하기위해 사용하면 안된다. `key`는 데이터에서 생성되어야한다.
- 서버 렌더링에서 서버와 클라이언트에서 렌더링한 트리가 정확히 일치하지 않으면 생성된 ID가 일치하지 않는다.

### 카운터 vs useId
전역 변수로 `nextId++`하는 것보다 왜 나은가?
- `useId`는 서버렌더링과 함께 작동한다. 서버와 클라이언트에서 렌더링한 트리가 정확히 일치하면, 같은 컴포넌트에 같은 id 값을 전달한다.
- 클라이언트 컴포넌트가 hydration 되는 순서는 서버와 다르기에 카운터로는 이를 보장하기가 어렵다. 


## Usage
### Generating unique IDs for accessibility attributes
```js
import { useId } from 'react';  

function PasswordField() {  
	const passwordHintId = useId();
	//...
	
	return (
		<>  
			<input type="password" aria-describedby={passwordHintId} />  
			<p id={passwordHintId}>  
		</>
	)
}
```
[aria-describedBy](https://developer.mozilla.org/en-US/docs/Web/Accessibility/ARIA/Attributes/aria-describedby)와 같은 html accessibility 속성으로 두 태그가 서로 연관 되어있음을 나타낼 수 있다.



### Generating IDs for several related elements
`useId`는 필요한 모든 요소에 대응하여 호출하지 않아도 된다.
```js
import { useId } from 'react';

export default function Form() {
  const id = useId();
  return (
    <form>
      <label htmlFor={id + '-firstName'}>First Name:</label>
      <input id={id + '-firstName'} type="text" />
      <hr />
      <label htmlFor={id + '-lastName'}>Last Name:</label>
      <input id={id + '-lastName'} type="text" />
    </form>
  );
}
```


### Specifying a shared prefix for all generated IDs

단일 페이지에서 여러 개의 독립된 React 앱을 렌더링할 때, `identifierPrefix` 옵션으로 고유한 접두사를 각 식별자에 전달할 수 있다

```js
const root1 = createRoot(document.getElementById('root1'), {
  identifierPrefix: 'my-first-app-'
});
root1.render(<App />);

const root2 = createRoot(document.getElementById('root2'), {
  identifierPrefix: 'my-second-app-'
});
root2.render(<App />);
```