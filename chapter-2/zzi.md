### 아이템 6 편집기를 사용하여 타입 시스템 탐색하기

---

편집기에서 타입스크립트 언어 서비스를 적극 활용해야 하고, 편집기를 사용하면 어떻게 타입 시스템이 동작하는지, 그리고 타입스크립트가 어떻게 타입을 추론하는지 개념을 잡을 수 있다.

### 아이템 7 타입이 값들의 집합이라고 생각하기

---

‘할당 가능한 값들의 집합’이 타입이라고 생각하면 이해하기 편하다.

`never`는 타입스크립트에서는 가장 작은 집합이다. `never` 타입으로 선언된 변수의 범위는 공집합이기 때문에 아무런 값도 할당할 수 없다. 

> **`never` 타입이 왜 필요할까?**
> 
> 
> > 아무런 값도 할당할 수 없는데 왜 필요할까? 라는 생각을 하게되었다. 찾아보니 `never`는 일반적으로 함수의 리턴 타입으로 사용된다고 한다. 함수의 리턴 타입으로 `never`가 사용될 경우, **항상 오류를 출력하거나 리턴 값을 절대로 내보내지 않음을 의미한다.**
> > 

```jsx
const x: never = 12; // '12' 형식은 'never' 형식에 할당할 수 없습니다.
```

그 다음으로 작은 집합은 유닛타입이라고도 불리고, 리터럴 타입이다. 리터럴 타입은 한 가지 값만 포함하는 타입이다. 

```jsx
type A = 'A';
type B = 'B';
type Twelve = 12;
```

두 개 혹은 세개로 묶으려면 유니온 타입을 사용해야한다. 유니온 타입은 값 집합들의 합집합을 일컫는다.

```jsx
type AB = 'A' | 'B';
type AB12 = 'A' | 'B' | 12;
```

& 연산자는 두 타입의 인터섹션을 계산합니다. 언뜻보기에 Person과 LifeSpan 인터페이스는 공통으로 가지는 속성이 없기 때문에, PersonSpan 타입을 never로 예상하기 쉽습니다. **그러나 타입 연산자는 인터페이스의 속성이 아닌 값의 집합에 적용됩니다.** 즉, Person와 LifeSpan의 인터섹션은 Person의 범위와 LifeSpan의 범위의 인터섹션이다. 

```tsx
interface Person {
  name: string;
}

interface LifeSpan {
  birth: Date;
  death?: Date;
}

type PersonSpan = Person & LifeSpan;

const ps: PersonSpan = {
  name: "zzi",
  birth: new Date("1999/04/20"),
};
```

### 아이템 8 타입 공간과 값 공간의 심벌 구분하기

---

타입스크립트의 심벌은 타입 공간이나 값 공간 중의 한 곳에 존재합니다. 심벌은 이름이 같더라도 속하는 공간에 따라 다른 것을 나타낼 수 있기 때문에 혼란스러울 수 있다.

```tsx
interface Cylinder {
	radius: number;
	height: number;
}

const Cylinder = (radius: number, height: number) => ({radius, height)};
```

interface Cylinder에서 Cylinder는 타입으로 쓰이고 const Cylinder에서 Cylinder는 값으로 쓰이며, 서로 아무런 관련이 없다. 

const, enum은 상황에 따라 타입과 값 두 가지 모두 가능한 예약어입니다. 하지만 연산자 중에서도 타입에서 쓰일 때와 값에서 쓰일 때 다른 기능을 하는 것들이 있다. 

```tsx
type T1 = typeof p; // 타입은 Person
type T2 = typeof email; // 타입은 (p: Person, subject: string, body: string) => Response

const v1 = typeof p; // 값은 "object"
const v2 = typeof email; // 값은 "function"
```

### 아이템 9 타입 단언보다는 타입 선언을 사용하기

---

타입스크립트에서 변수에 값을 할당하고 타입을 부여하는 방법은 타입 선언과 타입 단언 두 가지 이다.

```tsx
interface Person ( name: string; };

// 1. 타입 선언
const alice: Person = { name: 'Alice' };
// 2. 타입 단언 
const bob = { name: 'Alice' } as Person
```

타입 단언보다는 타입 선언을 사용해야한다.
⇒ 그 이유는 타입 단언은 강제로 타입을 지정하는 것이므로 타입체커에게 오류를 무시하라고 하는것 이기 때문! 

