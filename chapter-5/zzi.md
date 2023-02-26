
# 5장 any 다루기

### 아이템 38 any 타입은 가능한 한 좁은 범위에서만 사용하기

---

의도치 않은 타입 안정성의 손실을 피하기 위해서 any의 사용 범위를 최소한으로 좁혀야 한다. 
예를 들면, 

```jsx
function processBar(b: Bar) { /* ... */ }
function f() {
	const x = expressionReturningFoo();
	processBar(x); // 'Foo' 형식의 인수는 'Bar' 형식의 매개변수에 할당될 수 없습니다. 
}
```

**방법 1**

```jsx
function f() {
	const x: any = expressionReturningFoo();
	processBar(x); 
}
```

**방법 2**

```jsx
function f() {
	const x = expressionReturningFoo();
	processBar(x as any); 
}
```

방법 2이 방법이 권장된다. 그 이유는 `x as any` 는 다른 코드에 영향을 미치지 않기 때문이다. 

또한 함수의 반환 타입을 추론할 수 있는 경우에도 함수 반환 타입을 명시하는 것이 좋다. 

### 아이템 39 any를 구체적으로 변형해서 사용하기

---

`any`는 자바스크립트에서 표현할 수 있는 모든 값을 아우르는 매우 큰 범위의 타입이다. `any`보다 구체적인 타입이 존재한다면 더 구체적인 타입을 찾아 타입 안정성을 높이도록 하자. 예를 들면,

1. 배열이라면, `any` 보다는 `any[]`, 배열의 배열이면 `any[][]`
2. 객체라면, `{[key: string]: any}` 

### 아이템 40 함수 안으로 타입 단언문 감추기

---

함수의 모든 부분을 안전한 타입으로 구현하는 것이 이상적이지만, 불필요한 예외 상황까지 고려해 가며 타입 정보를 힘들게 구성할 필요는 없다. 함수 내부에는 타입 단언을 사용하고 함수 외부로 드러나는 타입 정의를 정확히 명시하는 것이 낫다. 

### 아이템 41 any의 진화를 이해하기

---

```jsx
function range(start: number, limit: number) {
  const out = []; // any[]
  out.push(start);
  return out; // number[]
}
```

out의 타입은 처음에  `any[]`로 선언이 되었지만 number 타입의 값을 넣는 순간부터 타입은 `number[]`로 진화한다. 

`any` 타입의 진화는 `noImplicitAny`가 설정된 상태에서 변수의 타입이 암시적 `any`인 경우에만 일어난다. 

```jsx
function range(start: number, limit: number) {
  const out:any[] = []; // any[]
  out.push(start);
  return out; // any[]
}
```

명시적으로 `any`를 선언하면 타입이 그대로 유지된다. `any` 타입의 진화는 암시적 `any` 타입에 어떤 값을 할당할 때만 발생한다. 그리고 암시적 `any` 타입의 값을 읽으려고 하면 오류가 발생한다. 

또한 암시적 `any` 타입은 함수 호출을 거쳐도 진화하지 않는다. 다음 코드에서 `forEach` 안의 화살표 함수는 추론에 영향을 미치지 않는다.

```jsx
function makeSquares(start: number, limit: number) {
  const out = []; // Error: 'out' 변수는 형식을 확인할 수 없는 경우 일부 위치에서 암시적으로 'any[]' 형식입니다.
  range(start, limit).forEach((i) => {
    out.push(i * i);
  });
  
  return out; // Error: 'out' 변수에는 암시적으로 'any[]' 형식이 포함됩니다.
}
```

⭐️ 타입을 안전하게 지키기 위해서는 암시적 `any`를 진화시키는 방식보다는 명시적 타입 구문을 사용하는 것이 더 좋은 설계이다. 

### 아이템 42 모르는 타입의 값에는 any 대신 unknown을 사용하기

---

함수의 반환값으로 any 사용시, 사용되는 곳 마다 타입 오류가 발생하게된다. 그대신 unknown을 사용하자.

