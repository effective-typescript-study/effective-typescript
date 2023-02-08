# 2장 타입스크립트의 타입시스템

타입스크립트는에서 타입은 ‘할당 가능한 값들의 집합’으로 생각하는 것이 좋다.

타입스크립트 용어와 집합 용어

| never | ∅ (empty set - 공집합) |
| --- | --- |
| Literal type | Single element set (원소가 1개인 집합) |
| Value assignable to T | Value ∈ T (member of) |
| T1 assignable to T2 | T1 ⊆ T2 (subset of) |
| T1 extends T2 | T1 ⊆ T2 (subset of) |
| T1 | T2 | T1 ∪ T2 (union) |
| T1 & T2 | T1 ∩ T2 (intersection) |
| unknown | Universal set - 전체집합 |

타입공간 vs 값 공간 

symbol

- JS에는 6개의 primitive 타입 (**Number, String, Boolean, null, undefined, Symbol)과** 1개의 객체 타입**(Object)**
- **symbol은 ES6에서 추가된 원시타입**
- 일반적으로 심볼 타입은 객체의 프로퍼티 키를 고유하게 설정함으로써 프로퍼티 키의 충돌을 방지하기 위해 사용
- **심볼은 Symbol 함수를 호출함으로써 생성**할 수 있다. 이때 생성되는 심볼은 **변경이 불가능한 원시 값**

한편, **Symbol 함수를 호출하면 매번 새로운(고유한) 심볼이 생성**된다. 일치 연산자(===)를 통해 이를 확인해 보자.

```jsx
const sym1 = Symbol();
const sym2 = Symbol();
const sym3 = Symbol('foo');
const sym4 = Symbol('foo');

console.log(sym1 === sym1);// trueconsole.log(sym1 === sym2);// falseconsole.log(sym3 === sym4);// false
```

그런데 심볼 타입에는 특이한 점이 하나 있다. 그것은 바로 Number, String, Boolean 타입과 달리 **new 연산자를 이용한 래퍼 객체의 생성이 불가능**하다는 점이다. new 연산자를 이용하여 래퍼 객체를 생성하려고 하면 TypeError가 발생한다. new 연산자를 이용할 수 없다는 것은 곧 Symbol 함수를 생성자로 사용할 수 없음을 의미한다.

```jsx
const sym = new Symbol();// Uncaught TypeError: Symbol is not a constructor
```

Number, String, Boolean 타입의 경우 new 연산자를 이용한 래퍼 객체의 생성이 가능하다. 이렇게 생성되는 래퍼 객체는 해당 타입의 원시 값을 저장하고 있고, 유용한 몇몇 메소드들을 가지고 있다. 만약 new 연산자를 이용하지 않고 단순히 Number, String, Boolean 함수를 호출하기만 하면 해당 타입의 원시 값이 생성되기만 하고 래퍼 객체는 생성되지 않는다.

보통 type,interface 다음은 타입

```
interface Cylinder {
  radius: number;
  height: number;
}

const Cylinder = (radius: number, height: number) => ({ radius, height });

function calculateVolume(shape: unknown) {
  if (shape instanceof Cylinder) {
    // instanceof는 타입이 아닌 함수를 참조한다.
    shape.radius; // '{}' 형식에 'radius' 속성이 없습니다.
  }
}
//질문! 무엇으로 참조될지
// shape instanceof Cylinder 에서 Cylinder 는 함수로 참조된다.
```

let, const 다음은 값입니다.

```jsx

class Cylinder {
  radius=1;
  height=1;
}
// 클래스가 타입으로 쓰일 때는 형태(속성과 메서드가 사용됨)

function calculateVolume(shape: unknown) {
  if (shape instanceof Cylinder) {
    shape// OK, type is Cylinder
    shape.radius// OK, type is number
  }
}
// 값으로 쓰일 때는 생성자가 사용됨.

const v = typeof Cylinder;
// Value is "function"
type T = typeof Cylinder;
// Type is typeof Cylinder// 문제는 클래스의 타입은 생성자의 타입이라는 것임.
// 제너릭 이용해 생성자 타입과 인스턴스 타입 전환

type C = InstanceType<typeof Cylinder>;
// Type is Cylinder
```

질문 ! js에서 존재하는 런타임 타입은 몇개? (6개)
6개는 무엇일까요? (string, number, boolean, undefined, object, function)

### typeof 연산자는 값에서 쓰일때와 타입으로 쓰일때 다른기능을 함

