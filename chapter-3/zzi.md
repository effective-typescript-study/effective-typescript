### 아이템 19 추론 가능한 타입을 사용해 장황한 코드 방지하기

---

타입스크립트를 처음 접한 개발자들은 보통 자바스크립트에서 코드를 포팅할 때 가장 먼저 하는 일은 타입 구문을 넣는 것이다. 하지만 타입이 추론 되기 때문에 명시적인 타입 구문은 필요하지 않다. 또한 타입이 추론되면 리팩터링이 용이해지고 코드를 간결하게 작성할 수 있다. 

타입이 추론되지만 타입을 명시해야 하는 몇 가지의 상황들이 있다.

1. 객체 리터럴 정의 ⇒ 객체 리터럴 정의를 하면 잉여 속성 체크가 동작하여 타입의 오타와 같은 오류를 잡기 효율적이다. 
2. 함수의 반환 ⇒ 타입 추론이 가능해도 구현상의 오류가 함수를 호출한 곳까지 영향을 미치지 않도록 타입 구문을 명시하는게 좋다. 또한 반환 타입을 명시하면 함수를 더욱 명확하게 파악할 수 있다. 

### 아이템 20 다른 타입에는 다른 변수 사용하기

---

자바스크립트에서는 한 변수를 다른 목적을 가지는 다른 타입으로 재사용해도 된다.

```tsx
let id = '12-34-56'
fetchProduct(id);
id = 123456
fetchProductSerialNumber(id);
```

하지만 타입스크립트에서는 오류가 발생한다. 유니온 타입을 쓰면 해결이 가능하지만 더 큰 문제가 발생할 수 있고, 코드가 간결해지고 타입체커가 추론하기에도 좋기 때문에 차라리 별도의 변수를 도입하는 것이 낫다.

```tsx
const id = '12-34-56'
fetchProduct(id);
const serial = 123456
fetchProductSerialNumber(serial);
```

### 아이템 21 타입 넓히기

---

타입 추론을 할 때 할당 가능한 값들의 집합을 유추하는데 이 과정을 ‘넓히기’라고 부른다. 

```tsx
interface Vector3 {
	x: number;
	y: number;
	z: number;
}

function getComponent(vector: Vector3, axis: 'x' | 'y' | 'z') {
	return vector[axis];
}

let x = 'x';
let vec = {x: 10, y: 20, z: 30}
getComponent(vec, x); // 오류 발생 
```

`getComponent` 함수는 두 번째 매개변수에 `‘x’`| `‘y’` | `‘z’` 타입을 기대했지만, `‘x’`의 타입은 할당 시점에 넓히기가 동작해서 `string`으로 추론되었다. 

넓히기를 제어할 수 있는 몇 가지 방법들이 존재한다.

1. `const` 사용하기 ⇒ const로 변수를 선언하면 재할당될 수 없으므로 더 좁은 타입으로 추론할 수 있다.
2. 명시적 타입구문 사용하기 
3. 타입체커에 추가적인 문맥 제공하기
4. `as const` 단언문 사용하기 

### 아이템 22 타입 좁히기

---

타입 좁히기는 타입스크립트가 넓은 타입으로부터 좁은 타입으로 진행하는 과정을 말한다.

아래의 예시처럼 속성 체크로 타입을 좁힐수 있다. 

```tsx
function contains(text: string, search: string|RegExp) {
	if (search instanceof RegExp) {
		// 타입이 RegExp
	} else {
		// 타입이 string
	}
```

또 다른 타입 좁히기의 방식은 명시적 ‘태그’를 붙이는 것이다. 

### 아이템 23 한꺼번에 객체 생성하기

---

객체를 생성할 때는 속성을 하나씩 추가하기보다는 여러 속성을 포함해서 한꺼번에 생성해야 타입 추론에 유리하다.

### 아이템 24 일관성 있는 별칭 사용하기

---

- 별칭을 사용하면 제어 흐름을 분석하기 어렵기 때문에 신중하게 사용해야 한다.
- 비구조화 문법을 사용해 일관된 이름을 사용하는 것이 좋다.

### 아이템 25 비동기 코드에는 콜백 대신 async 함수 사용하기

---

콜백보다는 Promise나 async/await를 사용해야 하는 이유는 다음과 같다.

- 콜백보다는 Promise가 코드를 작성하기 쉽다.
- 콜백보다는 Promise가 타입을 추론하기 쉽다.

### 아이템 26 타입 추론에 문맥이 어떻게 사용되는지 이해하기

---

타입스크립트는 타입을 추론할 때 단순히 값만 고려하지는 않고, 값이 존재하는 곳의 문맥까지도 살핀다. 

`language`의 값을 `string`으로 추론하여, `Language` 타입으로 할당이 불가능하여 오류가 발생했다. 이때 타입 선언을 하는 방법과 `language`를 상수로 만드는 방법이 있다. 

```tsx
type Language = 'JavaScript' | 'TypeScript' | 'Python';
function setLanguage(language: Language) { /* */ }

setLanguage('JavaScript'); // 정상

// Error
let language = 'JavaScript'; 
setLanguage(language); // 오류

// 1번 방법 => 타입선언
let language: Language = 'JavaScript'; 
setLanguage(language); // 정상

// 2번 방법 => 상수로 만들기
const language = 'JavaScript'; 
setLanguage(language); // 정상
```

튜플이나 객체의 경우 `as const`를 사용한다. 그러나 `as const`를 사용하면 정의한 곳이 아니라 사용한 곳에서 오류가 발생하므로 주의해야한다.

```tsx
type Language = "JavaScript" | "TypeSrcipt" | "Python";
interface ConvernedLanguage {
  language: Language;
  organization: string;
}

const ts = {
  language: "TypeSrcipt" as const,
  organization: "Microsoft",
};

complain(ts);

function complain(language: ConvernedLanguage) { /* */ }
```

### 아이템 27 함수형 기법과 라이브러리로 타입 흐름 유지하기

---

타입 흐름을 개선하고, 가독성을 높이고, 명시적인 타입 구문의 필요성을 줄이기 위해 직접 구현하기보다는 내장된 함수형 기법과 로대시 같은 유틸리티 라이브러리를 사용하는 것이 좋다.