unknown은 any 대신 사용할 수 있는 안전한 타입이다. 어떠한 값이 있지만 그 타입을 알지 못하는 경우 unknown을 사용하면 된다. unknown 상태로 사용하면 오류가 발생하기 때문에 타입 단언문이나 타입 체크를 사용하여 타입을 좁히도록 강제할 수 있다. 

> {}, object, unknown 차이
> 
> 
> > {} 타입은 null과 undefined를 제외한 모든 값을 포함한다.
> > 
> > 
> > object 타입은 모든 non-primitive 타입으로 이루어진다. 
> > 

```jsx
// {}, object, unknown 타입 차이점 비교
// {} 는 null, undefined를 뺀 모든 값, object는 non-primitive 값

const a: {} = "a"; // 정상
const b: {} = null; // 에러

const c: object = "foo"; // 에러
const d: object = null; // 에러

const un: unknown = null; // 정상
```

### 아이템 43 몽키 패치보다는 안전한 타입을 사용하기

---

> **몽키패치란?**
> 
> 
> > 런타임 중에 property object를 직접적으로 수정하는 일련의 작업들을 말한다. 전역 변수로 적용이 되기 때문에 다른 코드에도 영향을 주고 **부작용**을 발생시킨다.
> > 

```jsx
document.monkey = 'Tamarin';

// 1. 가장 간단한 방법 : any 단언문 사용 => But, 타입 안전성 상실
(document as any).monkey = 'Tamarin';

// 2. 보강 사용 => 전역적으로 적용되기때문에 코드나 라이브러리로부터 분리할 수 없음
interface Document {
	monkey: string;
}
document.monkey = 'Tamarin';

// 3. 더 구체적인 타입 단언문 사용 => import하는 영역에만 해당됨
interface MoneyDocument extends Document {
	monkey: string;
}
document.monkey = 'Tamarin';
```

### 아이템 44 타입 커버리지를 추적하여 타입 안전성 유지하기

---

noImplicitAny를 설정한다해도 any 타입과 관련된 문제들로부터 안전하다고 할 수 없다. any 타입이 여전히 프로그램 내에 존재할 수 있는 두 가지 경우가 있다.

1. 명시적 any 타입
2. 서드파티 타입 선언

any 타입은 타입 안전성과 생산성에 부정적인 영향을 미칠수 있으므로 프로젝트에서 any 개수를 추적하는 것이 좋다. npm의 type-coverage 패키지를 활용하여 any를 추적할 수 있다.!

`npx type-coverage` ⇒ any가 아니거나 any의 별칭이 아닌 타입을 가지고 있는 비율

`npx type-coverage-detail` ⇒ any의 근원지를 출력해줌

<aside>
❓ 내가 진행했던 보아즈 웹 사이트 2849 / 2919 97.60%가 나왔다. 아무래도 코드의 양이 적고, 복잡도가 적어서 좀 높게 나온듯하다. 타입스크립트 사용할 때는 라이브러리의 사용을 좀 더 신중하게 해야겠다는 생각이 들었다.

</aside>

### References

---

- [https://sunup1992.tistory.com/66?category=907510](https://sunup1992.tistory.com/66?category=907510)

---

아이템 38 p.205 타입이 추론되는 경우에는 적지 않는게 좋다고 했는데 추론이 되는 경우에도 함수 반환타입을 명시하는게 좋음.

아이템 42 p.220 {}라는 타입이 있는게 신기하고, 기본형도 포함가능한게 신기 

아이템 43 p.223 애플리케이션이 실행되는 동안 속성을 할당하면 실행 시점에서 보강을 적용할 방법이 없습니다. ⇒ 이해 ❌ , 특히 웹 페이지 내의 HTML 엘리먼트를 조작할 때 ~ ⇒ 전역으로 적용이 되는데 그런 문제가 왜 생기지? (속성을 추가하기전에는 있고, 속성을 추가한 후에는 있다는 얘기인가?)

아이템 44 p.225 타입커버리지, 보아즈 프로젝트 ⇒ 2849 / 2919 97.60%, 타입스크립트 사용시 좀 더 라이브러리 사용에 신중해야겠다.
