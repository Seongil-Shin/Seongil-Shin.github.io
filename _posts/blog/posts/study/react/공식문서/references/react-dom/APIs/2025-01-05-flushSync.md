https://react-ko.dev/reference/react-dom/flushSync
콜백 내부의 모든 업데이트를 동기적으로 flush 하도록 하여 DOM이 즉시 업데이트 된다.
```js
flushSync(callback)
```


### Caveats
- `flushSync`는 Suspense 경계를 강제로 `fallback` state로 표시할 수 있다
- `flushSync`는 보류중인 Effect들을 실행하고 반환하기 전에 모든 업데이트를 동기적으로 적용할 수 있다
- `flushSync`는 콜백 내부의 업데이트를 flush하기 위해 필요한 경우 콜백 외부의 업데이트를 flush 할 수 있다.
	- 클릭으로 인해 보류중인 업데이트가 있는 경우, 리액트는 콜백 내부의 업데이트를 flush하기 전에 해당 업데이트를 먼저 flush 할 수 있다.
- 브라우저 API나 UI 라이브러리와 같은 서드파티 코드와 통합할 때 사용하고, 그 외에 경우에는 크게 필요하지 않다.