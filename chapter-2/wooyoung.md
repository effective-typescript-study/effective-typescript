# 타입스크립트의 타입 시스템

## 아이템 6 편집기를 사용하여 타입 시스템 탐색하기

타입스크립트를 설치하면, 다음 두가지를 실행할 수 있다.

- 타입스크립트 컴파일러(tsc)
- 단독으로 실행할 수 있는 타입스크립트 서버(tsserver) - 편집기에서 사용하는 언어 서비스 제공

편집기를 통해 특정 시점의 추론된 타입 값을 알 수 있다.

---

## 아이템 7 타입이 값들의 집합이라고 생각하기

타입 : 할당 가능한 값들의 집합, 범위

- 유닛(unit)타입, 리터럴(literal) 타입 : 한가지의 타입만 포함
- 유니온(union) : 값 집합들의 합집합

```tsx
interface Person {
  name: string;
}
interface Lifespan {
  birth: Date;
  death?: Date;
}

type PersonSpan = Person & Lifespan;

const person: PersonSpan = {
  name: "Alan Turing",
  birth: new Date("1912/06/23"),
  death: new Date("1954/06/07"),
}; // OK
```

타입 연산자는 인터페이스의 속성이 아닌, `값의 집합(타입의 범위)에 적용`된다. 그리고 추가적인 속성을 가지는 값도 여전히 그 타입에 속한다.

Person와 LifeSpan의 인터섹션은 Person의 범위와 LifeSpan의 범위의 인터섹션이다. 그래서 Person과 Lifespan을 둘 다 가지는 값은 인터섹션(&)에 속하게 된다.

name, birth, death 보다 더 많은 속성을 가지는 값도 PersonSpan 타입에 속한다.

```tsx
type PersonSpan = Person | Lifespan; // Type is never
```

유니온 타입에 속하는 값은 어떤 키도 없기에 유니온에 대한 keyof는 공집합(never)이다.
타입스크립트에서는 PersonSpan에 Person이 올지 Lifespan이 올지 모르니까 두 타입의 공통 속성에만 접근가능하다.

```tsx
keyof (A&B) = (keyof A) | (keyof B)
keyof (A|B) = (keyof A) & (keyof B)
```

상속을 구현할때는 extends 키워드를 사용한다.

---

## 아이템 8 타입 공간과 값 공간의 심벌 구분하기

타입스크립트의 심벌(symbol)은 타입 공간이나 값 공간 중의 한 곳에 존재한다.

- 심벌(symbol) : 변경 불가능한 원시값, symbol 함수를 호출하면 이름이 같더라도매번 고유한 심볼 생성

```jsx
iinterface Cylinder { // 타입
  radius: number;
  height: number;
}

const Cylinder = (radius: number, height: number) => ({radius, height}); // 값

function calculateVolume(shape: unknown) {
  if (shape instanceof Cylinder) {
    shape.radius
       // ~~~~~~ Property 'radius' does not exist on type '{}'
  }
}
```

**instanceof**는 자바스크립트의 런타임 연산자이고, 값에 대해서 연산을 한다.

한 심벌이 타입인지 값인지 확인하기 위해서는 문맥으로 파악해야한다.

일반적으로 type이나 interface, 타입선언(:), 단언문(as) 다음에 오는 심벌은 타입인 반면, const나 let에 쓰이는 거나 = 다음에 나오는 것은 값이다.

**class**와 **enum**은 상황에 따라서 타입과 값 두가지 모두 가능한 예약어이다.

**typeof**는 타입에서 쓰일 때와 값에서 쓰일 때 다른 기능을 한다.

```jsx
// 타입의 관점 : 값을 읽어서 타입을 반환
type T1 = typeof p; // 타입은 Person
type T2 = typeof email; // 타입은 (p: Person, subject: string, body: string) => Response

// 값의 관점 : 자바스크립트 런타임의 typeof 연산자, 대상 심벌의 런타임 타입을 가리키는 문자열을 반환
const v1 = typeof p; // 값은 "object"
const v2 = typeof email; // 값은 "function"
```

---

## 아이템 9 타입 단언보다는 타입 선언 사용하기

