https://react.dev/learn/updating-objects-in-state
https://react.dev/learn/updating-arrays-in-state

- object, array 변경시 보통 spread 연산자를 사용한다.
- 기존 객체의 mutate를 막고, 새로운 객체로 업데이트 해주어야함.
```js
// BAD : mutate
const myNextList = [...myList];  
const artwork = myNextList.find(a => a.id === artworkId);  
artwork.seen = nextSeen; // Problem: mutates an existing item  
setMyList(myNextList);

// GOOD
setMyList(myList.map(artwork => {  
	if (artwork.id === artworkId) {  
		// Create a *new* object with changes  
		return { ...artwork, seen: nextSeen };  
	} else {  
		// No changes  
		return artwork;  
	}  
}));
```
- Immer를 사용해서 복잡한 object의 상태 변화를 좀 더 쉽게 할 수 있다. (array 포함)
```js
  const [person, updatePerson] = useImmer({
    name: 'Niki de Saint Phalle',
    artwork: {
      title: 'Blue Nana',
      city: 'Hamburg',
      image: 'https://i.imgur.com/Sd1AgUOm.jpg',
    }
  });

  function handleNameChange(e) {
	// 변경을 원하는 값만 mutate 해주듯 변경하면 됨
    updatePerson(draft => {
      draft.name = e.target.value;
    });
  }

  function handleTitleChange(e) {
    updatePerson(draft => {
      draft.artwork.title = e.target.value;
    });
  }

  function handleCityChange(e) {
    updatePerson(draft => {
      draft.artwork.city = e.target.value;
    });
  }
```

```js
// mutate지만, immer에선 괜찮음
updateMyTodos(draft => {  
	const artwork = draft.find(a => a.id === artworkId);  
	artwork.seen = nextSeen;  
});
```
