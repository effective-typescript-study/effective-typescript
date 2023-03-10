## 타입 선언과 @types

라이브러리 의존성 관리를 위해서는 타입스크립트에서 의존성이 어떻게 동작하는지 이해해야한다.

### 아이템 45 devDepndencies 에 typescript와 @types 추가하기

npm은 각 라이브러리를 의존성에 따라 package.json 파일 안에서 3가지로 구분해서 관리한다.

**dependencies** : 현재 프로젝트를 실행하는데 필수적인 라이브러리

**devDepndencies** : 현재 프로젝트를 개발하고 테스트하는데 사용되지만, 런타임에서는 필요없는 라이브러리

**peerDepndencies** : 런타임에 필요하긴 하지만, 의존성을 직접 관리하지 않는 라이브러리

→ 타입스크립트는 `devDepndencies` 에 속한다.

---

### 아이템 46 타입 선언과 관련된 세 가지 버전 이해하기

타입스크립트를 사용할 때에는 1️⃣ 라이브러리의 버전 2️⃣ 타입선언(@types) 의 버전 3️⃣ 타입스크립트의 버전 을 고려해야한다.

다음 세 가지 버전 중 하나라도 맞지 않으면 엉뚱한 곳에서 오류가 발생할 수 있다.

따라서 특정 라이브러리는 **dependencies** 로 설치하고, 타입정보는 **devDepndencies**로 설치한다.

```jsx
// 라이브러리
npm install react
// 타입 정보
npm install --save-dev @types/react
```

실제 라이브러리와 타입 정보의 버전을 별도로 관리하면 네 가지의 문제점이 있다.

1. 라이브러리를 업데이트 했지만 실수로 타입 선언은 업데이트 하지 않는 경우
2. 라이브러리보다 타입 선언의 버전이 최신인 경우
3. 프로젝트에서 사용하는 타입스크립트 버전보다 라이브러리에서 필요로 하는 타입스크립트 버전이 최신인 경우
4. @types 의존성이 중복되는 경우

다음과 같은 문제를 해결하기 위해 번들링(타입 선언을 라이브러리에 포함하는) 경우도 있지만 부수적인 네 가지 문제가 발생한다.

1. 번들된 타입에 보강으로 해결할 수 없는 문제가 발생하는 경우
2. 프로젝트 내의 타입선언이 다른 라이브러리의 타입 선언에 의존하는 경우
3. 프로젝트의 과거 버전에 있는 타입선언에 문제가 있는 경우
4. 타입 선언의 패치 업데이트가 어려움

---

### 아이템 47 공개 API에 등장하는 모든 타입을 익스포트하기

라이브러리 제작자는 공개(export) API에 사용되는 타입들도 export 해야한다.

### 아이템 48 API 주석에 TSDoc 사용하기

```jsx
// Generate a greeting. Result is formatted for display.
function greet(name: string, title: string) {
  return `Hello ${title} ${name}`;
}

/** Generate a greeting. Result is formatted for display. */
function greetJSDoc(name: string, title: string) {
  return `Hello ${title} ${name}`;
}
```

→ 사용자를 위한 문서라면 `//` 대신 `/** */` 로 JSDoc 스타일의 주석을 사용하는 것이 좋다. 편집기로 주석을 확인할 수 있다.

```jsx
/**
 * Generate a greeting.
 * @param name Name of the person to greet
 * @param salutation The person's title
 * @returns A greeting formatted for human consumption.
 */
function greetFullTSDoc(name: string, title: string) {
  return `Hello ${title} ${name}`;
}
```

→ @param, @returns 매개변수와 반환값에 대한 설명

```jsx
interface Vector3D {}
/** A measurement performed at a time and place. */
interface Measurement {
  /** Where was the measurement made? */
  position: Vector3D;
  /** When was the measurement made? In seconds since epoch. */
  time: number;
  /** Observed momentum */
  momentum: Vector3D;
}
```

-> 객체의 필드에 대한 설명

`lib.es5.d.ts`를 보면 TSDoc 으로 설명되어있다.

[TypeScript/lib.es5.d.ts at main · microsoft/TypeScript](https://github.com/microsoft/TypeScript/blob/main/lib/lib.es5.d.ts)

---

### 아이템 49 콜백에서 this에 대한 타입 제공하기

let, const : 렉시컬 스코프

this : 다이나믹 스코프 , 값은 ‘정의된’ 방식이 아니라 ‘호출된’ 방식에 따라 달라진다.

this는 전형적으로 객체의 현재 인스턴스를 참조하는 클래스에서 가장 많이 쓰인다. this에 call()을 사용해 명시적으로 this를 바인딩하여 this 바인딩을 온전히 제어할 수 있다.

타입스크립트에서도 this 바인딩에 대한 타입을 지정해줘야한다.

---

### 아이템 50 오버로딩 타입보다는 조건부 타입을 사용하기

조건부 타입은 추가적인 오버로딩 없이 유니온 타입을 지원한다.

```jsx
function double<T extends number | string>(
  x: T
): T extends string ? string : number;

function double(x: any) { return x + x; }

const num = double(12);  // number
const str = double('x');  // string

// function f(x: string | number): string | number
function f(x: number|string) {
  return double(x);
}
```

---

### 아이템 51 의존성 분리를 위해 미러 타입 사용하기

작성중인 라이브러리가 의존하는 라이브러리의 구현과 무관하게 타입에만 의존한다면, 필요한 선언부만 추출하여 작성중인 라이브러리에 넣는 미러링을 사용하는 것이 좋다.

---

### 아이템 52 테스팅 타입의 함정에 주의하기

타입 체커와 독립적으로 작용하는 dtslint 또는 타입 시스템 외부에서 타입을 검사하는 유사한 도구를 사용하여 테스트 코드를 작성하면 된다.

```jsx
declare function map<U, V>(
  array: U[],
  fn: (this: U[], u: U, i: number, array: U[]) => V
): V[];

const beatles = ["john", "paul", "george", "ringo"];
map(
  beatles,
  function (
    name, // $ExpectType string
    i, // $ExpectType number
    array // $ExpectType string[]
  ) {
    this; // $ExpectType string[]
    return name.length;
  }
); // $ExpectType number[]
```

반환 타입을 체크하는 것이 좋은 테스트 코드이다.
