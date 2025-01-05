- 신 가이드 : https://react.dev/learn/preserving-and-resetting-state

## Summary  
리액트에서 컴포넌트의 상태(State)는 UI 트리(Virtual Dom)의 위치에 연결되며, 같은 위치에서 컴포넌트가 계속 렌더링될 때 상태가 유지되고, 다른 위치로 이동하거나 삭제될 때 상태가 재설정됩니다. 상태를 보존하거나 재설정하는 방법을 이해하고, 상태를 유지하려면 컴포넌트 구조가 일치해야 하며, 키와 타입이 상태 보존에 영향을 미치는 방법을 배웁니다.  
  
### Facts  
#### The UI tree
- 🌳 브라우저는 UI를 모델링하기 위해 다양한 트리 구조를 사용합니다. DOM은 HTML 요소를 나타내고, CSSOM은 CSS를 위해 사용됩니다. Accessibility 트리도 있습니다.  
- ⚛️ 리액트는 UI를 관리하고 모델링하기 위해 트리 구조를 사용합니다. JSX로부터 UI 트리를 생성하고, React DOM은 브라우저 DOM 요소를 해당 UI 트리와 일치하도록 업데이트합니다. (리액트 네이티브는 모바일 플랫폼에 특화된 요소로 이러한 트리를 번역합니다.)

#### State is tied to a position in the tree
- 상태(State)는 UI 트리(Virtual DOM)의 위치와 연결되며, 컴포넌트에서 상태를 설정하면 실제로는 리액트 내부에 상태가 저장됩니다. 리액트는 UI 트리의 위치에 따라 올바른 컴포넌트와 상태를 연결합니다.  
- 리액트에서 화면의 각 컴포넌트는 완전히 격리된 상태를 가집니다. 예를 들어, 두 개의 <Counter /> 컴포넌트를 나란히 렌더링하면 각각 독립적인 상태를 갖습니다.  
- **상태는 동일한 컴포넌트가 동일한 위치에서 렌더링될 때** 유지됩니다. 컴포넌트를 다른 위치로 이동하거나 삭제하면(렌더링이 종료되면) 리액트는 해당 상태를 폐기합니다.  

#### Same component at the same position preserves state
- **같은 위치에 같은 컴포넌트면 상태를 유지한다. 따라서 props가 바뀌더라도, 같은 컴포넌트가 같은 위치에 있으면 상태를 폐기하지 않는다.**
- 이떄의 위치는 UI tree에서의 위치로, JSX 마크업에서의 위치가 아니다. 예를 들어 다음 예시에서 `Counter`는 서로 다른 곳에서 return 되지만, `isFancy`가 변경되더라도 `Counter` 상태는 유지된다.
```js
import { useState } from 'react';

export default function App() {
  const [isFancy, setIsFancy] = useState(false);
  if (isFancy) {
    return (
      <div>
        <Counter isFancy={true} />
        <input
            type="checkbox"
            checked={isFancy}
            onChange={e => {
              setIsFancy(e.target.checked)
            }}
        />
      </div>
    );
  }
  return (
    <div>
      <Counter isFancy={false} />
	  <input
          type="checkbox"
          checked={isFancy}
          onChange={e => {
            setIsFancy(e.target.checked)
          }}
	   />
    </div>
  );
}

function Counter({ isFancy }) {
  const [score, setScore] = useState(0);

  let className = 'counter';
  if (isFancy) {
    className += ' fancy';
  }

  return (
    <div className={className}>
      <h1>{score}</h1>
      <button onClick={() => setScore(score + 1)}>
        Add one
      </button>
    </div>
  );
}

```


#### Different components at the same position reset state
- 조건부 렌더링으로 같은 위치에 서로 다른 컴포넌트가 나타나면 기존 컴포넌트와 그 하위 컴포넌트의 상태는 모두 폐기한다. 
- 다음 예시는 위 예시와 다르게, 같은 위치에 `Counter`가 있지만, 상위 컴포넌트가 다르기에 조건이 변경되면 기존 `Counter`의 상태는 페기한다.
```js
import { useState } from 'react';

export default function App() {
  const [isFancy, setIsFancy] = useState(false);
  return (
    <div>
      {isFancy ? (
        <div>
          <Counter isFancy={true} /> 
        </div>
      ) : (
        <section>
          <Counter isFancy={false} />
        </section>
      )}
        <input
          type="checkbox"
          checked={isFancy}
          onChange={e => {
            setIsFancy(e.target.checked)
          }}
        />
    </div>
  );
}

function Counter({ isFancy }) {
  const [score, setScore] = useState(0);

  let className = 'counter';
  if (isFancy) {
    className += ' fancy';
  }

  return (
    <div className={className}>
      <h1>{score}</h1>
      <button onClick={() => setScore(score + 1)}>
        Add one
      </button>
    </div>
  );
}
```


