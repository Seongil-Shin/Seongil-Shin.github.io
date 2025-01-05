https://react.dev/learn/choosing-the-state-structure


리액트에서 상태 구조를 짤 때의 원칙들에 대해서 소개하는 섹션이다. DB의 정규화처럼 실수나 버그가 발생할 부분을 줄이는 것이 목적이다.

- Group related state
```js
// BAD
const [x, setX] = useState(0);
const [y, setY] = useState(0);

// GOOD
const [position, setPosition] = useState({x:0, y:0});
```

- Avoid contradictions in state
```js
// BAD
const [isSending, setIsSending] = useState(false);
const [isSent, setIsSent] = useState(false);

// GOOD
const [status, setStatus] = useState("typing");

const isSending = status === "sending";
const isSent = status === "sent";
```
	- BAD에서는 `isSending`과 `isSent`라는 상반된 상태를 두었다. 동작은 하겠지만, 실수로 한가지 상태를 업데이트 하지 않는 경우가 발생할 수 있다. 
	- 따라서 GOOD에서와 같이 status를 나타내는 하나의 상태를 만들고, isSending과 isSent는 그 status에 따라 값이 자동으로 업데이트 되도록하였다.

- Avoid redundant in state
```js
// BAD
const [firstName, setFirstName] = useState("");
const [lastName, setLastName] = useState("");
const [fullName, setFullName] = useState("");

// GOOD
const [firstName, setFirstName] = useState("");
const [lastName, setLastName] = useState("");

const fullName = firstName + ' ' + lastName;
```

	- props나 다른 상태들로부터 계산이 가능한 값은 상태로 지정하지 말라는 원칙이다.


- Avoid duplication in state
```js
// BAD
const [items, setItems] = useState(initialItems);
const [selectedItem, setSelectedItem] = useState(initialItems[0]);


// GOOD
const [items, setItems] = useState(initialItems);
const [selectedId, setSelectedId] = useState(0);
```
	- BAD에서는 items에 담긴 요소 중 하나를 selectedItem에도 가지고 있다. 이렇게 되면 두개다 업데이트 해야하는 상황에서 실수로 하나를 뺴고 업데이트 할 수 있게된다.
	- 따라서 GOOD에서처럼 중복된 정보를 제거하는 방식이 바람직하다.


- Avoid deeply nested state
```js
// BAD
export const initialTravelPlan = {
  id: 0,
  title: '(Root)',
  childPlaces: [{
    id: 1,
    title: 'Earth',
    childPlaces: [{
      id: 2,
      title: 'Africa',
      childPlaces: [{
        id: 3,
        title: 'Botswana',
        childPlaces: []
      }, {
        id: 4,
        title: 'Egypt',
        childPlaces: []
      }, {
        id: 5,
        title: 'Kenya',
        childPlaces: []
      }, {
        id: 6,
        title: 'Madagascar',
        childPlaces: []
      }, {
        id: 7,
        title: 'Morocco',
        childPlaces: []
      }, {
        id: 8,
        title: 'Nigeria',
        childPlaces: []
      }, {
        id: 9,
        title: 'South Africa',
        childPlaces: []
      }]
    }]
  }]
};

// GOOD
export const initialTravelPlan = {
  0: {
    id: 0,
    title: '(Root)',
    childIds: [1],
  },
  1: {
    id: 1,
    title: 'Earth',
    childIds: [2, 10, 19, 26, 34]
  },
  2: {
    id: 2,
    title: 'Africa',
    childIds: [3, 4, 5, 6 , 7, 8, 9]
  }, 
  3: {
    id: 3,
    title: 'Botswana',
    childIds: []
  },
  4: {
    id: 4,
    title: 'Egypt',
    childIds: []
  },
  5: {
    id: 5,
    title: 'Kenya',
    childIds: []
  },
  6: {
    id: 6,
    title: 'Madagascar',
    childIds: []
  }, 
  7: {
    id: 7,
    title: 'Morocco',
    childIds: []
  },
  8: {
    id: 8,
    title: 'Nigeria',
    childIds: []
  },
  9: {
    id: 9,
    title: 'South Africa',
    childIds: []
  },
}

```
		- 너무 깊은 object를 상태에 넣으면 상태를 업데이트할때 코드가 장황해질 수 있다.
		- 추천하는 방법은 nested object를 flat하게 변경하여, 각자 자기 자식의 id를 가지고 있는 식으로 변경할 수 있다.



#review 