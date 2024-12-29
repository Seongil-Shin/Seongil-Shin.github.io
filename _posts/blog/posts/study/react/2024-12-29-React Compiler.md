


## 1. 정의

빌드 시 적용되는 도구로, [[React]]를 자동으로 최적화해준다. 관련 eslint 플러그인도 제공하여 코드 품질을 높이는데 사용할 수 있다.


### 1.1 무엇을 하는가?

기존에는 `useMemo`, `useCallback`, `React.memo` 와 같은 API 로 직접 메모이제이션하였다. 하지만 개발하다보면 이를 잊어버릴수 있다.

리액트 컴파일러는 자동으로 컴포넌트와 Hooks 내의 값 또는 값 그룹을 메모이제이션 한다. 만약 JS나 React 규칙을 위반한 경우에는 해당 컴포넌트 또는 Hooks를 건너뛰고 다른 코드를 컴파일한다.

컴파일러가 초점을 맞춘 두가지 사례
- 컴포넌트의 연쇄적인 리렌더링 건너뛰기 : 부모만이 변경되었어도 자식까지 전부 리렌더링되는 경우
- React 외부에서의 비용이 많이 드는 계산 건너뛰기 : 컴포넌트 내에서 호출하는 경우 메모이제이션됨
```js
// 함수 자체는 리액트 컴파일러에 의해 메모이제이션되지 않음.
function expensiveProcess() { ... }

function Table() {
	// 리액트 컴파일러에 의해 컴포넌트 내에서 메모이제이션 됨.
	const data = expensiveProcess();
}
```

### 1.2 컴파일러가 가정하는 것

1. 올바르고 의미있는 JS 코드로 작성되었다 (valid, semantic JavaScript.)
2. nullable/optional 값과 속성에 접근하기 전에 그 값이 정의되어있는지 테스트한다. (TS를 사용하는 경우 `strictNullCheck`를 활성화하여 수행함)
3. [[React 규칙]]을 따른다.

만약 규칙을 어긋나는 것이 있다면 그 컴포넌트는 컴파일을 건너뛴다.

## 2. 내부 코드

- 컴파일 전 리액트 코드
```js
function FriendList({ friends }) {
	const onlineCount = useFriendOnlineCount();
	if (friends.length === 0) {
		return <NoFriends />;
	}
	
	return (
		<div>
			<span>{onlineCount} online</span>
			{friends.map((friend) => (
				<FriendListCard key={friend.id} friend={friend} />
			))}
			<MessageButton />
		</div>
	);
}
```

- 컴파일 후 코드
	- 의존성이 없는 하위 컴포넌트는 `Symbol.for("react.memo_cache_sentinel")` 이면 렌더링, 아니면 이전에 만든 값 사용
	- 의존성이 있는 컴포넌트는 의존하는 값들이 이전과 같은지 다른지에따라 동작 구분
```js
function FriendList(t0) {
    const $ = _c(9);
    const { friends } = t0;
    const onlineCount = useFriendOnlineCount();
    if (friends.length === 0) {
        let t1;
        if ($[0] === Symbol.for("react.memo_cache_sentinel")) {
            t1 = <NoFriends />;
            $[0] = t1;
        } else {
            t1 = $[0];
        }
        return t1;
    }
    let t1;
    if ($[1] !== onlineCount) {
        t1 = <span>{onlineCount} online</span>;
        $[1] = onlineCount;
        $[2] = t1;
    } else {
        t1 = $[2];
    }
    let t2;
    if ($[3] !== friends) {
        t2 = friends.map(_temp);
        $[3] = friends;
        $[4] = t2;
    } else {
        t2 = $[4];
    }
    let t3;
    if ($[5] === Symbol.for("react.memo_cache_sentinel")) {
        t3 = <MessageButton />;
        $[5] = t3;
    } else {
        t3 = $[5];
    }
    let t4;
    if ($[6] !== t1 || $[7] !== t2) {
        t4 = (
            <div>
                {t1}
                {t2}
                {t3}
            </div>
        );
        $[6] = t1;
        $[7] = t2;
        $[8] = t4;
    } else {
        t4 = $[8];
    }
    return t4;
}
function _temp(friend) {
    return <FriendListCard key={friend.id} friend={friend} />;
}

```


컴파일 후 코드를 보면 `_c` 함수에서 메모이제이션을 담당하고 있는것을 확인할 수 있다.  
- 필요한 의존성만큼의 사이즈를 가진 배열 상태를 반환
- 각 요소의 초기값은 `Symbol.for("react.memo_cache_sentinel")`

```ts
// compiler/packages/react-compiler-runtimes/src/index.ts

type MemoCache = Array<number | typeof $empty>;

const $empty = Symbol.for('react.memo_cache_sentinel');
/**
 * DANGER: this hook is NEVER meant to be called directly!
 **/
export function c(size: number) {
  return React.useState(() => {
    const $ = new Array(size);
    for (let ii = 0; ii < size; ii++) {
      $[ii] = $empty;
    }
    // This symbol is added to tell the react devtools that this array is from
    // useMemoCache.
    // @ts-ignore
    $[$empty] = true;
    return $;
  })[0];
}
```

## 3. 궁금증


- 올바르고 의미있는 JS 라는 것은? (valid, semantic JavaScript.)
	- valid : ECMAScript 표준에 부합하는 문법과 구조를 가지고 있고, 파싱 과정에서 에러나 경고를 일으키지 않는 코드
	- semantic : 문법적으로만 맞추는 것을 넘어, 의도한 바를 명확히 드러내고 예상 가능한 동작을 하도록 구성된 코드를 의미.
		- ex) 변수나 함수 사용 시 의도된 스코프에서 유효하게 사용. 타입 변환 로직이나 비동기 흐름이 명확하게 관리됨
- 최초 로딩 성능에 영향을 줄까? 
	- 큰 영향이 없다고 함
- 유저가 상호작용시 성능은 어떨까?
	- 불필요한 리렌더링을 줄이니 매우 빨라짐
- 모든 리렌더링을 잡아서 더이상 개발자는 메모이제이션을 신경쓰지 않아도 될까?
	- 아니다. 리액트 컴파일러는 props 가 변하지 않을때만 대응되지, props 설계를 잘못하면 리렌더링은 여전히 발생하기에 신경은 써야한다.
- 참조를 props로 넣었을때 리액트 컴파일러는 작동하지 않는가?
	- 아니다. 참조를 사용했을때도 적절히 컴파일 된다. 
- 외부 라이브러리 사용시에는?
	- 리액트 입장으론 대응을 아직 잘 못함









## 참고
- https://www.youtube.com/watch?v=T-rHmWSZajc&ab_channel=ReactConferencesbyGitNation
- https://ko.react.dev/learn/react-compiler
- https://helloinyong.tistory.com/365