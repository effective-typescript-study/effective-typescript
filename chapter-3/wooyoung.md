# 타입 추론

타입스크립트는 타입 추론을 적극적으로 수행한다.

## 아이템 19 추론 가능한 타입을 사용해 장황한 코드 방지하기

타입 추론이 된다면 명시적 타입 구문은 필요하지 않다.

```jsx
let x = 12; // number로 추론됨
```

비구조화 할당을 통해 타입이 추론된다면 명시적 타입 선언은 불필요하다.

```jsx
interface Product {
  id: string;
  name: string;
  price: number;
}

function logProduct(product: Product) {
  // 비구조화 할당
  const { id, name, price } = product;
  console.log(id, name, price);
}
```

객체 리터럴 정의할 때 타입을 명시하면 잉여 속성 체크로 속성 오타 등 언어서비스를 사용할 수 있고, **변수가 할당되는 시점**에 오류를 표시해준다.

```jsx
// 👎 변수가 사용되는 순간에 오류 표시
interface Product {
  id: string;
  name: string;
  price: number;
}

function logProduct(product: Product) {
  const id: number = product.id;
  // ~~ Type 'string' is not assignable to type 'number'
  const name: string = product.name;
  const price: number = product.price;
  console.log(id, name, price);
}

// 👍 변수가 할당되는 순간에 오류 표시
const furby = {
  name: "Furby",
  id: 630509430963,
  price: 35,
};
logProduct(furby);
// ~~~~~ Argument .. is not assignable to parameter of type 'Product'
//         Types of property 'id' are incompatible
//         Type 'number' is not assignable to type 'string'
```

함수의 반환에도 타입을 명시하여 오류를 방지하면 좋다.

---

## 아이템 20 다른 타입에는 다른 변수 사용하기

“변수의 값은 바뀔 수 있지만 보통 그 타입은 바뀌지 않기”때문에 타입이 다르면 별도의 변수를 생성한다.

const로 변수를 선언하면 코드가 간결해지고, 타입체커가 타입을 추론하기에도 좋다.

```jsx
// 👎
let id: string | number = "12-34-56";
fetchProduct(id);

id = 123456; // OK
fetchProductBySerialNumber(id); // OK

// 👍
const id = "12-34-56";
fetchProduct(id);

const serial = 123456; // OK
fetchProductBySerialNumber(serial); // OK
```

---

## 아이템 21 타입 넓히기

런타임에는 모든 변수는 유일한 값을 가진다. 그러나 타입스크립트가 작성된 코드를 체크하는 정적 분석 시점에, 변수는 가능한 값들의 집합인 타입을 가진다.

다시 말하면, 지정된 단일 값을 가지고 할당 가능한 값들의 집합을 유추해야한다. 이 과정을 타입 넓히기라고 부른다.

타입 추론의 강도를 직접 제어하려면 타입스크립트의 기본 동작을 재정의해야한다.

- const로 변수 선언하면 재할당이 불가능해 좁은 타입으로 추론
- 명시적 타입 구문을 제공
- 타입 체커에 추가적이 문맥을 제공
- const 단언문(as const)

---

## 아이템 22 타입 좁히기

대표적으로 null 체크

타입 좁히는 방법

- 분기문
  ```jsx
  const el = document.getElementById("foo"); // Type is HTMLElement | null
  if (!el) throw new Error("Unable to find #foo");
  el; // Now type is HTMLElement
  el.innerHTML = "Party Time".blink();
  ```
- instanceof
  ```jsx
  function contains(text: string, search: string | RegExp) {
    if (search instanceof RegExp) {
      search; // Type is RegExp
      return !!search.exec(text);
    }
    search; // Type is string
    return text.includes(search);
  }
  ```
- 속성체크
  ```jsx
  interface A {
    a: number;
  }
  interface B {
    b: number;
  }
  function pickAB(ab: A | B) {
    if ("a" in ab) {
      ab; // Type is A
    } else {
      ab; // Type is B
    }
    ab; // Type is A | B
  }
  ```
