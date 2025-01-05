
- 첫번째처럼 할 경우 모든 요소에 같은 배열의 참조가 들어감
```js
// bad
var arr = new Array(1000).fill(new Array())

// good
var arr = new Array(1000).fill(0).map(() => []);
```

- dfs 와 같이 함수를 깊이 탐색해야할때 런타임 오류가 발생하면, 재귀함수가 아닌 반복문으로 dfs 구현해야함
- 진수변환
```js
// 문자열 -> 숫자
var num = parseInt("12341", 4); // (문자열, 진수)

// 숫자 -> 문자열
var str = 1314123.toString(6); // 숫자.toString(진수)
```

- array의 `.shift()`, `.unshift()`를 통해 큐를 구현할 수 있다. 하지만 성능은 느리다
	- `shift()` :  첫번째 요소를 제거하고 그 값을 반환
	- `.unshift()` : 첫번째에 요소를 넣고, 길이를 반환
- js에서 성능좋은 큐를 구현하려면 linked list 형태로 직접 구현해야한다.
- 스택은 `.pop()`으로 구현 가능하다