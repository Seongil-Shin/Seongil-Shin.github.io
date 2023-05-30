---
title: 타입스크립트
author: 신성일
date: 2021-08-25 16:00:00 +0900
categories: [study, typescript]
tags: []
---

## 타입스크립트 연습

- tsconfig.json

  - compilerOptions:{

    ​ "outDir" : "./dist" // 컴파일된 파일 저장. expo 에선 안해도 될듯

    }

- jsx -> tsx, js -> ts

### 기본타입

```tsx
let mightBeUndefined: string | undefined = undefined; // string 일수도 있고 undefined 일수도 있음
let nullableNumber: number | null = null; // number 일수도 있고 null 일수도 있음

let color: "red" | "orange" | "yellow" = "red"; // red, orange, yellow 중 하나임
```

### interface, type

- interface, class 사용

  ```tsx
  interface Shape {
    getArea(): number;
  }
  class Circle implements Shape {
    radius: number; // private, public 설정으로 외부에서의 접근 제한 가능
    constructor(radius: number) {
      this.radius = radius;
    }
    getArea() {
      return this.radius * this.radius * Math.PI;
    }
  }

  const obj: Shape = new Circle(5); // Shape, Circle 모두 타입으로 넣을 수 있음

  console.log(obj.getArea());
  ```

- interface extends

  ```tsx
  interface Person {
    name: string;
    age?: number; // ? 는 있어도 되고 없어도 된다는 뜻.
    getAge(age: number): number; // 함수는 이렇게 해도 되고
    getAge: (age: number) => number; // 이렇게 해도 된다.
  }
  interface Developer extends Person {
    skills: string[];
  }
  ```

- Type alias

  ```tsx
  type Person = {
    name: string;
    age?: number;
  };
  type Developer = Person & {
    skills: string[];
  };
  ```

- interface 와 type 모두 객체를 위해 쓸 수 있는데, interface는 클래스를 위해, type은 일반 객체를 위해 쓰는 것이 좋다.

  - 근데 그냥 일관성있게만 써도 됨.

### generics

- 객체 A와 B를 합치는 함수에서 객체 A, B가 무슨 타입일 지 모를때 다음과 같이 any를 사용할 수 있다.

  ```tsx
  function merge(a: any, b: any): any {
    return {
      ...a,
      ...b,
    };
  }

  const merged = merge({ foo: 1 }, { bar: 1 });
  ```

- 하지만 위처럼하면 타입 추정을 깨버리기에 다음과 같이 generics를 사용해서 표현할 수도 있다

  ```tsx
  function merge<A, B>(a: A, b: B): A & B {
    return {
      ...a,
      ...b,
    };
  }

  const merged = merge({ foo: 1 }, { bar: 1 });
  /*
  	좀더 확실히 하기 위해 아래처럼 지정할 수도 있다.
  	const merged = merge<object, object>({foo:1}, {bar:1});
  */

  function wrap<T>(param: T) {
    return {
      param,
    };
  }

  const wrapped = wrap(10);
  ```

- interface, type에서도 사용가능하다

  ```tsx
  interface Items<T> {
    list: T[];
  }
  type Items<T> {
     list:T[];
  }

  const items: Items<string> = {
    list: ['a', 'b', 'c']
  };
  ```

- class 에서도 비슷한 방식으로 사용하면 된다.

  ```tsx
  class Queue<T> {
    list: T[] = [];
    get length() {
      return this.list.length;
    }
    enqueue(item: T) {
      this.list.push(item);
    }
    dequeue() {
      return this.list.shift();
    }
  }

  const queue = new Queue<number>();

  queue.enqueue(0);
  console.log(queue.dequeue());
  ```

## 리액트

- 합수 타입을 다음처럼 지정하면 됨

  ```tsx
  type GreetingsProps = {
    name: string;
    mark: string;
    optional?: string;
    onClick: (name: string) => void; // 아무것도 리턴하지 않는다는 함수를 의미합니다.
  };

  function Greetings({ name, mark, optional, onClick }: GreetingsProps) {
    const handleClick = () => onClick(name);
    return (
      <div>
        Hello, {name} {mark}
        {optional && <p>{optional}</p>}
        <div>
          <button onClick={handleClick}>Click Me</button>
        </div>
      </div>
    );
  }

  function App() {
    const onClick = (name: string) => {
      console.log(`${name} says hello`);
    };
    return <Greetings name="Hello" onClick={onClick} />;
  }
  ```

- useState

  ```tsx
  const [count, setCount] = useState<number>(0);
  ```

  - 그런데 useState에선 알아서 타입을 유추하기에 안써도 된다.
  - 다만 상태가 null일 수도 있고, 아닐수도 있을때 Generics를 사용한다.

  ```tsx
  type Information = { name: string; description: string };
  const [info, setInformation] = useState<Information | null>(null);
  ```

- onPress, onChange 등의 객체타입이 뭔지 모를때는, 커서를 onChange 위에 올려두면 무엇인지 알려준다.

## 리덕스

- 리덕스를 사용할땐 @types/react-redux 를 해줘야 타입스크립트가 지원이 된다.
- typesafe-actions 라는 라이브러리로 쉽게 redux 파일을 작성할 수 있다.
