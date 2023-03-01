## 타입스크립트 알아보기

라이브러리 의존성 관리를 위해서는 타입스크립트에서 의존성이 어떻게 동작하는지 이해해야한다.

### 아이템 1 타입스크립트와 자바스크립트의 관계 이해하기

타입스크립트는 문법적으로 자바스크립트의 상위 집합이다.

그러나 문법의 유효성과 동작의 이슈는 독립적인 문제이다. → 문법의 오류가 없어도 타입 체커에게 지적당할 수 있다.

모든 자바스크립트는 타입스크립트이지만, 모든 타입스크립트는 자바스크립트가 아니다.

타입스크립트는 코드의 동작과 의도가 다른 부분을 찾아낸다.

```jsx
let city = "new york city";
console.log(city.toUppercase());
// ~~~~~~~~~~~ Property 'toUppercase' does not exist on type
//             'string'. Did you mean 'toUpperCase'?

// HIDE
export const foo = true;
const states = [
  { name: "Alabama", capital: "Montgomery" },
  { name: "Alaska", capital: "Juneau" },
  { name: "Arizona", capital: "Phoenix" },
  // ...
];
// END

for (const state of states) {
  console.log(state.capitol);
  // ~~~~~~~ Property 'capitol' does not exist on type
  //         '{ name: string; capital: string; }'.
  //         Did you mean 'capital'?
}
```

타입스크립트는 자바스크립트의 런타임 동작을 **모델링**한다.

타입스크립트는 모델링 뿐만 아니라 추가적인 타입 체크를 한다. 하지만 타입 체크를 해도 런타임 오류가 발생하는 경우가 있다.

```jsx
// 런타임 오류가 될 코드인데, 타입체커가 정상적으로 작동한다.
const x = 2 + "3"; // OK, type is string
const y = "2" + 3; // OK, type is string

// 런타임 오류가 발생하지 않는 코드인데 타입 체커가 문제접을 표시한다.
const a = null + 7; // Evaluates to 7 in JS
// ~~~~ Operator '+' cannot be applied to types ...
const b = [] + 12; // Evaluates to '12' in JS
// ~~~~~~~ Operator '+' cannot be applied to types ...
alert("Hello", "TypeScript"); // alerts "Hello"
// ~~~~~~~~~~~~ Expected 0-1 arguments, but got 2
```

타입 시스템이 정적 타입의 정확성을 보장하지는 않는다. ( 이를 원하면 다른 언어를 사용해야한다. )

→ 그러면 ts를 왜 쓰는걸까? 코드의 타입 오류를 체크하고( p.12) 의도와 다르게 동작하는 코드도 찾으려고 한다! (p.62)

---

### 아이템 2 타입스크립트 설정 이해하기

tsconfig.json에서 `noImplicitAny : true` , `strickNullChecks : true` 설정 해야한다.

→ null과 undefined를 잡아내는건 빠르면 빠를수록 좋다!

이 모든 체크를 설정하고 싶다면 `strict : true`로 대부분의 오류를 잡을 수 있다.

---

### 아이템 3 코드 생성과 타입이 관계없음을 이해하기

타입스크립트는 두가지 역할을 한다.

1. 최신 타입스크립트/자바스크립트를 브라우저에서 동작할 수 있도록 구버전의 자바스크립트로 트랜스파일을 한다.
2. 코드의 타입 오류를 체크한다

→ 이 두가지는 완벽하게 독립적이기 때문에 타입스크립트는 다음과 같은 특성을 가진다.

1️⃣  타입 오류가 있는 코드도 컴파일 가능하다.

오류가 있을 때 컴파일 하지 않으려면 tsconfig.json에 `noEmitOnError : true` 로 설정하거나 빌드 도구에 적용하면 된다.

2️⃣ 자바스크립트로 컴파일 되는 과정에서 모든 인터페이스, 타입, 타입 구문은 제거되기 때문에 런타임에서는 타입체크가 불가능하다.

```jsx
interface Square {
  width: number;
}
interface Rectangle extends Square {
  height: number;
}
type Shape = Square | Rectangle;

function calculateArea(shape: Shape) {
  if (shape instanceof Rectangle) {
    // ~~~~~~~~~ 'Rectangle' only refers to a type,
    //           but is being used as a value here
    return shape.width * shape.height;
    //         ~~~~~~ Property 'height' does not exist
    //                on type 'Shape'
  } else {
    return shape.width * shape.width;
  }
}
```

→ instanceof 체크는 런타임에 일어나지만, Rectangle은 타입이기 때문에 런타임에 아무런 역할을 할 수 없다.

```jsx
function calculateArea(shape: Shape) {
  if ("height" in shape) {
    shape; // Type is Rectangle
    return shape.width * shape.height;
  } else {
    shape; // Type is Square
    return shape.width * shape.width;
  }
}
```

→ in을 사용하여 런타임에서도 타입을 체크할 수 있다.

```jsx
function calculateArea(shape: Shape) {
  if (shape.kind === "rectangle") {
    shape; // Type is Rectangle
    return shape.width * shape.height;
  } else {
    shape; // Type is Square
    return shape.width * shape.width;
  }
}
```

