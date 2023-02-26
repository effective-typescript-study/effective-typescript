# any 다루기

타입스크립트는 프로그램의 일부분에만 타입 시스템을 적용할 수 있다는 특성 덕분에 점진적으로 자바스크립트를 타입스크립트로 전환할 수 있다.

any가 남용되면 타입 체크의 의미를 퇴색시키기에 적절한 위치에서 사용해야한다.

---

## 아이템 38 any 타입은 가능한 한 좁은 범위에서만 사용하기

```jsx
interface Foo { foo: string; }
interface Bar { bar: string; }

declare function expressionReturningFoo(): Foo;
function processBar(b: Bar) { /* ... */ }

// 👎
function f1() {
  const x: any = expressionReturningFoo();  // Don't do this
  processBar(x);
	return x; // any type
}

// 👍
function f2() {
  const x = expressionReturningFoo();
  processBar(x as any);  // Prefer this
	return x; // Foo type
}
```

→ f1과 같이 x에 any타입을 부여하게 되면 함수의 리턴값까지 any를 반환하기 때문에 프로젝트에 전반적으로 영향을 끼치게 된다.

만약 타입 오류를 제거하고 싶다면 `@ts-ignore`을 사용하면 된다.

```jsx
function f1() {
  const x = expressionReturningFoo();
  // @ts-ignore
  processBar(x);
  return x;
}
```

```jsx
// 👎
const config: Config = {
  a: 1,
  b: 2,
  c: {
    key: value
  }
} as any;  // Don't do this!
```

→ 다른 속성들(a,b) 역시 타입체크가 되지 않는다.

```jsx
// 👍
const config: Config = {
  a: 1,
  b: 2,  // These properties are still checked
  c: {
    key: value as any // 최소한의 범위에서만 any 사용!
  }
};
```

---

## 아이템 39 any를 구체적으로 변형해서 사용하기

any 타입 : 모든 숫자, 문자열, 배열, 객체, 정규식, 함수, 클래스, DOM 엘리먼트, null , undefined

→ any를 더 구체적으로 표현할 수 있는 타입을 사용하자!

```jsx
// 👎
function getLengthBad(array: any) {
  // Don't do this!
  return array.length;
}

// 👍
function getLength(array: any[]) {
  return array.length;
}
```

함수의 매개변수가 객체이긴 하지만 값을 알 수 없다면 `{[key : string] : any}` 로 선언

- object 타입은 객체의 키를 열거할 수 있지만 속성에 접근할 수 없음

```jsx
// {[key : string] : any}
function hasTwelveLetterKey(o: **{[key: string]: any}**) {
  for (const key in o) {
    if (key.length === 12) {
      return true;
    }
  }
  return false;
}

// object
function hasTwelveLetterKey(o: **object**) {
  for (const key in o) {
    if (key.length === 12) {
      console.log(key, o[key]);
                   //  ~~~~~~ Element implicitly has an 'any' type
                   //         because type '{}' has no index signature
      return true;
    }
  }
  return false;
}
```

함수 타입에도 any를 구체화 해서 사용하자

```jsx
type Fn0 = () => any; // any function callable with no params
type Fn1 = (arg: any) => any; // With one param
type FnN = (...args: any[]) => any; // With any number of params
// same as "Function" type
```

---

## 아이템 40 함수 안으로 타입 단언문 감추기

프로젝트에 전반적으로 위험한 타입 단언문이 드러나 있는 것보다 제대로 타입이 정의된 함수 안으로 타입 단언문을 감추는 것이 더 좋다.

```jsx
declare function shallowEqual(a: any, b: any): boolean;
function shallowObjectEqual<T extends object>(a: T, b: T): boolean {
  for (const [k, aVal] of Object.entries(a)) {
    if (!(k in b) || aVal !== (**b as any**)[k]) {
      return false;
    }
  }
  return Object.keys(a).length === Object.keys(b).length;
}
```

---

## 아이템 41 any의 진화를 이해하기

변수의 타입은 변수를 선언할 때 결정되지만, any 타입과 관련해서 예외인 경우가 존재한다.

```jsx
function range(start: number, limit: number) {
  const out = []; // Type is any[]
  for (let i = start; i < limit; i++) {
    out.push(i); // Type of out is any[]
  }
  return out; // Type is number[]
}

const result = []; // Type is any[]
result.push("a");
result; // Type is string[]
result.push(1);
result; // Type is (string | number)[]
```

→ 배열에 값이 들어갈 때마다 배열의 타입이 확장(진화)된다.

```jsx
let val; // Type is any
if (Math.random() < 0.5) {
  val = /hello/;
  val; // Type is RegExp
} else {
  val = 12;
  val; // Type is number
}
val; // Type is number | RegExp
```

→ 조건문에서는 분기에 따라 타입이 변할 수도 있다.

any 타입의 진화는 `noImplicitAny` 가 설정된 상태에서 변수의 타입이 `암시적 any` 인 경우에만 일어난다.

