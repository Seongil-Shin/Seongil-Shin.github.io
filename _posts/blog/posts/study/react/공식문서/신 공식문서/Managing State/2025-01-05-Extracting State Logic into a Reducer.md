https://react-ko.dev/learn/extracting-state-logic-into-a-reducer

컴포넌트 내 여러 이벤트핸들러에 복잡한 상태 업데이트 로직이 흩어져있다면 reducer를 사용하는 것이 좋을 수 있다.

- before
```js
  const [tasks, setTasks] = useState(initialTasks);

  function handleAddTask(text) {
    setTasks([
      ...tasks,
      {
        id: nextId++,
        text: text,
        done: false,
      },
    ]);
  }

  function handleChangeTask(task) {
    setTasks(
      tasks.map((t) => {
        if (t.id === task.id) {
          return task;
        } else {
          return t;
        }
      })
    );
  }

  function handleDeleteTask(taskId) {
    setTasks(tasks.filter((t) => t.id !== taskId));
  }
```
- after
```js
export default function TaskApp() {
  const [tasks, dispatch] = useReducer(tasksReducers, initialTasks);

  function handleAddTask(text) {
    dispatch([
      {
	    type: 'added',
        id: nextId++,
        text: text,
      },
    ]);
  }

  function handleChangeTask(task) {
    dispatch([
      {
	    type: 'changed',
	    task
      },
    ]);
  }

  function handleDeleteTask(taskId) {
    dispatch([
      {
	    type: 'deleted',
	    id: taskId
      },
    ]);
  }
}

function tasksReducers(tasks, action) {
		switch(action.type) {
			case "added": {
				return [
					...tasks, 
					{
						id: action.id,
						text: action.text,
						done: false
					}
				]
			}
			case "changed": {
				return tasks.map((t) => {
			        if (t.id === action.task.id) {
			          return action.task;
			        } else {
			          return t;
			        }
			      })
			}
			case "deleted": {
				return tasks.filter((t) => t.id !== action.id);
			}
			default: {
				throw Error("Unknown action : " + action.type)
			}
		}
}
```
 - `case` 문의 각 블럭은 중괄호로 감싸는게 좋은데, case 들 안에서 선언된 변수들이 서로 충돌하지 않도록 하기 위함이다.


## `useState` vs `useReducer`

- Code Size
	- 일반적으로 `useState`가 적으나, 복잡한 상태변화가 여러 이벤트핸들러에서 반복해서 나타난다면 `useReducer`가 적다.
- Readability
	- 단순한 상태변화만 있다면 `useState`가 낫지만, 복잡하면 `useReducer`가 낫다
- Debugging
	- `useReducer`는 한 곳에서 상태 변화를 총괄하니 로그를 찍어 debugging하기가 쉽다.
- Testing
	- reducer 함수는 컴포넌트에 의존하지 않으므로 별도로 분리해서 테스트할 수 있다.

구조가 복잡하면 `useReducer`, 단순하면 `useState`가 나음

## `useReducer` 잘 사용하기

- reducer 함수는 순수해야한다.
	- 같은 입력에는 같은 값을 반환해야한다.
	- 요청을 보내거나 timeout 스케쥴링, 사이드이펙트를 내서는 안된다.
- 각 action은 여러 상태를 변경하더라도 하나의 유저 인터랙션을 의미해야한다.


## `immer` 사용하기

`immer`를 사용하여 간결한 reducer를 작성할 수있다.