하지만 타입 단언이 꼭 필요한 경우도 있다.
⇒ 타입 체커가 추론한 타입보다 개발자가 판단하는 타입이 더 정확할 때 의미가 있다. 예를 들어, DOM 엘리먼트에 대해서는 타입스크립트는 DOM에 접근할 수 없기 때문에 개발자들이 더 정확하게 알고 있을 것이다. 

### 아이템 10 객체 래퍼 타입 피하기

---

타입스크립트 객체 래퍼 타입은 지양하고, 기본형 타입을 사용해야 한다. 

⇒ 오해하기 쉽고, 굳이 그렇게 할 이유가 없기 때문에

### 아이템 11 잉여 속성 체크의 한계 인지하기

---

타입이 명시된 변수에 객체 리터럴을 할다할 때 타입스크립트는 해당 타입의 속성이 있는지, 그리고 ‘그 외의 속성은 없는지’ 확인 합니다.

```tsx
interface Room {
	numDoors: number;
	ceilingHeightFt: number;
}

const r: Room = { // 오류 발생
	numDors: 1,
  ceilingHeightFt: 10,
	elephant: 'present',
} 
```

```tsx
const obj = {
	numDors: 1,
  ceilingHeightFt: 10,
	elephant: 'present'
}

const r: Room = obj; // 정상 
```

첫 번째 예제에서는 ‘잉여 속성 체크’ 라는 과정이 수행되었다. 그러나 잉여 속성 체크는 조건에 따라 동작하지 않는다.  잉여 속성 체크는 할당 가능 검사와는 별도의 과정이라는 사실을 알아야한다. 

객체 리터럴을 사용하면 잉여 속성 체크가 적용되고, 그렇지 않은 경우는 잉여 속성 체크가 적용되지 않는다. 
또한 타입 단언문을 사용할 때도 잉여 속성 체크가 적용되지 않는다. 

### 아이템 12 함수 표현식에 타입 적용하기

---

타입스크립트에서는 함수 표현식을 사용하는 것이 좋다. 함수의 매개변수부터 반환값까지 전체를 함수 타입으로 선언하여 함수 표현식에 재사용할 수 있다는 장점이 있기 때문이다.

```tsx
type BinaryFn = (a: number, b:number) => number;
const add: BinaryFn = (a,b) => a + b;
const sub: BinaryFn = (a,b) => a - b;
const mul: BinaryFn = (a,b) => a * b;
const div: BinaryFn = (a,b) => a / b;
```

### 아이템 13 타입과 인터페이스의 차이점 알기

---

타입을 정의하는 방법은 타입을 사용하는 것과 인터페이스를 사용하는 것 두 가지가 있다.

**비슷한 점** 

- 명명된 타입
- 인덱스 시그니처
- 제네릭 가능
- 인터페이스는 타입을 확장할 수 있고, 타입은 인터페이스를 확장할 수 있다.
    - 주의할 점은 인터페이스는 유니온 타입 같은 복잡한 타입을 확장하지 못하고, 복잡한 타입을 확장하려면 타입과 &를 사용해야 함
- 클래스 구현

**다른 점**

- 인터페이스는 보강이 가능
    
    ```tsx
    interface IState = {
    	name: string;
    }
    
    interface IState = {
    	population: number;
    }
    
    const wyoming: IState ={
    	name:'Wyoming',
    	population: 50000,
    } // 정상
    ```
    

그러면 타입과 인터페이스 중 어느 것을 사용해야할까?

> 타입을 써야하는 경우
> 
> - 복잡한 타입인 경우

> 인터페이스를 써야하는 경우
> 
> - 변경될 가능성이 있는 API에 대한 타입 선언

두 가지 모두 표현 될 수 있는 경우라면 일관되게 사용해야 함! ⇒ 일관되게 인터페이스를 사용했다면 인터페이스를 사용하고, 일관되게 타입을 사용했다면 타입을 사용하면 된다.

### 아이템 14 타입 연산과 제너릭 사용으로 반복 줄이기

---

DRY(don’t repeat yourself) 원칙을 타입에도 최대한 적용을 해야한다.
반복을 피하는 방법들..

- 중복되는 타입들은 타입에 이름을 붙여 반복을 줄여야 함
- extends를 사용해 인터페이스 필드의 반복을 피해야 함
- 제너릭 사용