→ ‘태그’ 기법으로 런타임에서 접근 가능한 타입 정보를 명시해줄 수 도 있다.

타입인지 값인지 확인하는 방법은 js로 변환할때 사라지는지 → 타입은 사라지고 값은 남는다 (p. 48)

- 타입 : 런타임 접근 불가, type, interface, class, enum
- 값 : 런타임 접근 가능, class, enum, const, let

3️⃣ 타입 연산은 런타임에 영향을 주지 않는다.

```jsx
function asNumber(val: number | string): number {
  return val as number;
}

// 변환된 자바스크립트 코드
function asNumber(val) {
  return val;
}

// 값을 정제하려면 자바스크립트 연산을 수행해야한다 (typeof)
function asNumber(val: number | string): number {
  return typeof(val) === 'string' ? Number(val) : val;
}
```

4️⃣ 런타임의 타입은 선언된 타입과 다를 수 있다.

```jsx
function turnLightOn() {}
function turnLightOff() {}
function setLightSwitch(value: boolean) {
  switch (value) {
    case true:
      turnLightOn();
      break;
    case false:
      turnLightOff();
      break;
    default:
      console.log(`I'm afraid I can't do that.`);
  }
}
```

→ 런타임에서 value : boolean 은 제거되기 때문에 value 값으로 “ON”과 같은 string이 들어올 경우 default가 실행된다.

5️⃣ 타입스크립트 타입으로는 함수를 오버로드 할 수 없다.

함수 오버로딩 : 동일한 이름에 매개변수만 다른 여러 버전의 함수를 허용하는 것

타입스크립트에서 타입은 런타임시 사라지기 때문에 함수 오버로딩이 불가하다.

오직 하나의 구현체만이 존재한다.

6️⃣ 타입스크립트 타입은 런타임 성능에 영향을 주지 않는다.

---

### 아이템 4 **구조적 타이핑에 익숙해지기**

자바스크립트는 본질적으로 덕 타이핑(duck typing)기반이다.

덕 타이핑 : 매개변수 값이 요구사항을 만족하면 타입이 무엇인지 신경쓰지 않고 동작을 그대로 모델링한다. 예를 들어 어떤 새가 ‘오리처럼’ 걷고, 헤엄치고, 꽥꽥 울음소리를 내면 오리라고 할 수 있다.

```jsx
interface Vector2D {
  x: number
  y: number
}

interface NamedVector {
  name: string
  x: number
  y: number
}

function calculateLength(v: Vector2D) {
  return Math.sqrt(v.x * v.x + v.y * v.y)
}

const v: NamedVector = { x: 3, y: 4, name: 'Kim' }
calculateLength(v) // NamedVector는 Vector2D의 x,y 속성이 있기 때문에 호출 가능하다.
```

→ calculateLength(v) 에서 NamedVector는 Vector2D의 x,y 속성이 있기 때문에 호출 가능하다.

```jsx
interface NamedVector {
  name: string;
  x: number;
  y: number;
}
interface Vector3D {
  x: number;
  y: number;
  z: number;
}
function calculateLengthL1(v: Vector3D) {
  let length = 0;
  for (const axis of Object.keys(v)) {
    const coord = v[axis];
               // ~~~~~~~ Element implicitly has an 'any' type because ...
               //         'string' can't be used to index type 'Vector3D'
    length += Math.abs(coord);
  }
  return length;
}

}
const vec3D = {x: 3, y: 4, z: 1, address: '123 Broadway'};
calculateLengthL1(vec3D);  // OK, returns NaN
```

→ calculateLengthL1(vec3D) 에서 vec3D는 Vector3D의 x,y,z의 속성을 가지고 있어 호출이 가능하지만,

키 address의 값에는 '123 Broadway' 이라는 string이 들어가기 때문에 coord는 any값을 가지게 되어 오류가 난다.

---

### 아이템 5 any 타입 지양하기

1️⃣ any 타입에는 타입 안정성이 없다.

2️⃣ any는 함수 시그니처( (key : string) : string)를 무시해 버린다.

3️⃣ 타입스크립트 언어 서비스는 자동완성 기능과 적절한 도움말을 제공한다. 하지만 any 타입인 심벌을 사용하면 아무런 도움을 받지 못한다.

```jsx
interface Person {
  first: string; // rename symbole로 first -> firstName 으로 바꾸게 되면
  last: string;
}

const formatName = (p: Person) => `${p.first} ${p.last}`; // first -> firstName으로 자동으로 바뀐다.
const formatNameAny = (p: any) => `${p.first} ${p.last}`; // first -> firstName으로 자동으로 바뀌지 않는다.
```

4️⃣ any 타입은 코드 리팩터링 때 버그를 감춘다.

5️⃣ any는 타입 설계를 감춰버리기 때문에 any를 사용하지 않고 설계가 명확히 보이도록 타입을 일일이 작성하는 것이 좋다.

6️⃣ 타입 체커가 사람의 실수를 잡아주고 코드의 신뢰도를 높여주는데 런타임에 타임 오류를 발견하게 된다면 타입 체커를 신뢰할 수 없게된다. 따라서 any 타입을 쓰지 않으면 런타임에 발견될 오류를 미리 잡을 수 있고 신뢰도를 높일 수 있다.