- 타입에서 쓰일 때 : 타입스크립트의 타입
- 값에서 쓰일 때 : 자바스크립트 런타임 typeof 연산자

```
interface Person {
 name: string;
}
const p: Person = { name: 'hee' };

type T = typeof p; // 타입은 Person
const v1 = typeof p; // 값은 'object'
```

### 클래스에서 typeof 를 사용한 경우는 다음과 같다.

- 타입으로 쓰일 때 : 인스턴스 타입이 아닌, **생성자 함수**
- 값으로 쓰일 때 : 'function'

```
class Cylinder {
    radius=1;
    height=1;
}

const v = typeof Cylinder; 
// 값이 function

type T = typeof Cylinder; 
// 타입이 class Cylinder, 즉 생성자 함수

declare let fn : T;
const c = new fn();	
// 타입이 Cylinder

type C = InstanceType<typeof Cylinder>	
// 타입이 Cylinder
```

만약 클래스의 인스턴스를 타입으로 사용하고 싶다면 다음과 같이 `InstanceType`를 작성하면 된다.

결론 

- 타입: type, interface
- 값: const, let, var
- 둘 다: class, 생성자함수, enum

```jsx
type blockchat_backend_leader = ‘DAWN’
// literal

type blockchat_backend = ‘DAWN’ | JAKE | JONNY
// unio
```

### 잉여속성 체크를 하는경우와 하지않는 케이스 (61)

- 임시변수를 도입하면 잉여속성 체크를 건너뛸 수 있음
    - 선언 시 할당 (잉여속성체크 처리)
    - 할당된 변수에 대한 다른 변수할당 (처리 x)

### 인터페이스 vs 타입 차이점

- 인터페이스는 유니온 타입같은 복잡한 타입을 확장하지 못함 (71)
    - 복잡한 타입확장을 하고싶다면 type과 &(intersection)연산자 사용
    - 인터페이스는 확장은 가능하나 유니온타입 적용불가
        - 유니온타입 확장을 위해서는 type 키워드사용
    - type은 유니온 및 매팁된 타입 조건부타입 등 다양한 기능제공 (튜플과 배열 처리도 type이 조금 더 이상적임)
        - interface도 가능하나 비효율적(73)
- 선언 병합 → 타입의 속성 확장
    - 인터페이스만 가능

### 타입연산과 제네릭사용

- 타입확장의 경우 인터섹션 연산자사용(77)
- typeof의 선언순서 주의(81)
    - 타입정의를 먼저하고 값이 그타입에 할당 가능하다고 선언하는것이 좋음
    → 타입의 명확성 및 예상치못한 타입변동 예방
- index signature → 레포 케이스 보여주기 (85)

### 아이템16 인덱스 시그니처 대체

- js에는 hash기능없음 (90)
    - 자바와 같은 언어는 객체 hash 값으로 유니크 id 객체확인
    - 객체 키로 숫자를 사용할 수 없음
        - 키 참조 시 toString 문자열로 자동적으로 배열 인덱스 찾음
        - [1], [’1’] 동일동작
- 타입스크립트는 숫자 키 허용
    
    ```jsx
    interface Array<T>{
    	//...
    	[n: number]: T;
    }
    // 컴파일 단계에서만 동작함
    ```
    
    - for - of 혹은 Array.prototype.forEach사용 (92)
        - forEach 사용하면 비동기 함수 처리 불가함으로 유의해야함!!
- for - in 함수는 다른 반복문에 비해 시간복잡도가 낮음
- [ArrayLike](https://yceffort.kr/2021/11/array-arraylike-promise-promiselike) 타입 사용을 권장함
    
    → 특정 길이를 가지는 배열과 비슷한 형태의 튜플 사용 시 사용
    
    - 키는 여전히 문자열 사용함을 주의
- readonly를 사용하여 정합성보장
    - 배열은 메모리 할당이기 때문에 shallow-copy 처리되므로  [deep-copy](https://ljlm0402.netlify.app/javascript/performance/)필요
        - deepcopy (lodash 혹은 json.parse 사용)

<aside>
💡 참고자료

[Array vs ArrayLike, Promise vs PromiseLike](https://yceffort.kr/2021/11/array-arraylike-promise-promiselike)

[JavaScript로 Deep Copy 하는 여러 방법](https://chaewonkong.github.io/posts/js-deep-copy.html)

[성능 비교 📈 ▻ Lodash vs ES6](https://ljlm0402.netlify.app/javascript/performance/)

</aside>