- 따라서 **nest component functuon definition을 피해야한다**. 예를들어 다음과 같은 예시는, `counter` 상태가 변경될때마다, `MyComponent`가 매번 새롭게 만들어지고, 리액트는 이것을 다른 컴포넌트라고 인식하여 상태를 초기화 한다.
```js
import { useState } from 'react';

export default function MyComponent() {
  const [counter, setCounter] = useState(0);

  function MyTextField() {
    const [text, setText] = useState('');

    return (
      <input
        value={text}
        onChange={e => setText(e.target.value)}
      />
    );
  }

  return (
    <>
      <MyTextField />
      <button onClick={() => {
        setCounter(counter + 1)
      }}>Clicked {counter} times</button>
    </>
  );
}
```
	- 단, 이는 컴포넌트 안에 컴포넌트가 있는 상황이고, 컴포넌트 안에 컴포넌트를 만드는 함수가 있는 것은 괜찮다. (대신 컴포넌트가 아니므로 상태를 사용할 수 없음)

### Resetting state at the same position

리액트는 컴포넌트가 같은 위치에 있는한 상태를 유지한다. 하지만 상태를 초기화 하고 싶을 떄가 있다. 그럴떈 다음 두가지 방법이 있다.

1. Rendering a component in different positions
```js
import { useState } from 'react';

export default function Scoreboard() {
  const [isPlayerA, setIsPlayerA] = useState(true);
  return (
    <div>
      {isPlayerA &&
        <Counter person="Taylor" />
      }
      {!isPlayerA &&
        <Counter person="Sarah" />
      }
      <button onClick={() => {
        setIsPlayerA(!isPlayerA);
      }}>
        Next player!
      </button>
    </div>
  );
}

function Counter({ person }) {
  const [score, setScore] = useState(0);

  let className = 'counter';

  return (
    <div className={className}>
      <h1>{person}'s score: {score}</h1>
      <button onClick={() => setScore(score + 1)}>
        Add one
      </button>
    </div>
  );
}
```
	
- 위와같이 하면 버튼을 누를때마다 Counter가 DOM에서 제거되고 상태는 초기화된다.
- Question : Same component at the same position preserves state 예제랑 뭐가 다른거지?
	- UI 트리는 Virtual DOM을 의미하고, Virtual DOM에서는 같은 `{}`으로 나타나 있어야 같은 위치로 취급한다. 다른 `{}`에 감싸져있으면 다른 위치로 판단함. 위 예제는 다른 `{}`에서 하나는 `false`를 가지게 되니 다른 위치로 구별되는 것이다.
	- 이거에 관해서 다른 사람들도 많이 헷갈리는 모양이다
			- [림크1](https://stackoverflow.com/questions/76809228/question-about-preserving-and-resetting-state-in-react)
			- [링크2](https://stackoverflow.com/questions/75884367/how-to-undetstand-react-keeps-state-for-as-long-as-the-same-component-is-rendere)

2. Resetting state with a key
```js
import { useState } from 'react';

export default function Scoreboard() {
  const [isPlayerA, setIsPlayerA] = useState(true);
  return (
    <div>
      {isPlayerA ? (
        <Counter key="Taylor" person="Taylor" />
      ) : (
        <Counter key="Sarah" person="Sarah" />
      )}
      <button onClick={() => {
        setIsPlayerA(!isPlayerA);
      }}>
        Next player!
      </button>
    </div>
  );
}

function Counter({ person }) {
  const [score, setScore] = useState(0);

  let className = 'counter';

  return (
    <div className={className}>
      <h1>{person}'s score: {score}</h1>
      <button onClick={() => setScore(score + 1)}>
        Add one
      </button>
    </div>
  );
}
```
	- 위와같이 `key` prop을 변경하여 상태를 초기화 할 수 있다.



### for more details
- [재조정](재조정.md)



#review 