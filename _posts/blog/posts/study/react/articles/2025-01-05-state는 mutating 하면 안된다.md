[Updating Objects, Array in state](obsidian://open?vault=Obsidian%20Vault&file=react%2F%EC%8B%A0%20%EA%B3%B5%EC%8B%9D%EB%AC%B8%EC%84%9C%2FAdding%20interactivity%2FUpdating%20Objects%2C%20Array%20in%20State)에서 state는 read-only로 취급하고, mutate하면 안된다고 하였다. 

이유는 다음과 같다.
1. `setState` 없이 mutate 해도 리액트가 리렌더링을 수행하지 않는다.
2. 리렌더링을 수행하지 않는데, 메모리 속의 state 객체는 변경된다. 따라서 추후 리렌더링 되었을때 데이터가 의도치않게 변경될 수 있다.
```js
<button
	onClick={() => {
		nums.a += 1;
		nums.b += 1;
	}}
>	
	Increase
</button>
<button
	onClick={() => {
		setNums((prev) => ({ a: prev.a + 1, b: prev.b + 1 }));
	}}
>	
	Increase with setState
</button>
```
	- `Increase`를 여러번 누르고 `Increase with setState` 버튼을 누르면 많은 숫자가 한번에 증가한다.

3. 같은 함수 내에서 mutating과 setState가 동시에 수행하면 의도치 않은 결과를 낳을 수 있다.
```js
<button
	onClick={() => {
		nums.b *= 2;
		setNums((prev) => ({ a: prev.a + 1, b: prev.b + 1 }));
		setNums((prev) => ({ a: prev.a + 1, b: prev.b + 1 }));
	}}
>	
	Increase
</button>
```
	- 클릭시 nums.b * 2 + 1 + 1 로 결과물이 나타남
```js
<button
	onClick={() => {
		setNums((prev) => ({ a: prev.a + 1, b: prev.b + 1 }));
		nums.b *= 2;
		setNums((prev) => ({ a: prev.a + 1, b: prev.b + 1 }));
	}}
>	
	Increase
</button>
```
	- 클릭시 최초 1회는 nums.b + 1 + 1 의 결과물이 나타남
	- 이후론 nums.b * 2 + 1 + 1 로 나타남
```js
<button
	onClick={() => {
		setNums((prev) => ({ a: prev.a + 1, b: prev.b + 1 }));
		setNums((prev) => ({ a: prev.a + 1, b: prev.b + 1 }));
		nums.b *= 2;
	}}
>	
	Increase
</button>
```
	- 두번쨰와 마찬가지
- 이와 같은 동작이 발생하는 이유는, 리액트에서는 setState를 통한 상태 변화를 그 즉시 수행하지않고, 배치처리를 위해 큐에 모았다가 실행하기 때문이다. [링크](obsidian://open?vault=Obsidian%20Vault&file=react%2F%EC%8B%A0%20%EA%B3%B5%EC%8B%9D%EB%AC%B8%EC%84%9C%2FAdding%20interactivity%2FQueueing%20a%20Series%20of%20State%20Updates)
- 따라서 mutate 먼저 실행하고, setState 함수들을 수행하기에 mutate 문의 순서에 상관없이 비슷한 결과물이 나타난다.

위와 같은 이상동작들 떄문에 state를 직접 mutate하는 것은 비권장된다.
굳이 이상동작만이 아니라, 리액트의 원칙이 state는 불변하다는 것이고 함수형 프로그래밍의 원칙으로 직접 mutate는 피해야한다.


좀 더 자세히 https://www.google.com/search?q=mutating+state+react&sca_esv=563685443&sxsrf=AB5stBgI6pM0gLY3C0hsTcakxKj87k-yJg%3A1694166240879&ei=4Oz6ZPScNbuO2roPhqWWuAI&oq=state+mutating+&gs_lp=Egxnd3Mtd2l6LXNlcnAiD3N0YXRlIG11dGF0aW5nICoCCAAyBhAAGAgYHjIGEAAYCBgeMgYQABgIGB4yCBAAGAgYHhgPMgYQABgIGB4yBhAAGAgYHkjVOVDYLVjYLXADeAGQAQCYAbkBoAG5AaoBAzAuMbgBAcgBAPgBAcICChAAGEcY1gQYsAPiAwQYACBBiAYBkAYI&sclient=gws-wiz-serp
