## 코드를 작성하고 실행하기

### 아이템 53 타입스크립트 기능보다는 ECMAScript 기능을 사용하기

타입스크립트는 타입 기능만 발전시킨다.

1. 열거형(enum) 대신 리터럴 타입의 유니온 사용하기
2. 매개변수 속성 대신 일반 속성 사용하기

```jsx
class Person {
  first: string; // 일반 속성
  last: string; // 일반 속성
  constructor(public name: string) { // 매개변수 속성
    [this.first, this.last] = name.split(' ');
  }
}
```

1. ES2015의 import/export 모듈 사용하기
2. 데코레이터(ex. logged 애너테이션) 사용하지 않기

---

### 아이템 54 객체를 순회하는 노하우

```jsx
interface ABC {
  a: string;
  b: string;
  c: number;
}

function foo(abc: ABC) {
  for (const k in abc) {  // const k: string
    const v = abc[k];
           // ~~~~~~ Element implicitly has an 'any' type
           //        because type 'ABC' has no index signature
  }
}
const x = {a: 'a', b: 'b', c: 2, d: new Date()}; // d 는 ABC에 없어서 무시되었다.
foo(x);  // OK

function foo(abc: ABC) {
  **let k: keyof ABC;**
  for (k in abc) {  // let k: "a" | "b" | "c"
    const v = abc[k];  // Type is string | number
  }
}
```

처음에는 k의 타입은 string으로 추론되었지만 `keyof`를 사용해 키 타입을 한정지을 수 있어 k의 타입은 "a" | "b" | "c" 가 되었다.

하지만 ABC에 존재하지 않는 이외의 속성들(예를 들어 d) 이 할당될 수도 있고

→ 객체의 키와 값을 순회하기 위해서는 `object.entries` 사용

```jsx
interface ABC {
  a: string;
  b: string;
  c: number;
}

function foo(abc: ABC) {
  for (const [k, v] of Object.entries(abc)) {
    k; // Type is string
    v; // Type is any
  }
}
```

---

### 아이템 55 DOM 계층 구조 이해하기

타입스크립트는 DOM 계층 구조 파악하기 용이하다.

![image](https://user-images.githubusercontent.com/62867581/224232864-98f08095-36bf-4a99-bcf4-6aaa999ccf80.png)

1. EventTarget : DOM 타입 중 가장 추상화된 타입
2. Node 타입
3. Element와 HTMLElement
4. HTMLxxxElement : 특정 엘리먼트는 고유한 속성을 가지고 있다. ex.ImageElement → src 속성
5. Event - UIEvent, MouseEvent …

4,5 번은 특정 엘리먼트가 가진 속성에 접근하려면 타입 단언(as)으로 분기해야하고 if문으로 null일 가능성을 체크해야한다.

---

### 아이템 56 정보를 감추는 목적으로 private 사용하지 않기

정보를 감추기 위해서는 private을 사용하지 말고 클로저(closure)를 사용해야한다.

---

### 아이템 57 소스맵을 사용하여 타입스크립트 디버깅하기

```jsx
// tsconfig.json 파일
{
	"compilerOptions" : {
		"sourceMap" : true
	}
}
```

변환된 자바스크립트를 디버깅하는 것이 아닌, 소스맵을 사용해 런타임에 타입스크립트 코드르 디버깅해야한다.
