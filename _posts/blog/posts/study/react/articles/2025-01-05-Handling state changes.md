https://reactkungfu.com/2016/03/dive-into-react-codebase-handling-state-changes/
### 요약  
React.js에서 상태(State)는 프로젝트에서 복잡한 개념 중 하나이며, 몇몇 개발자는 상태를 외부로 분리함으로써 이를 해결했지만 (Redux를 사용한 경우도 있음) 여전히 React.js의 널리 사용되는 기능 중 하나입니다. 상태(State)는 편리하지만 일부 문제를 일으킬 수 있으며, setState가 비동기적으로 작동하기 때문에 발생하는 문제도 있습니다.  
  
### 상태 유지 문제 해결
`setState`가 비동기적으로 동작하기 때문에 발생하는 문제점은 다음과 같다.
```js
changeTitle: function changeTitle (event) {
  this.setState({ title: event.target.value });
  this.validateTitle();
},
validateTitle: function validateTitle () {
  if(this.title.length === 0) {
    this.setState({ titleError: "Title can't be blank" });
  }
},
```
위와 같은 코드에서, `validateTitle`은 정상적으로 동작하지않는데, `setState`가 비동기적으로 동작하기에 `validateTitle`에서 검사하는 `title`은 이전 상태이기 때문이다.

React 0.14.7에서 위에서 설명한 문제를 해결하는 여러 가지 방법이 있습니다:  
- this.setState의 내장 콜백 기능을 사용하여 상태 업데이트 후에 콜백을 호출합니다.  
	- `setState({ title: event.target.value }, function () { ... })`
	- 위와 같이 상태업데이트 후 호출할 콜백을 넣을 수 있다.
	- `useState`로 생성한 경우에는 안되는 거 같음.
- 다음 상태를 외부로 뽑아, `setState`와 업데이트 후 실행할 함수에 모두 넣는다.
- 상태를 외부화하여 React 부분에서 상태 문제를 피할 수 있습니다.  

하지만 이 방법이 정말 유효할까? react codebase를 살펴보자.
  
### React setState 메서드 분석  
setState 메서드는 ReactComponent 프로토타입에 정의되어 있으며, React 클래스의 정의에서 확장됩니다. setState 메서드는 상태 변경을 예약하고 콜백을 예약할 수 있도록 합니다.  
  
### ReactUpdateQueue를 통한 상태 변경 예약  
setState 메서드 내에서는 setState 메서드와 콜백을 예약하는 두 가지 메서드가 호출됩니다. ReactUpdateQueue는 이러한 메서드의 구현을 담당하며 상태 변경 및 콜백을 예약하고 변경 사항을 처리합니다.  
  
### ReactUpdates를 통한 업데이트 수행  
ReactUpdates는 업데이트를 수행하는 역할을 합니다. setState 메서드를 호출하면 상태 변경 및 콜백이 업데이트 큐에 추가되며, 이 업데이트 큐는 나중에 처리됩니다. ReactUpdates는 업데이트를 일괄 처리하고 상태를 업데이트하는 역할을 합니다.
