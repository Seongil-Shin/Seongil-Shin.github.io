https://www.joshwcomeau.com/react/server-components/



## 기존 SSR 방식의 단점
1. 라우트 레벨에서만 동작하여 컴포넌트 별로 적용할 수는 없었음
2. 각 메타프레임워크마다 각자의 방식을 사용함. (next.js, gatsby, remix  etc)
3. 모든 컴포넌트가 필요 없더라도 hydration 되어야함. 


## React Server Component
위 SSR 방식의 문제점을 해결하기 위해 리액트에서 만든 방식이다.

특징
- Server Components never re-render
	- 서버에서 한번 실행되고 클라이언트로 보내져 고정된다. 
	- state, effect, event handler 사용이 안됨
- 리액트 서버 컴포넌트는 서버에서만 렌더링되고, JS 번들에 포함되지 않는다. 따라서 hydrate 되지 않고 rerender 되지도 않는다.
- 클라이언트 컴포넌트 하위는 클라이언트 컴포넌트로 간주됨. (`use client`를 안붙여도)
	- 하지만 children으로 넘어가면 서버컴포넌트로 넣을 수 있음. (children으로 넘어간 컴포넌트(A)가 렌더되는 건 children을 넘겨준 컴포넌트(B)이기에, B가 서버컴포넌트이면 A도 서버컴포넌트로 존재할 수 있다.)
	- 좀 더 자세히는 클라이언트 컴포넌트로 import 된 클라이언트는 클라이언트 컴포넌트로 간주 된다는 것


장점
- 번들사이즈를 줄일 수 있다.
	- 만약 라이브러리들도 서버컴포넌트에서만 사용한다면 사용하는 라이브러리를 번들에 포함하지 않아도 되어서 크기를 많이 줄일 수 있다.

### Compatible Enviroments
React Server Component를 바로 사용하기는 어렵고, React 외에도 번들러, 서버, 라우터 등과의 결합이 중요하다.
현재는 next.js 13.4 이상에서만 사용가능하고, [Bleeding-edge frameworks](https://react.dev/learn/start-a-new-react-project#bleeding-edge-react-frameworks)에서 다른 프레임워크가 나타나는 걸 기다릴 수 있다.


### Peeking under the hood
만약 아래와 같은 서버컴포넌트가 있다면
```jsx
function Homepage() {
  return (
    <p>
      Hello world!
    </p>
  );
}
```
만들어지는 html는 다음과 같다.
```html
<!DOCTYPE html>
<html>
  <body>
    <p>Hello world!</p>
    <script src="/static/js/bundle.js"></script>
    <script>
      self.__next['$Homepage-1'] = {
        type: 'p',
        props: null,
        children: "Hello world!",
      };
    </script>
  </body>
</html>
```
- 서버에서 생성된 `Hello world!` html이 삽입되어있음
- `<script>`를 통해 JS 번들이 로딩됨. 여기에는 React 의존성이 포함되고 클라이언트 컴포넌트가 포함됨. 서버컴포넌트는 포함되지 않음
- `<script>`에서 inline JS를 삽입함. 
	- 리액트에게 `Homepage` 컴포넌트가 번들에 없는데 이건 이미 렌더링되었다고 알려주는 역할
	- 리액트는 virtual dom을 만드는데, 이 정보를 재사용함으로써 서버컴포넌트의 virtual dom을 만든다.
	- 이것으로 children으로 넘긴 서버컴포넌트가 클라이언트 컴포넌트 하위에 위치할 수 있게 된다.
		- 클라이언트 컴포넌트는 리렌더 가능하고, children으로 넘긴 컴포넌트는 정적인 값이기에.