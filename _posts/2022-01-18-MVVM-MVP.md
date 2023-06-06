---
title: MVVM, MVP 패턴
author: 신성일
date: 2022-01-18 14:00:00 +0900
categories: [study, design pattern]
tags: [architecture]
---

## MVVM

Model - View - View Model 의 약자로, 프로그램의 비지니스 로직과 프레젠테이션 로직을 UI로 명확하게 분리하는 패턴

![img1.daumcdn](/assets/img/2022-01-18-MVVM-MVP/img1.daumcdn-16424351335101.png)

### 구성요소

- Model : 데이터를 보관하고 있는 부분으로, 데이터를 불러오거나 업데이트하는 로직이 있음.
- View Model : Model에 데이터를 요청하고 가공함. 비지니스 로직을 처리. View로부터 입력을 받아 적절한 처리를 함
- View : UI 담당. UI에 연관된 로직만 수행한다. 필요한 데이터는 View Model과 data binding을 통해 얻는다. 사용자의 입력을 VM으로 전달.

### 특징

- View와 ViewModel이 n : 1 관계이다. 하나의 View Model에 여러 View가 결합이 가능하여 View Model의 재사용성이 높다.

### 장점

- 데이터를 관리하는 로직과 UI로직을 깔끔하게 분리하여 유지보수에 용이해짐.

- 코드의 재사용성을 개선
- UI 디자이너와 개발자가 쉽게 협업할 수 있는 디자인 패턴

### 단점

- View Model의 설계가 쉽지 않다.

### React와 MVVM 패턴

**Model**

- Redux를 통해 구현할 수 있음

```react
class Model {
    constructor() {
      books = [
        {id: 'RCB-123',name: "React Cook Book", isFavorite: false},
        {id: 'VCB-123',name: "Vue Cook Book", isFavorite: false},
        {id: 'ACB-123',name: "Angular Cook Book", isFavorite: false}
      ];
    }

    getBooks() {
        return this.books
    }

    toggleFavorite(bookId) {
      const target = this.books.filter(item => item.id === bookId)[0];
      target.isFavorite = !target.isFavorite
    }
}
```

**View Model**

- React-hooks와 dispatch를 통해 구현할 수 있음
- Model과 직접 상호 작용하며, View Model이 모델을 업데이트할 때마다, 모든 변경사항이 자동으로 View에 반영된다.

```react
class ViewModel {
    constructor(bookStore) {
        this.store = bookStore
    }

    getBooks() {
        return this.store.getBooks()
    }

    toggleFavorite(bookId) {
       this.store.toggleFavorite(bookId)
    }
}

export default ViewModel
```

**View**

- 비지니스 로직은 최대한 피하고, UI에 관련된 로직만 나타내도록 해야함.

  ```react
  const ViewComponent = ({books, handleToggleFavorite, handleClickNeverView}) => {
    return (
      <div>
        <BooksList
          books={books}
          handleToggleFavorite={handleToggleFavorite}
          handleClickNeverView={handleClickNeverView}
         />
      </div>
    )
  }
  ```

- View Controller

  - View Model에 대한 참조를 소유하고, View는 View Model을 인식하지 못하며 View controller에 의존하여 필요한 데이터 및 이벤트를 전달한다.

  ```react
  const ViewControllerComponent = ({viewModel}) => {
      const [isNeverView, setIsNeverView] = useState(false);

      const handleToggleFavorite = useCallback((bookId) => {
        viewModel.toggleFavorite(bookId)
      }, [viewModel]);

      const handleClickNeverView = useCallback(() => {
        setIsNeverView(!isNeverView);
      }, [isNeverView]);

      return (
        <ViewComponent
          books={viewModel.getBooks()}
          handleToggleFavorite={handleToggleFavorite}
          handleClickNeverView={handleClickNeverView}
        />
      )
  }
  ```

- Provider

  - MVVM의 일부는 아니지만, 모든 것을 이어 붙이기 위한 요소이다.
  - View Controller로 View Model을 인스턴스화하고, 필요한 모든 종속성을 전달해주기 위한 요소
  - ViewModel의 인스턴스는 props를 통해 View Controller 구성요소로 전달 된다.
  - 공급자는 모든 것을 연결하는 것이 목적이기에 논리없이 깨끗해야한다.

  ```react
  const BooksProvider = () => {
      const model = new Model()
      const [viewModel] = useState(new ViewModel(model))
  
      return (
      	<ViewControllerComponent viewModel={viewModel} />
      )
  }
  ```

## MVP

Model - View - Prensenter

![img1.daumcdn](/assets/img/2022-01-18-MVVM-MVP/img1.daumcdn.png)

### 구조

- Model : MVVM과 동일
- View : MVVM과 동일
- Prensenter : View에서 요청한 정보로 Model을 가공하여 View에 전달해주는 부분. View와 Model을 붙여주는 접착제 같은 역할이다.

### 동작

1. 사용자의 Action 들은 View를 통해 들어온다.
2. View는 데이터를 presenter로 요청한다.
3. presenter는 model에게 데이터를 요청한다.
4. model은 prensenter에게 데이터를 응답한다.
5. presenter는 view에게 데이터를 응답한다.
6. view는 presenter가 응답한 데이터로 화면을 꾸린다.

### 특징

