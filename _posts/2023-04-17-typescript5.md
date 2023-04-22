---
title: Typescript 5.0
author: 신성일
date: 2023-04-17 22:30:00 +0900
categories: [study, typescript]
tags: [ts, typescript]
---

typescript 5.0이 나왔다고 하여 간단히 어떤 기능들이 추가되었는지 살펴보았다. 자세한 내용은 [Typescript 5.0 번역](https://velog.io/@hustle-dev/TypeScript-5.0-%EB%B2%88%EC%97%AD#%EB%8D%B0%EC%BD%94%EB%A0%88%EC%9D%B4%ED%84%B0)에서 확인 가능하다.



## 데코레이터

데코레이터는 재사용 가능한 방식으로 클래스와 그 멤버를 사용자 정의하는 ECMAScript의 기능이다. 데코레이터는 이전부터 지원되었으나 Typescript 5.0부터는 공식 기능으로 지원한다.

```typescript
function loggedMethod(originalMethod: any, context: ClassMethodDecoratorContext) {
    const methodName = String(context.name);

    function replacementMethod(this: any, ...args: any[]) {
        console.log(`LOG: Entering method '${methodName}'.`)
        const result = originalMethod.call(this, ...args);
        console.log(`LOG: Exiting method '${methodName}'.`)
        return result;
    }

    return replacementMethod;
}

class Person {
    name: string;
    constructor(name: string) {
        this.name = name;
    }

    @loggedMethod
    greet() {
        console.log(`Hello, my name is ${this.name}.`);
    }
}

const p = new Person("Ron");
p.greet();
```

## const 타입 파라미터

특정 문자열로 타입을 구성하고 싶을 때, 주로 `as const`를 사용하였다.

```ts
// The type we wanted:
//    readonly ["Alice", "Bob", "Eve"]
// The type we got:
//    string[]
const names1 = getNamesExactly({ names: ["Alice", "Bob", "Eve"]});

// Correctly gets what we wanted:
//    readonly ["Alice", "Bob", "Eve"]
const names2 = getNamesExactly({ names: ["Alice", "Bob", "Eve"]} as const);
```

하지만 이제는 제네릭에 `const`를 추가하여 비슷한 효과를 얻을 수 있다.

```ts
type HasNames = { names: readonly string[] };
function getNamesExactly<const T extends HasNames>(arg: T): T["names"] {
//                       ^^^^^
    return arg.names;
}

// Inferred type: readonly ["Alice", "Bob", "Eve"]
// Note: Didn't need to write 'as const' here
const names = getNamesExactly({ names: ["Alice", "Bob", "Eve"] });
```

<br/>

## extends에서 여러 파일 가져오기 가능

typescript 5.0부터는 다음과 같이 `tsconfig.json`의 `extends`에서 여러 파일을 작성할 수 있다. 따라서 여러 파일을 확장할 수 있게 된다.

```json
{
    "extends": ["a", "b", "c"],
    "compilerOptions": {
        // ...
    }
}
```

c는 b를 확장하고, b는 a를 확장하는 것과 비슷한 효과를 가진다. 따라서 설정이 충돌되는 경우 후자의 항목이 우선된다.

<br/>

## 모든 enum은 유니온 enum이다.

typescript 2.0에서 enum literal type이 도입되면서 enum은 좀 더 특별해졌다. enum literal type은 각 enum 멤버에 고유한 타입을 부여하고 enum 자체를 각 멤버타입의 union으로 만들었다.

```typescript
// Color is like a union of Red | Orange | Yellow | Green | Blue | Violet
enum Color {
    Red, Orange, Yellow, Green, Blue, /* Indigo */, Violet
}

// Each enum member has its own type that we can refer to!
type PrimaryColor = Color.Red | Color.Green | Color.Blue;

function isPrimaryColor(c: Color): c is PrimaryColor {
    // Narrowing literal types can catch bugs.
    // TypeScript will error here because
    // we'll end up comparing 'Color.Red' to 'Color.Green'.
    // We meant to use ||, but accidentally wrote &&.
    return c === Color.Red && c === Color.Green && c === Color.Blue;
}
```

이때 문제는 멤버 타입이 실제 값과 일부 연관되어있다는 것이다. 그래서 멤버가 함수 호출로 초기화되면 값을 계산할 수 없는 경우가 있다.

```typescript
enum E {
    Blah = Math.random()
}
```

Typescript 5.0 은 계산된 각 멤버에 대해 고유한 타입을 생성하여 모든 enum을 공용체 enum으로 만들 수 있다. 

```typescript
// typescript 5.0
const a : E.Blah = 10;		// 문제 없음
// typescript 5.0 미만
const a : E.Blah = 10; 		// Enum type 'E' has members with initializers that are not literals.
```

<br/>

## --moduleResolution bundler

typesciprt 4.7에는 `--module`, `--moduleResolution`에 `node16`, `nodenext` 옵션을 도입하여 ECMAScript 모듈에 대한 정확한 조회 규칙을 모델링할 수 있게 되었다. 하지만 이는 몇몇 개발자에겐 번거로운 설정이었다. 

5.0에서는 compilerOptionsdp `moduleResolution`에 `bundler` 옵션을 추가하여 번들러의 동작을 모델링할 수 있다.

```json
{
    "compilerOptions": {
        "target": "esnext",
        "moduleResolution": "bundler"
    }
}
```

하이브리드 조회 전략을 구현하는 Vite, esbuild, swc, Webpack, Parcel 등의 최신 번들러를 사용 중이라면 새로운 `bundler`옵션이 적합할 것이다. 하지만, 라이브러리 제작자일 경우 `bundler` 옵션을 사용하면, 번들러를 사용하지 않는 사용자에게 문제가 생길 수 있으니 `node16`, `nodenext` 옵션을 추천한다.

<br/>

## Resolution 커스터마이징 플래그

- allowImportingTsExtensions : `.ts`, `.mts`, `.tsx`와 같이 Typescript 전용 확장자를 사용하여 서로를 import 할 수 있다.
- resolvePackageJsonExports : typescript가 `node_modules`의 패키지를 읽을 경우, package.json의 exports 필드를 참조하도록 한다.
- resolvePackageJsonImports : package.json을 포함하는 조상 디렉터리의 파일로부터 `#`으로 시작하는 조회를 수행할 때, typescript가 package.json의 import 플드를 참조하도록 강제함
- allowArbitraryExtensions  
  - typescript 5.0에서 import 경로가 js또는 ts 파일 확장자가 아닌 경우 `{파일 기본 이름}.d.{확장자}.ts` 형식의 해당 경로에서 타입을 찾는다. 
  - Typescript에서는 이 파일 형식을 이해하지 못하여 런타임에서 import를 지원하지 않을 수 있음을 알리는 오류를 내지만, 이 플래그를 통해 억제 할 수 있다.

<br/>

## customConditions

package.json의 `exports`, `imports` 필드를 Typescript가 리졸브할 때 성공해야하는 추가 조건의 목록을 받는다. 현존하는 조건에 추가된다.

```json
{
    "compilerOptions": {
        "target": "es2022",
        "moduleResolution": "bundler",
        "customConditions": ["my-condition"]
    }
}
```

```json
{
    // ...
    "exports": {
        ".": {
            "my-condition": "./foo.mjs",
            "node": "./bar.mjs",
            "import": "./baz.mjs",
            "require": "./biz.mjs"
        }
    }
}
```

위와 같은 경우 `exports`, `imports` 필드를 리졸브할 때 항상 `my-condition`을 고려한다.

 <br/>

## --verbatimModuleSyntax

기본적으로 Typescript는 `import elision`을 수행한다. 예를들어 다음과 같이 `import`를 type을 가져오기만을 위해 사용한다면, 이를 지운다.

```typescript
import { Car } from "./car";

export function drive(car: Car) {
    // ...
}
```

```javascript
// import 문을 지움
export function drive(car) {
    // ...
}
```

type은 값이 아니기에 지우지 않으면 런타임 에러가 날 수 있기 때문이다. 하지만 몇가지 엣지 케이스가 있을 수 있다. 예를들어 변환된 코드는 `import ".car"`조차 남기지 않는데, 이는 사이드 이펙트를 일으킬 수도 있다.

따라서 Typescript에서는 `type` 수정자를 사용하도록 하는 `--importsNotUsedAsValues`, 일부 모듈 엘리션 동작을 방지하는 `--perserveValueImports`, Typescript가 여러 컴파일러에서 작동하는지 확인하는 `--isolatedModules` 플래그가 있다. 하지만 이 플래그들은 이해하기 어렵고, 에지 케이스 역시 존재한다.

Typescript 5.0에서는 `--verbatimModuleSyntax` 를 적용하여 `type` 수정자가 없는 모든 import, export는 그대로 유지하도록 했다. `type` 수정자를 사용하는 모든 항목은 완전히 삭제된다.

```typescript
// Erased away entirely.
import type { A } from "a";

// Rewritten to 'import { b } from "bcd";'
import { b, type c, type d } from "bcd";

// Rewritten to 'import {} from "xyz";'
import { type xyz } from "xyz";
```

하지만 이 플래그를 사용하면 ECMAScript `import`와 `export`는, 다른 모듈 시스템을 암시하는 파일 확장자 또는 세팅이 있을때 `require` 로 재작성되지 않는다. 대신 오류를 낸다. 만약 `require`나  `module.exports`를 사용하는 코드를 emit 하고 싶다면 ES2015 이전의 Typescript 모듈 구문을 사용해야한다.

| Input Typescript                                             | Output Javascript                                            |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| `import foo = require("foo"); `                              | `const foo = require("foo"); `                               |
| **function** **foo**() {} <br />**function** **bar**() {} <br />**function** **baz**() {} <br />**export** = {    foo,    bar,    baz }; | **function** **foo**() {} <br />**function** **bar**() {} <br />**function** **baz**() {} <br />module.exports = {    foo,    bar,    baz }; |

이는 몇몇 상황에서 도움이 되는데, 예를들어 개발자가 `package.json`의 `--module node16`을 설정하지 않은 경우가 있다. 개발자는 인지하지 못한채 ES 모듈 대신 CommonJS 모듈을 작성하기 시작하여 예상치 못한 조회 규칙과 JavaScript 출력을 제공하게 된다. 이 플래그는 구문이 의도적으로 다르기 때문에 사용중인 파일 형식에 대해 의도적으로 확인할 수 있다.

`-verbatimModuleSyntax`은 `--importsNotUsedAsValues` 및 `--preserveValueImports`보다 더 일관된 스토리를 제공하므로, 기존의 두 플래그는 이 구문으로 대체된다.

<br/>

## Support for export type *

기존엔 `export * from "module"`이나 `export * as ns from "module"` 처럼 re-export가 불가능했다. Typescript 5.0 부터는 가능하다.

```typescript
// models/vehicles.ts
export class Spaceship {
  // ...
}

// models/index.ts
export type * as vehicles from "./vehicles";

// main.ts
import { vehicles } from "./models";

function takeASpaceship(s: vehicles.Spaceship) {
  // ✅ ok - `vehicles` only used in a type position
}

function makeASpaceship() {
  return new vehicles.Spaceship();
  //         ^^^^^^^^
  // 'vehicles' cannot be used as a value because it was exported using 'export type'.
}
```

<br/>

## `@satisfies` Support in JSDoc

Typescript 4.9에서 `satisfies` 연산자가 추가됐다. 이 연산자는 타입에 영향을 주지 않고, 표현식이 타입에 호환되는지만 확인한다. 

```typescript
interface CompilerOptions {
    strict?: boolean;
    outDir?: string;
    // ...
}

interface ConfigSettings {
    compilerOptions?: CompilerOptions;
    extends?: string | string[];
    // ...
}

let myConfigSettings = {
    compilerOptions: {
        strict: true,
        outDir: "../lib",
        // ...
    },

    extends: [
        "@tsconfig/strictest/tsconfig.json",
        "../../../tsconfig.base.json"
    ],

} satisfies ConfigSettings;
```

위 코드에서 Typescript는 `myConfigSettings.extends`가 배열임을 아는데, 이는 `ConfigSettings`와 호환 가능한지만 확인하고, `myConfigSettings`의 타입을 변경하지는 않았기 떄문이다. 따라서 `extends`에 map 함수를 사용해도 오류가 나지 않는다.

이 `satisfies`가 JSDoc 태그에 추가되었다.

```typescript
// @ts-check

/**
 * @typedef CompilerOptions
 * @prop {boolean} [strict]
 * @prop {string} [outDir]
 */

/**
 * @satisfies {CompilerOptions}
 */
let myCompilerOptions = {
    outdir: "../lib",
//  ~~~~~~ oops! we meant outDir
};
```

다음과 같이 `/** @satisfies */ `인라인으로 사용하거나, 함수 호출과 같은 곳에서도 사용할 수 있다.

```typescript
let myConfigSettings = /** @satisfies {ConfigSettings} */ ({
    compilerOptions: {
        strict: true,
        outDir: "../lib",
    },
    extends: [
        "@tsconfig/strictest/tsconfig.json",
        "../../../tsconfig.base.json"
    ],
});

// 함수 호출에서 사용
compileCode(/** @satisfies {ConfigSettings} */ ({
    // ...
}));
```

<br/>

## `@overload` Support in JSDoc

Typescript에서는 함수 오버로드를 만들 수 있다. 이로써 함수를 사용하는 방법을 제한할 수 있다.

```typescript
// Our overloads:
function printValue(str: string): void;
function printValue(num: number, maxFractionDigits?: number): void;

// Our implementation:
function printValue(value: string | number, maximumFractionDigits?: number) {
    if (typeof value === "number") {
        const formatter = Intl.NumberFormat("en-US", {
            maximumFractionDigits,
        });
        value = formatter.format(value);
    }

    console.log(value);
}
```

이러한 오버로드 기능을 `@overload` 태그를 사용해서 JSDoc 안에서 사용할 수 있다.

```typescript
// @ts-check

/**
 * @overload
 * @param {string} value
 * @return {void}
 */

/**
 * @overload
 * @param {number} value
 * @param {number} [maximumFractionDigits]
 * @return {void}
 */

/**
 * @param {string | number} value
 * @param {number} [maximumFractionDigits]
 */
function printValue(value, maximumFractionDigits) {
    if (typeof value === "number") {
        const formatter = Intl.NumberFormat("en-US", {
            maximumFractionDigits,
        });
        value = formatter.format(value);
    }

    console.log(value);
}
```

<br/>

## Passing Emit-Specific Flags Under `--build`

`--build` 모드에서 다음 플래그가 추가되었다. 빌드 시 커스터마이징을 위해 사용한다.

- `--declaration`
- `—-emitDeclarationOnly`
- `—-declarationMap`
- `—-sourceMap`
- `—-inlineSourceMap`

<br/>

## Case-Insensitive Import Sorting in Editors

VSCode 와 같은 코드에디터에서 TypeScript는 import, export 목록 정렬을 지원한다. 하지만 Typescript는 기본적으로 대소문자를 구분하여 정렬한다.  아스키 문자 인코딩 상 대문자는 소문자 앞이기에 다음 정렬은 Typescript에서 적절하다.

```typescript
import {
    Toggle,
    freeze,
    toBoolean,
} from "./utils";
```

하지만 대소문자를 구분하지 않고 정렬하고 싶을 수도 있다. 이제는 `organizeImports` 옵션이 추가되어 해당 설정을 할 수 있다.

```json
{
    "typescript.unstable": {
        // Should sorting be case-sensitive? Can be:
        // - true
        // - false
        // - "auto" (auto-detect)
        "organizeImportsIgnoreCase": "auto",

        // Should sorting be "ordinal" and use code points or consider Unicode rules? Can be:
        // - "ordinal"
        // - "unicode"
        "organizeImportsCollation": "ordinal",

        // Under `"organizeImportsCollation": "unicode"`,
        // what is the current locale? Can be:
        // - [any other locale code]
        // - "auto" (use the editor's locale)
        "organizeImportsLocale": "en",

        // Under `"organizeImportsCollation": "unicode"`,
        // should upper-case letters or lower-case letters come first? Can be:
        // - false (locale-specific)
        // - "upper"
        // - "lower"
        "organizeImportsCaseFirst": false,

        // Under `"organizeImportsCollation": "unicode"`,
        // do runs of numbers get compared numerically (i.e. "a1" < "a2" < "a100")? Can be:
        // - true
        // - false
        "organizeImportsNumericCollation": true,

        // Under `"organizeImportsCollation": "unicode"`,
        // do letters with accent marks/diacritics get sorted distinctly
        // from their "base" letter (i.e. is é different from e)? Can be
        // - true
        // - false
        "organizeImportsAccentCollation": true
    },
    "javascript.unstable": {
        // same options valid here...
    },
}
```

<br/>

## **Exhaustive `switch`/`case` Completions**

switch 문을 작성할 때 검사 대상에 리터럴 타입이 있는지 감지한다. 감지되면 발견된지 않은 각 대소문자를 스카폴딩하는 완결성을 제공한다.

```typescript
type Fruit = 
	| { kind : "apple" }
	| { kind : "orange" }
	| { kind : "banana" }

function nom(fruit: Fruit) {
	switch(fruit.kind) {
    case 						// case 문을 작성하면 자동 완성으로 case "apple": case "orange": case "banana"가 생김
  }
}
```

<br/>

## Speed, Memory, and Package Size Optimizations

Typescript 5.0에서는 코드 구조, 데이터 구조, 알고리즘 구현을 변경하여 전체 경험을 좀 더 빠르게 만들었다.

![image](/assets/image.png)

![img](/assets/image-20230421235917311.png)

<br/>

## Breaking Changes and Deprecations

### Runtime Requirements

Typescript 5.0부터는 ECMAScript 2018을 대상으로 한다. 또한, Typescipt 패키지는 최소 요구 엔진을 12.20으로 설정했다.

### `lib.d.ts` Changes

DOM 유형이 생성되는 방식이 변경되어 기존 코드에 영향을 미칠 수 있다. 특히 특정 property가 숫자에서 숫자 리터럴 타입으로 변환되었다. 잘라내기, 복사, 붙여넣기 이벤트 처리를 위한 프로퍼티와 메서드가 인터페이스 전반으로 이동되었다.

### API Breaking Changes

모듈로 전환하고, 불필요한 인터페이스를 제거했으며 일부 정확성을 개선했다.

### Forbidden Implicit Coercions in Relational Operators

Typescript의 특정 연산은 암시적으로 문자열을 숫자로 강제 변환할 수 있는 코드를 작성할 경우 경고하고 있다.

```typescript
function func(ns: number | string) {
  return ns * 4; // Error, possible implicit coercion
}
```

5.0 부터는 관계 연산자 >, <, <=, >= 에도 이 기능이 적용된다. 원하는 경우 `+`를 사용하여 피연산자를 숫자로 명시적으로 강제할 수 있다.

```typescript
function func(ns: number | string) {
  return ns > 4; // Now also an error
}

function func(ns: number | string) {
  return +ns > 4; // OK
}
```

### Enum Overhaul

Enum을 사용할 때 Enum 유형에 도메인 외부 리터럴을 할당하면 오류가 발생하도록 추가됐다.

```typescript
enum SomeEvenDigit {
    Zero = 0,
    Two = 2,
    Four = 4
}

// Now correctly an error
let m: SomeEvenDigit = 1;
```

또 숫자와 간접 string enum 참조가 혼합되어 선언된 값이 있는 enum에서 오류를 제대로 생성하도록 변경됐다.

```typescript
enum Letters {
    A = "a"
}
enum Numbers {
    one = 1,
    two = Letters.A
}

// Now correctly an error
const t: number = Numbers.two;
```

### **More Accurate Type-Checking for Parameter Decorators in Constructors Under `--experimentalDecorators`**

decorator에 대한 좀 더 정확한 type-checking이 되도록 하는 옵션이 추가되었다. 특히 생성자 파라미터에서 사용할 때 효과적이다.

```typescript
export declare const inject:
  (entity: any) =>
    (target: object, key: string | symbol, index?: number) => void;

export class Foo {}

export class C {
    constructor(@inject(Foo) private x: any) {
    }
}
```

`key`가 `string | symbol`을 예상하기에 이 호출은 실패하지만 생성자 파라미터는 `undefined`키를 받는다. 올바른 수정은 `inject`의  `key`의 타입을 변경하는 것이다. 만약 변경할 수 없는 라이브러리를 사용중인 경우에는 `inject`를 좀 더 type-safe한 decorator로 감싸고  `key`에 type assertion을 사용하는 것이다.

### **Deprecations and Default Changes**

이제 다음 설정값들은 사용되지 않는다.

- `--target: ES3`
- `--out`
- `--noImplicitUseStrict`
- `--keyofStringsOnly`
- `--suppressExcessPropertyErrors`
- `--suppressImplicitAnyIndexErrors`
- `--noStrictGenericChecks`
- `--charset`
- `--importsNotUsedAsValues`
- `--preserveValueImports`
- `prepend` in project references

Typescript 5.5까지는 허용하지만 이후로는 경고가 발생한다. 단, `"ignoreDeperecations": "5.0"`을 사용하면 경고를 없앨 수 있다. 

프로젝트 내에서 동일한 파일 이름을 참조하는 모든 참조가 케이스에 대해 동의하도록 보장하는 `--forceConsistentCasingInFileNames`는 이제 `true`로 기본 설정된다.



<Br/>

## 출처

https://velog.io/@hustle-dev/TypeScript-5.0-%EB%B2%88%EC%97%AD#%EB%8D%B0%EC%BD%94%EB%A0%88%EC%9D%B4%ED%84%B0

https://devblogs.microsoft.com/typescript/announcing-typescript-5-0/#moduleresolution-bundler