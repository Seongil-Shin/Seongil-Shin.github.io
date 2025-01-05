https://react-ko.dev/reference/react/Profiler
React 트리의 렌더링 성능을 프로그램적으로 측정할 때 사용할 수 있다.
```jsx
<Profiler id="App" onRender={onRender}>  
	<App />  
</Profiler>
```

### Parameter
- `id`
	- 측정 중인 UI 부분을 식별하는 문자열
- `onRender`
	- 프로파일링된 트리 내의 컴포넌트가 업데이트될 때마다 React가 호출하는 콜백.
	- 렌더링된 내용과 소요된 시간에 대한 정보를 받음

### Caveats
- 오버헤드가 있기에 프로덕션에서는 비활성화되어있다.
- 프로덕션에서도 사용하려면 [특수 프로덕션 빌드](https://gist.github.com/bvaughn/25e6233aeb1b4f0cdb8d8366e54a3977)를 사용해야한다.

