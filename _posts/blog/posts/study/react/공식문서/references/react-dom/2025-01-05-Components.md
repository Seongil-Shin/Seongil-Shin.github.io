https://react-ko.dev/reference/react-dom/components
React는 모든 브라우저 빌트인 HTML 및 SVG 컴포넌트를 지원한다.


## Common
`<div>`와 같은 모든 브라우저 빌트인 컴포넌트를 몇가지 props와 함께 이벤트를 지원한다.

리액트가 추가한 props
- `children`
- `dangerouslySetInnerHTML`
- `ref`
- `suppressContentEditableWarning`
	- `children`과 `contentEditable={true}`가 함께 있을때 React의 경고를 엊제함
- `suppressHydrationWarning`
	- 서버렌더링 시 서버와 클라이언트가 서로 다른 콘텐츠를 렌더릴할 때 오류를 내지 않기위해 사용함
	- 타임스탬프를 렌더링하는 같은 경우 맞추기가 어렵기에 사용한다.
- `style`
- `accessKey`
- `aria-*`
- `autoCapitalize` : 사용자 입력을 대문자로 표시할지 여부
- `className`
- `contentEditable`
	- `true`면 브라우저는 사용자가 렌더링된 엘리먼트를 직접 편집할 수 있도록 허용한다.
	- 사용자가 편집한 후 리액트는 그 내용을 업데이트할 수 없다
- `data-*`
- `dir` : `ltr` 또는 `rtl`. 엘리먼트의 텍스트 방향을 지정함
- `draggable`
- `enterKeyHint`
- `htmlFor`
- `hidden` : 엘리먼트의 숨김여부를 지정함
- `id`
- `inputMode` : 표시할 키보드의 종류(텍스트, 숫자)를 지정한다
- `itemProp` : 구조화된 데이터 크롤러에 대해 엘리먼트가 나타내는 속성을 지정
- `lang` : 엘리먼트의 언어를 지정
- `on~` : 이벤트
- `role` : 보조기술에 대한 엘리먼트의 역할을 명시적으로 지정함
- `slot` : 섀도우 DOM을 사용할 떄 슬롯의 이름을 지정함. 
- `spellCheck` : 맞춤법 검사를 활성화 또는 비활성화
- `tabIndex`: 기본 탭버튼 동작을 재정의함
- `title` : 엘리먼트의 툴팁 텍스트를 지정함
- `translate` : `yes` | `no`. `no`를 전달하면 엘리먼트가 번역에서 제외됨


## Form components
다음 브라우저 빌트인 컴포넌트들은 사용자입력을 허용함
- `<input>`
- `<select>`
- `<textarea>`
`value` prop을 전달하면 이들은 제어 컴포넌트가 된다.

### `<input>`, `<select>` , `<textarea>`
- `autoFocus` : 엘리먼트가 마운트될 때 초점을 맞춤
- `dirname` : 엘리먼트의 방향성에 대한 폼 필드 이름을 지정
- `form` : input이 속한 form. 없으면 가장 가까운 form으로 지정됨
- `onInvalid` : 폼 유효성 검사가 실패했을 경우 발생함
	- `required`와 `pattern`에 부합하지 않으면 발생함.