|                | 타입 선언                                       | 타입 단언                                         |
| -------------- | ----------------------------------------------- | ------------------------------------------------- |
| 타입 부여      | const alice : Person = { name : ‘Alice’ }       | const bob = { name : ‘bob’ } as Person            |
| 타입 결과      | Person                                          | Person                                            |
| 타입 체커      | 할당되는 값이 해당 인터페이스를 만족하는지 검사 | 강제로 타입을 지정한 거라 타입 체크 오류를 무시함 |
| 타입 속성 추가 | 불가능, 잉여 속성 체크 동작                     | 가능                                              |
| 사용법         | 화살표함수의 리턴 타입 명시할때                 | 개발자가 판단하는 타입이 더 정확할 때             |
| ex. DOM타입    |

- 잉여 속성 체크 : 타입이 명시된 변수에 `객체 리터럴` 을 할당할때 타입스크립트는 해당 타입의 속성이 있는지, 그 외의 속성은 없는지 확인(p.61)

---

## 아이템 10 객체 레퍼 타입 피하기

래퍼객체(wrapper object) 1. Number 2. String 3. Boolean

자바스크립트는 기본형을 String 객체로 래핑(wrap)하고, 메서드를 호출하고, 마지막에 래핑한 객체를 버린다.

- 몽키패치 : 런타임 프로그램의 어떤 기능을 수정해서 사용하는 기법, js에서는 주로 프로토타입을 변경하는 것이 해당된다.
  - 유래 : 게릴라 패치 → 고릴라 패치(발음의 유사성) → 몽키 패치(고릴라가 너무 거대하고 위협적으로 들려 작고 귀여운 몽키로 )

타입스크립트는 기본형과 객체 레퍼 타입을 별도로 모델링한다.

기본형 타입을 사용하자!

---

## 아이템 11 잉여 속성 체크의 한계 인지하기

잉여 속성 체크 : 객체 리터럴을 변수에 할당하거나 함수에 매개변수로 전달할 때 “그 외의 속성은 없는지” 확인

```jsx
interface Room {
  numDoors: number;
  ceilingHeightFt: number;
}
const r: Room = {
  numDoors: 1,
  ceilingHeightFt: 10,
  elephant: "present",
  // ~~~~~~~~~~~~~~~~~~ Object literal may only specify known properties,
  //                    and 'elephant' does not exist in type 'Room'
};
```

잉여 속성 체크 ≠ 할당 가능 검사 이다. 위 예제 코드를 실행하면 런타임에서는 오류가 발생하지 않는다.

잉여 속성 체크는 구조적 타이핑 시스템에서 허용되는 속성의 오타 같은 실수를 잡는데 효과적이다.

```jsx
interface LineChartOptions {
  logscale?: boolean;
  invertedYAxis?: boolean;
  areaChart?: boolean;
}
const opts = { logScale: true };
const o: LineChartOptions = opts;
// ~ Type '{ logScale: boolean; }' has no properties in common
//   with type 'LineChartOptions'
```

타입 단언(as)을 사용하면 잉여 속성 체크가 되지 않는다.

---

## 아이템 12 함수 표현식에 타입 적용하기

```jsx
function rollDice1(sides: number): number {
  /* COMPRESS */ return 0; /* END */
} // 문장
const rollDice2 = function (sides: number): number {
  /* COMPRESS */ return 0; /* END */
}; // 표현식
const rollDice3 = (sides: number): number => {
  /* COMPRESS */ return 0; /* END */
}; // 표현식
```

함수 표현식을 사용하면 함수의 매개변수부터 반환값까지 전체를 함수 타입으로 선언하여 함수 표현식에 재사용할 수 있다.

```jsx
function add(a: number, b: number) {
  return a + b;
}
function sub(a: number, b: number) {
  return a - b;
}
function mul(a: number, b: number) {
  return a * b;
}
function div(a: number, b: number) {
  return a / b;
}

type BinaryFn = (a: number, b: number) => number; // 하나의 타입구문만 사용
const add: BinaryFn = (a, b) => a + b;
const sub: BinaryFn = (a, b) => a - b;
const mul: BinaryFn = (a, b) => a * b;
const div: BinaryFn = (a, b) => a / b;
```

매개변수나 반환 값에 타입을 명시하기보다는 함수 표현식 전체에 타입 구문을 적용하는 것이 더 좋다.