Presenter는 View와 Model의 인스턴스를 가지고 있어 둘을 연결하는 접착제 같은 역할을 한다.

또, Prensenter와 View는 1:1 관계이다.

### 장점

- View와 Model 사이의 의존성이 없다.

### 단점

- View와 Prensenter 사이의 의존성이 높다.

## 나는 어떻게 하고 있지?

MVC, MVVM, MVP 등의 디자인 패턴을 공부하면서 그동한 무지성으로 개발했던 내 과거 프로젝트가 과연 어떤 디자인패턴에 가까운지 궁금해져 한번 과거 프로젝트를 살펴보기로 했다.

이글을 작성하고 있는 시점에서의 나의 가장 최근 프로젝트인 yourlist-web-renewal 프로젝트의 디렉터리 구조는 다음과 같다.

---

- components : 컴포넌트
  - elements : 공통으로 사용하는 작은 컴포넌트들
  - [컴포넌트명] - 주로 페이지 단위로 나뉨
    - index.tsx : 컴포넌트 최상단
    - container : 컴포넌트를 구성하는 작은 단위 컴포넌트들의 컨테이너
    - view : 컴포넌트를 구성하는 작은 단위 컴포넌트들의 뷰
    - elements : 컴포넌트 내에서 공통으로 자주 사용되는 작은 컴포넌트
- modules
  - index.ts : 모듈 최상단
  - moduleTypes.ts : 모듈에서 공통으로 사용되는 타입들
  - [모듈명]
    - index.ts : 모듈 최상단. 리듀서 포함
    - saga.ts : 모듈의 리덕스 사가 관련 코드 모음
    - action.ts : 액션 타입, 액션 생성 함수 등 포함
    - types.ts : 모듈에서 사용되는 타입들

---

간단히 디자인 패턴과 관련이 있는 부분만 적었다.

먼저 modules를 살펴보면, 앱에 필요한 데이터를 가지고 있고, 각종 api와 직접 소통하여 데이터를 받아들이고, 간단한 가공을 담당하는 부분이니 **Model** 에 해당한다고 볼 수 있다.

그리고 각 컴포넌트의 최상단에 있는 index.tsx 는 모델과 소통하여 데이터를 받아들이고, 그 컴포넌트에서 사용되는 비지니스 로직을 담당한다. 따라서 이는 MVP의 **Prensenter**나 MVVM의 **View Model**에 해당한다고 볼 수 있다.

container 는 Model과 대화하지 않고, 자신이 담당하는 view의 이벤트를 처리해주고, 자신이 담당하는 부분에서의 Model과 연관이 없는 비지니스 로직을 처리한다. 따라서 이는 MVVM 구조의 **View controller**에 가깝다고 보인다.

이때 내 프로젝트에서 view는 container나 index로 데이터를 요청하지 않는다. 상태를 통해 container 또는 index로부터 데이터를 받아서 그것을 활용하여 UI를 구성하고, UI에 관련된 로직만을 포함하고 있다. 이는 MVVM 구조에서의 **View**에 좀더 가깝다.

따라서 내 프로젝트는 굳이 따지자면, **MVVM** 구조를 따르고 있다고 볼 수 있다.

하지만 MVVM의 패턴에서 벗어난 부분도 존재한다.

1. index.tsx는 View Model과 Provieder를 동시에 겸하고 있다.
   - Model과 대화하며 각종 비지니스 로직을 처리하며(View Model), View Controller이나 View로 데이터를 보내준다(Provider)
   - 이는 각 컴포넌트 별로 하나의 거대한 View Model + Provider를 만들게 하여, View Model과 View과 1:n으로 연결되어, View Model을 재사용할 수 있다는 MVVM의 장점을 없앤다.
   - 또 view controller의 작업을 담당하고 있는 부분도 보인다.
2. 부분부분적으로 index.tsx, container에서 ui를 담당한 부분이 존재한다
   - 예를 들어, index.tsx에서 간단한 wrapper를 둘러싸서 view를 조합하는 등의 View 작업을 조금 맡아 하고 있다.
   - 이는 view의 작업을 분산시키는 행위로, 디자인 변경사항이 있을 시 수정을 까다롭게 하고 있다.

**개선 사항**

무의식적으로 했던 패턴이 MVVM에 가장 가깝다는 것을 알고, 나에게 MVVM이 가장 잘 맞는다는 것을 알았으나, MVVM 패턴을 지키지 않은 부분도 존재하기에 아래와 같이 개선할 점을 뽑아 보았다.

1. View Model을 분리
   - 재사용이 가능하도록 View Model을 index.tsx에서 분리시켜야한다.
2. index.tsx는 Provider만 담당하도록
3. View의 조합은 index.tsx에서 하더라도, wrapper는 왠만하면 view에서 담당하도록 하기

위와 같이 한다면 좀 더 구조적으로 안정된 프로젝트를 할 수 있을 것이라 생각된다.

하지만 아직 구체적으로 MVVM의 패턴이 실제 프로젝트에 어떻게 적용되는지는 잘 모르기에, 리팩토링을 할때 한번 MVVM 패턴이 지켜진 프로젝트를 참고해보는 것이 좋아보인다.

## 출처

https://velog.io/@dlrmsghks7/whatismvvmpattern

https://beomy.tistory.com/43
