
https://youtu.be/g9FL8hKoNqE?si=tsCYUjwOX5Z0AqZk


## 분석

Typescript 옵션으로 성능 문제 진단
- `extendedDiagnostics` 로 대략적인 컴파일 시간 확인 가능
- `generateTrace` 로 트레이스 파일을 출력할 수 있음. 이 파일을 툴에 넣어 확인 가능
	- trace.json : 컴파일에 소요된 시간 중 오래 걸리는 부분을 출력.
	- types.json : 컴파일러가 인식한 타입의 목록을 출력.

툴을 사용해서 성능 문제 진단
- `@typescript/analyze-trace` : CLI로 trace를 분석하여 hot spot 식별 가능
- https://ui.perfetto.dev/ : GUI로 상세한 확인이 가능


## 개선

 `keyof JSX.IntrinsicElements;`은 175개의 태그 이름이 들어간 union type. 따라서 Row를 정의하는 순간 175개의 태그를 정의할 수 있는 타입이 정의됨

```ts
type Elements = keyof JSX.IntrinsicElements;

type FlexProps<E extends Elements> = E extends Elements
	? JSX.IntrinsicElements & { as?: E; gap?: number;}
	: never;

export const Row = <E extends Elements>({ as, ...props }:
									   PropsWithChildren<FlexProps<E>>) => {
	const &Element = as as React.ElementsType
	return <$Element {...props} />; 
}
```


영상의 해결책으론 두가지 방법을 제시함

### Type Narrowing

 일부 태그만 지정하여 타입의 범위를 줄임

```ts
type Elements = "div" | "section" | "label";
```

### Dynamic inference

제네릭을 통해 필요한 태그만 지정해서 사용할 수 있도록 함. Type Narrowing 보다 다양한 태그에 대응가능

```ts
type Elements = keyof JSX.IntrinsicElements;

type ElementType<P = any> = {
	[K in Elements]: P extends JSX.IntrinsicElements[K]
}[Elements] | ((props: P) => JSX.Element);

type ElementProps<E extends ElementType> = E extends Elements ? JSX.IntrinsicElements[E] : {};

type FlexProps<E extends ElementType> = {as?: E};

type Props<E extends ElementType> = FlexProps<E> & Omit<ElementProps<E>, keyof FlexProps<E>>;

export const Row = <E extends ElementType>({ as, ...props }: Props<E>) => {
	const &Element = as || "div";
	return <$Element {...props} />; 
}
```

위 코드를 아래와 같이 개선할 수 있음

```ts
import { ComponentProps, ElementType } from "react";

type FlexProps<E extends ElementType> = {as?: E};

type Props<E extends ElementType> = FlexProps<E> & Omit<ComponentProps<E>, keyof FlexProps<E>>;

export const Row = <E extends ElementType>({ as, ...props }: Props<E>) => {
	const &Element = as || "div";
	return <$Element {...props} />; 
}
```


## 그외 개선법

타입스크립트에서 권장하는 컴파일 되기 쉬운 코드 작성법 ([링크](https://github.com/microsoft/TypeScript/wiki/Performance#writing-easy-to-compile-code))
- Preferring Interfaces Over Intersections
- Using Type Annotations
- Preferring Base Types Over Unions
- Naming Complex Types