### 아이템 15 동적 데이터에 인덱스 시그니처 사용하기

---

인덱스 시그니처

```tsx
type Rocket={ [property:string]: string};
const rocket: Rocket={
	name: "Falcon 9",
	variant: "v1.0",
};
```

인덱스 시그니처를 사용하면 유연하게 매핑을 표현할 수 있지만 잘못된 키를 포함해 모든 키를 허용한다는점 (name 대신 Name으로 작성해도 유효한 Rocket 타입이 된다), 키마다 다른 타입을 가질 수 없다는 점(rocket에 number인 타입을 갖고싶은데 string 밖에 안된다), 타입스크립트 언어 서비스를 사용하지 못한다는 점(name을 입력할 때 자동 완성 기능이 동작하지 않는다.)

### 아이템 16 number 인덱스 시그니처보다는 Array,튜플,ArrayLike를 사용하기

---

자바스크립트는 숫자 키도, 문자열 키도 모두 문자열 키로 인식한다.

```tsx
const x =[1, 2, 3];
Object.keys(x) // ['0', '1', '2']
```

타입스크립트에서는 숫자 키를 허용하고, 문자열 키와 다른 것으로 인식한다.

```tsx
interface Array<T>{
  [n: number]: T;
}
```

하지만 Object.keys 같은 구문은 여전히 문자열로 반환된다. string이 number에 할당될 수 없기 때문에, 예제의 마지막 줄이 이상하게 보일것 이다. 그러나 배열을 순회하는 코드 스타일에 대한 허용이라고 생각하는것이 좋다.

```tsx
const xs = [1, 2, 3];
const keys = Object.keys(xs); // string[]
for (const key in xs) {
	key; // string
  const x = xs[key]; // number
} 
```

인덱스 시그니처가 number로 표현되어 있다면 값이 number여야 한다는 것을 의미하지만, 실제 런타임에 사용되는 키는 string 이다. number 인덱스 시그니처를 사용할 때 혼란스러울 수 있으므로 Array,튜플,ArrayLike를 사용하는것이 좋다. 

인덱스에 신경 쓰지 않는다면, `for - of` 문
인덱스의 타입이 중요하다면, `forEach` 문
루프 중간에 멈춰야 한다면, `for(;;)` 문을 사용하는 것이 좋다.

타입이 불확실하다면, `for - in` 문은 `for - of` 문 또는  `for(;;)` 문보다 몇 배 느리다.

### 아이템 17 변경 관련된 오류 방지를 위해 readonly 사용하기

---

- 함수가 매개변수를 수정하지 않는다면 readonly로 선언하는 것이 좋다.
- readonly로 선언하면 변경하면서 발생하는 오류를 방지할 수 있고, 변경이 발생하는 코드를 쉽게 찾을 수 있다.
- readonly는 얕게 동작한다는 것을 명심해야 한다.

### 아이템 18 매핑된 타입을 사용하여 값을 동기화하기

---

1. 보수적 접근법 또는 실패에 닫힌 접근법

정확하지만 너무 자주 업데이트될 가능성이 있다

```tsx
function Update(oldProps: Props, newProps: Props){

    let k: key of Props;
    for(k in oldProps){
   	if(oldProps[k]!==newProps[k]
         if( k!=="onClick") return true;
    }
    return fasle;
 }
```

1. 실패에 열린 접근법 

불필요한 반복을 줄이지만, 실제 업데이트가 필요한 경우가 누락될 가능성이 있다

```
	function Update(oldProps: Props, newProps: Props){
    	return(
        	oldProps.x !== newProps.x ||
          ...//클릭여부 외 모든 속성을 비교
        );
    }
```

1. 매핑된 타입과 객체 사용하기

매핑된 타입은 한 객체가 또 다른 객체와 정확히 같은 속성을 가지게 할때 이상적이다. 

```tsx
const REQUIRES_UPDATE: {[k in keyof Props]: boolean}={
  xs: true,
  ys: true,
  xRange: true,
  yRange: true,
  color: true,
  onClick: false,
};

function Update(oldProps: Props, newProps: Props){
	let k: keyof Props;
	for(k in oldProps){
  	if(oldProps[k]!==newProps[k] && REQUIRES_UPDATE[k]) {
			return true;
		}
  }
	return false;
}
```

### References

---

- [https://ui.toast.com/posts/ko_20220323](https://ui.toast.com/posts/ko_20220323)