```jsx
// 매개변수에 타입 명시
async function checkedFetch(input: RequestInfo, init?: RequestInit) {
  const response = await fetch(input, init);
  if (!response.ok) {
    // Converted to a rejected Promise in an async function
    throw new Error("Request failed: " + response.status);
  }
  return response;
}

// ✨ 함수 표현식 전체에 타입 구문 적용
declare function fetch(
  input: RequestInfo,
  init?: RequestInit
): Promise<Response>;

const checkedFetch: typeof fetch = async (input, init) => {
  const response = await fetch(input, init);
  if (!response.ok) {
    throw new Error("Request failed: " + response.status);
  }
  return response;
};
```

---

## 아이템 13 타입과 인터페이스의 차이점 알기

인덱스 시그니처는 인터페이스와 타입에서 모두 사용할 수 있다. 또한 함수 타입도 인터페이스나 타입으로 정의할 수 있다.

```jsx
// 타입
type TState = {
  name: string,
  capital: string,
};

// 인터페이스
interface IState {
  name: string;
  capital: string;
}

// 인덱스 시그니처
type TDict = { [key: string]: string };
interface IDict {
  [key: string]: string;
}

// 함수 타입
type TFn = (x: number) => string;
interface IFn {
  (x: number): string;
}

const toStrT: TFn = (x) => "" + x; // OK
const toStrI: IFn = (x) => "" + x; // OK
```

인터페이스는 타입을 확장할 수 있으며 타입은 인터페이스를 확장할 수 있다.

- 클래스를 구현할 때는 둘 다 사용할 수 있다.
- 인터페이스는 유니온 타입과 같은 복잡한 타입을 확장하지는 못한다. ( 유니온 타입 O, 유니온 인터페이스 X)
- 인터페이스는 **보강(augment)**이 가능하다.

```jsx
interface IState {
  name: string;
  capital: string;
}
interface IState {
  population: number;
}
const wyoming: IState = {
  name: "Wyoming",
  capital: "Cheyenne",
  population: 500_000,
}; // OK
```

- 어떤 API에 대한 타입 선언을 작성해야한다면 API가 변경될 때 사용자가 인터페이스를 통해 새로운 필드를 병합할 수 있어 인터페이스를 사용하는게 좋다.

---

## 아이템 14 타입 연산과 제너릭 사용으로 반복 줄이기

같은 코드를 반복하지 말라(DRY, don’t repeat yourself)

- 중복되는 타입들은 타입에 이름을 붙여 반복을 줄인다.
- extends를 사용해 인터페이스 필드의 반복을 피한다.
- 제너릭 타입 (ex. Pick, Partial, Omit, ReturnType ) 사용한다.

---

## 아이템 15 동적 데이터에 인덱스 시그니처 사용하기

타입스크립트에서는 타입에 인덱스 시그니처를 명시하여 유연하게 매핑을 표현할 수 있다.

```jsx
// 인덱스 시그니처 [property: stirng]: string
type Rocket = { [property: string]: string };

const rocket: Rocket = {
  name: "Falcon 9",
  variant: "v1.0",
  thrust: "4,940 kN",
}; // OK
```

하지만 네 가지 단점이 있다.

- 잘못된 키를 포함해 모든 키를 허용(name 대신에 Name으로 작성해도 된다.)
- 특정 키가 필요하지 않는다. {}도 유효한 object이다.
- 모든 키는 같은 속성을 가져야한다.
- 모든 키를 허용하기에 자동 완성 기능이 동작하지 않는다.

---

## 아이템 16 number 인덱스 시그니처 보다는 Array, 튜플, ArrayLike를 사용하기

자바스크립트의 객체는 키와 값의 모음. 키는 값은 어떤 것이든 될 수 있다.

```tsx
const x = {};
x[[1, 2, 3]] = 2; // 키를 객체로 사용해도 가능
console.log(x); // {'1,2,3': 2};
```

자바스크립트는 ‘해시 가능’ 객체라는 표현이 없기 때문에 문자열이 아닌 더 복잡한 객체를 키로 사용하려 한다면, 내부적으로 toString 메서드가 호출되어 객체를 문자열로 반환하여 키로 사용해야한다.

- 해시 : 키를 테이블에 저장할 때 키 값을 Hash Method를 이용해 계산한 후, 데이터의 고유한 숫자값인 결과값을 배열의 인덱스로 사용하여 저장하는 방식