- 명시적 “태그” 사용 - 태그된 유니온, 구별된 유니온
  ```jsx
  interface UploadEvent {
    type: "upload";
    filename: string;
    contents: string;
  }
  interface DownloadEvent {
    type: "download";
    filename: string;
  }
  type AppEvent = UploadEvent | DownloadEvent;

  function handleEvent(e: AppEvent) {
    switch (e.type) {
      case "download":
        e; // Type is DownloadEvent
        break;
      case "upload":
        e; // Type is UploadEvent
        break;
    }
  }
  ```
- 사용자 저의 타입 가드(is)
  ```jsx
  function isInputElement(el: HTMLElement): el is HTMLInputElement {
    return 'value' in el;
  }

  function getElementContent(el: HTMLElement) {
    if (isInputElement(el)) {
      el; // Type is HTMLInputElement
      return el.value;
    }
    el; // Type is HTMLElement
    return el.textContent;
  }
  ```

❗️typeof null === object

❗️’’ === false, 0 === false

---

## 아이템 23 한꺼번에 객체 생성하기

객체를 생성할 때는 여러 속성을 포함해서 한번에 생성해야한다.

안전한 타입으로 속성을 추가하려면 전개 연산자로 객체 전개({…a, …b})를 사용한다.

---

## 아이템 24 일관성 있는 별칭 사용하기

비구조화 문법을 사용해 일관된 이름을 사용해야한다.

```jsx
interface Coordinate {
  x: number;
  y: number;
}

interface BoundingBox {
  x: [number, number];
  y: [number, number];
}

interface Polygon {
  exterior: Coordinate[];
  holes: Coordinate[][];
  bbox?: BoundingBox;
}

function isPointInPolygon(polygon: Polygon, pt: Coordinate) {
  const { bbox } = polygon; // 비구조화로 bbox 변수명 사용 가능
  if (bbox) {
    const { x, y } = bbox;
    if (pt.x < x[0] || pt.x > x[1] || pt.y < x[0] || pt.y > y[1]) {
      return false;
    }
  }
  // ...
}
```

---

## 아이템 25 비동기 코드에는 콜백 대신 async 함수 사용하기

콜백 함수 보다는 프로미스가 `코드를 작성`하기 쉬우며 `타입을 추론`하기 쉽다.

async 함수는 항상 프로미스를 반환하도록 강제된다. async 함수에서 프로미스로 반환하면 또 다른 프로미스로 래핑되지 않는다.

```jsx
const getNumber = async () => 42; // Type is () => Promise<number>
```

---

## 아이템 26 타입 추론에 문맥이 어떻게 사용되는지 이해하기

타입스크립트는 타입을 추론할때 값이 존재하는 곳의 문맥까지 살핀다.

```jsx
type Language = "JavaScript" | "TypeScript" | "Python";
function setLanguage(language: Language) {
  /* ... */
}

setLanguage("JavaScript"); // OK

let language = "JavaScript"; // language type을 string으로 추론
setLanguage(language);
// ~~~~~~~~ Argument of type 'string' is not assignable
//          to parameter of type 'Language'
```

타입스크립트는 일반적으로 값이 처음 등장할 때 타입을 결정한다.

```jsx
// language 타입 제한
let language: Language = "JavaScript";
setLanguage(language);

// const로 language를 상수로 만듦
const language = "JavaScript";
setLanguage(language); // OK
```

문맥과 값을 분리하기 위해 상수 단언을 사용하면 정의한 곳이 아니라 사용한 곳에서 오류가 발생하므로 주의해야한다.

---

## 아이템 27 함수형 기법과 라이브러리로 타입 흐름 유지하기

같은 코드를 타입스크립트로 작성하면 서드파트 라이브러리를 사용하는 것이 유리하다. 타입 정보를 탐고하며 작업할 수 있기 때문에 서드파티 라이브러리 기반으로 바꾸는데 시간이 훨씬 단축된다.
