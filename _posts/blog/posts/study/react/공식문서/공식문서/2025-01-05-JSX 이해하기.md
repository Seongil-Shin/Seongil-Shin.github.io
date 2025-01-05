- 구 가이드 : https://ko.legacy.reactjs.org/docs/jsx-in-depth.html

## JSX에서 자식 다루기


### 문자열 리터럴
여는 태그와 닫는 태그 사이에 문자열 리터럴을 넣을 수 있고, `props.children`은 그 자식이 된다.
이때 JSX는 다음과 같이 공백을 제거한다.
- 각 줄의 처음과 끝에 있는 공백을 제거
- 빈 줄 제거 
- 태그에 붙어있는 개행 제거
- 문자열 리터럴 중간에 있는 개행은 한 개의 공백으로 대체.
따라서 아래 예시는 모두 똑같이 렌더링 된다.
```html
<div>Hello World</div>

<div>
  Hello World
</div>

<div>
  Hello
  World
</div>

<div>

  Hello World
</div>
```

### 함수를 자식으로 사용하기
다음과 같이 함수를 `props.children`으로 넘겨 줄 수도 있다.
```js
// 자식 콜백인 numTimes를 호출하여 반복되는 컴포넌트를 생성합니다.
function Repeat(props) {
  let items = [];
  for (let i = 0; i < props.numTimes; i++) {    
	  items.push(props.children(i));
  }
  return <div>{items}</div>;
}

function ListOfTenThings() {
  return (
    <Repeat numTimes={10}>
      {(index) => <div key={index}>This is item {index} in the list</div>} 
	</Repeat>
  );
}
```

### boolean, null, undefined는 무시된다.
`true`, `false`, `null`, `undefined`는 렌더링 되지않는다. 아래는 모두 동일하게 렌더링된다.
```js
<div />

<div></div>

<div>{false}</div>

<div>{null}</div>

<div>{undefined}</div>

<div>{true}</div>
```
`0`과 같은 Falsy 값은 렌더링 된다.
```js
// length가 0일시, 0 렌더
<div>
  {props.messages.length &&
   <MessageList messages={props.messages} />
  }
</div>

// 0일시, 렌더링되지 않음
<div>
  {props.messages.length > 0 &&   
   <MessageList messages={props.messages} />
  }
</div>
```