```jsx
function makeSquares(start: number, limit: number) {
  const out = [];
  //    ~~~ Variable 'out' implicitly has type 'any[]' in some
  //        locations where its type cannot be determined
  if (start === limit) {
    return out;
    //     1️⃣ ~~~ Variable 'out' implicitly has an 'any[]' type
  }

  range(start, limit).forEach((i) => {
    out.push(i * i);
  });
  return out;
  // 2️⃣ ~~~ Variable 'out' implicitly has an 'any[]' type
}
```

→ 1️⃣ 어떤 변수가 `암시적 any`상태일 때 값을 읽을려고하면 오류가 발생한다.

→ 2️⃣ `암시적 any 타입` 은 함수호출( ex. forEach )을 거쳐도 진화하지 않는다. any를 진화시키려면 map, filter메서드를 통해 단일 구문으로 배열을 생성하면 된다.

타입을 안전하게 지키기 위해서는 암시적 any를 진화시키는 방식보다 `명시적 타입구문`을 사용해야한다.

---

## 아이템 42 모르는 타입의 값에는 any 대신 unknown을 사용하기

1️⃣ 함수의 반환값과 관련된 형태

호출한 곳에서 타입 선언을 생락하게 되면 암시적 any 가 반환되기 때문에 unknown을 반환하는 것이 더 안전하다

|       | any                                    | unknown                                 | never                                   |
| ----- | -------------------------------------- | --------------------------------------- | --------------------------------------- |
| 특징1 | 어떠한 타입이든 any타입에 할당 가능    | 어떠한 타입이든 unknown타입에 할당 가능 | 어떠한 타입이든 never타입에 할당 불가능 |
| 특징2 | any 타입은 어떠한 타입으로도 할당 가능 | unknown은 오직 unknown 과 any에만 할당  | 어떠한 타입으로도 할당 가능             |
| 문제  | 타입체커가 무용지물                    |                                         |                                         |

2️⃣ 변수 선언 관련된 형태

어떠한 값이 있지만 그 타입을 모르는 경우

```jsx
interface Feature {
  id?: string | number;
  geometry: Geometry;
  properties: **unknown**;
}
```

```jsx
function processValue(val: **unknown**) {
  if (val **instanceof** Date) {
    val  // Type is Date
  }
```

→ instanceof 를 체크한 후 unknown을 원하는 타입으로 변환

```jsx
function isBook(val: **unknown**): val **is** Book {
  return (
      typeof(val) === 'object' && val !== null &&
      'name' in val && 'author' in val
  );
}

function processValue(val: unknown) {
  if (isBook(val)) {
    val;  // Type is Book
  }
}
```

→ 사용자 정의 타입가드(is)로도 unknown을 원하는 타입으로 변환

3️⃣ 단언문과 관련된 형태

```jsx
function parseYAML(yaml: string): any {
  // ...
}
interface Foo { foo: string }
interface Bar { bar: string }

declare const foo: Foo;
let barAny = foo as any as Bar; // 👎
let barUnk = foo as **unknown** as Bar; // 👍
```

→ any는 분리되는 순간 영향력이 프로젝트 전체에 퍼짐, unknown 은 분리되는 경우 즉시 오류 발생

- {} 타입 : null 과 undefined를 제외한 모든 값을 포함
- object 타입 : 모든 비기본형(non-primitive)타입으로 이루어져있음. boolean, number, string은 포함되지 않지만 객체와 배열은 포함

---

## 아이템 43 몽키 패치보다는 안전한 타입을 사용하기

몽키 패치 : A monkey patch is a way for a program to **extend or modify** supporting system software locally (**affecting only the running instance** of the program)

자바스크립트에서 몽키 패치가 발생하는 경우는 객체와 클래스에 임의의 속성을 추가하는 경우

1️⃣  interface의 특수 기능 중 하나이 보강(argumentation)을 사용

```jsx
interface Document {
  /** Genus or species of monkey patch */
  monkey: string;
}

document.monkey = 'Tamarin';  // OK

// 모듈의 관점에서(타입스크립트 파일이 import/export 를 사용하는 경우)
export {};
declare global { // global cnrk
  interface Document {
    /** Genus or species of monkey patch */
    monkey: string;
  }
}
document.monkey = 'Tamarin';  // OK
```

→ 보강은 전역적으로 적용되기 때문에 코드의 다른 부분이나 라이브러리로부터 분리될 수 없다.

→ 애플리케이션이 실행되는 동안 속성을 할당하면 실행 시점에서 보강을 적용할 방법이 없다.

2️⃣ 더 구체적인 타입 단언문 사용

```jsx
interface MonkeyDocument **extends Document** {
  /** Genus or species of monkey patch */
  monkey: string;
}

(document as MonkeyDocument).monkey = 'Macaque';
```

## 아이템 44 타입 커버리지를 추적하여 타입 안정성 유지하기

noImplicitAny를 설정하고 모든 암시적 any 대신 명시적 타입 구문을 추가해도 `명시적 any 타입` 이나 `서드파티 타입 선언` 으로 인해 any가 존재할 수 있다.

따라서 `npm type-coverage` 를 활용해 any를 추적할 수 있다.

[https://github.com/plantain-00/type-coverage](https://github.com/plantain-00/type-coverage)
