https://react-ko.dev/reference/react/useImperativeHandle
ref로 노출되는 핸들을 사용자가 직접 정의할 수 있게 해준다.

```js
useImperativeHandle(ref, createHandle, dependencies?)
```

```js
import { forwardRef, useImperativeHandle } from 'react';  

const MyInput = forwardRef(function MyInput(props, ref) {  

	useImperativeHandle(ref, () => {  
		return {  
			// ... 메서드는 여기에 작성합니다 ...  
		};  
	}, []);
```

### Parameters
- `ref` : `forwardRef` 함수에서 두번째인자로 받은 객체
- `createHandle` : 노출하려는 ref 핸들을 반환하는 함수. 아무런 유형이든 될 수 있으나, 일반적으로는 노출하려는 메서드가 있는 객체를 반환한다.
- `dependencies` (optional)
	- `createHandle`에서 참조하는 모든 반응형 값을 나열. 
	- 각 의존성을 `Object.is`로 이전 값과 비교하고, 변경되었으면 `createHandle`을 다시 실행하고 재생성된 핸들을 ref에 할당한다.

### Caveats
- ref를 과도하게 사용하면 안된다. props로 표현할 수 없는 필수적인 행동에만 사용해야한다.
	- ex) scrollToView, focus, startAnimation, selecting text... etc
- props로 표현할 수 있는 것은 ref를 사용하지마라
	- ex) 모달에서 `open`, `close`를 노출하는 것보다 `<Modal isOpen={isOpen}/>`과 같이 `isOpen` prop을 사용하는 것이 더 좋다.

## Usage

### Exposing a custom ref handle to the parent component
`ref` 핸들 중 모든 것을 부모에게 노출하고 싶지 않을 수도 있다. 이럴떄 다음과 같이 `useImperativeHandle`를 사용해서 필요한 것만 노출이 가능하다.
```js
import { forwardRef, useRef, useImperativeHandle } from 'react';  

const MyInput = forwardRef(function MyInput(props, ref) {  
	const inputRef = useRef(null);  
	
	useImperativeHandle(ref, () => {  
		return {  
			focus() {  
				inputRef.current.focus();  
			},  
			scrollIntoView() {  
				inputRef.current.scrollIntoView();  
			},  
		};  
	}, []);  
	
	return <input {...props} ref={inputRef} />;  
});
```

### Exposing custom imperative methods

```js
import { forwardRef, useRef, useImperativeHandle } from 'react';

const CommentList = forwardRef(function CommentList(props, ref) {
  const divRef = useRef(null);

  useImperativeHandle(ref, () => {
    return {
      scrollToBottom() {
        const node = divRef.current;
        node.scrollTop = node.scrollHeight;
      }
    };
  }, []);

  return (
    <div className="CommentList" ref={divRef}>
      {props.comments}
    </div>
  );
});

export default CommentList;
```


