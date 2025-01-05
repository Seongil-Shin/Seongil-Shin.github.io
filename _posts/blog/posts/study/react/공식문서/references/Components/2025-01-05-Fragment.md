https://react-ko.dev/reference/react/Fragment


### Caveats
- React는 `<><Child/></>`와 `[<Child/>]` 사이, `<><Child/></>`와 `<Child/>` 사이를 번갈아 렌더링할 때 state를 재설정하지 않는다.  하지만 한단계 더 깊은 `<><><Child/></></>`와 `[<Child/>]`를 번갈아 렌더링할 때는 state를 재설정한다. [참고](https://gist.github.com/clemmy/b3ef00f9507909429d8aa0d3ee4f986b)