숫자를 키로 사용할 수 없어서 속성 이름으로 숫자로 사용한다면, 런타임시 문자열로 변환된다.

숫자로 인덱싱하는 배열 또한 객체로서 배열의 모든 숫자 인덱스들은 문자열로 변환되어 사용된다.

```tsx
const x = {
  1: 2,
  3: 4,
};
console.log(x); // {'1': 2, '3': 4};

console.log(typeof []); // 배열의 타입은 객체 ('object' )
const x = [1, 2, 3];
// 배열에 숫자 인덱스로 배열 요소에 접근 가능
console.log(x[0]); // 1
// 인덱스가 문자열로 변환되어 문자열 키로도 배열 요소에 접근 가능
console.log(x["1"]); // 2
// 키가 문자열로 출력
console.log(Object.keys(x)); // ['0', '1', '2']
// 키의 타입이 문자열
console.log(typeof Object.keys(x)[0]); // 'string'
```

타입스크립트는 숫자 키를 허용하며 문자열 키와는 다른 것으로 인식한다.

```tsx
// lib.es5.d.ts

interface Array<T> {
  ...
  [n: number]: T; // 숫자 키
}`

const xs = [1, 2, 3];
const x0 = xs[0]; // OK
const x1 = xs['1'];
// ~~~~ Element implicitly has an 'any' type
//      because index expression is not of type 'number' ( 숫자 키와 문자열 키가 다름 )
```

- for … of : 값의 타입 number 중요, 인덱스 타입 고려X
- forEach : 인덱스 타입(number)이 중요
- for 문 : 순회 도중 멈춰야하면

---

## 아이템 17 변경 관련된 오류 방지를 위해 readonly 사용하기

**readonly**

- 배열의 요소를 읽을 수 있지만, 쓸 수는 없다.
- length를 읽을 수 있지만 배열을 바꿀 수 없다.
- 따라서 배열을 변경하는 pop을 비롯한 메서들을 호출할 수 없다.
- 변경 가능한 배열을 readonly에 할당할 수 있지만 그 반대는 불가능하다.

---

## 아이템 18 매핑된 타입을 사용하여 값을 동기화하기

불필요한 리렌더링을 막기 위한 최적화 방법

1. 보수적 접근법 또는 실패에 닫힌 접근법 : 정확하지만 너무 자주 업데이트될 가능성이 있다.

```jsx
interface ScatterProps {
  // The data
  xs: number[];
  ys: number[];

  // Display
  xRange: [number, number];
  yRange: [number, number];
  color: string;

  // Events
  onClick: (x: number, y: number, index: number) => void;
}

function shouldUpdate(
  oldProps: ScatterProps,
  newProps: ScatterProps
) {
  let k: keyof ScatterProps;
  for (k in oldProps) {
    if (oldProps[k] !== newProps[k]) {
      if (k !== 'onClick') return true;
    }
  }
  return false;
}
```

1. 실패에 열린 접근법 : 불필요하게 다시 그리는 단점은 해결했지만 실제로 그려져야할 때 누락될 가능성이 있다.

```jsx
function shouldUpdate(
  oldProps: ScatterProps,
  newProps: ScatterProps
) {
  return (
    oldProps.xs !== newProps.xs ||
    oldProps.ys !== newProps.ys ||
    oldProps.xRange !== newProps.xRange ||
    oldProps.yRange !== newProps.yRange ||
    oldProps.color !== newProps.color
    // (no check for onClick)
  );
```

1. 매핑된 타입과 객체 사용하기 : 매핑된 타입은 한 객체가 또 다른 객체와 정확히 같은 속성을 가지게 할때 이상적이다.

```jsx
const REQUIRES_UPDATE: {[k in keyof ScatterProps]: boolean} = { // 타입 체커 동작
  xs: true,
  ys: true,
  xRange: true,
  yRange: true,
  color: true,
  onClick: false,
};

function shouldUpdate(
  oldProps: ScatterProps,
  newProps: ScatterProps
) {
  let k: keyof ScatterProps;
  for (k in oldProps) {
    if (oldProps[k] !== newProps[k] && REQUIRES_UPDATE[k]) {
      return true;
    }
  }
  return false;
}

```
