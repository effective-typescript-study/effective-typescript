# 2장

### p. 41

```tsx
interface Person {
  name: string;
}
interface Lifespan {
  birth: Date;
  death?: Date;
}
type PersonSpan = Person & Lifespan;

const ps: PersonSpan = {
  name: "Alan Turing",
  birth: new Date("1912/06/23"),
  death: new Date("1954/06/07"),
}; // OK
```

타입 연산자는 인터페이스의 속성이 아닌, 값의 집합(타입의 범위)에 적용된다. 그리고 추가적인 속성을 가지는 값도 여전히 그 타입에 속한다.

그래서 Person과 Lifespan을 둘 다 가지는 값은 인터섹션(&)에 속하게 된다.

name, birth, death 보다 더 많은 속성을 가지는 값도 PersonSpan 타입에 속한다.

```tsx
type PersonSpan = Person | Lifespan; // Type is never
```

유니온 타입에 속하는 값은 어떤 키도 없기에 유니온에 대한 keyof는 공집합(never)이다.
ts에서는 PersonSpan에 Person이 올지 Lifespan이 올지 모르니까 두 타입의 공통 속성에만 접근가능하다.

```tsx
keyof (A&B) = (keyof A) | (keyof B)
keyof (A|B) = (keyof A) & (keyof B)
```

![Untitled](%5B2%205%5D%20%E1%84%89%E1%85%B3%E1%84%90%E1%85%A5%E1%84%83%E1%85%B5%202%E1%84%8C%E1%85%A1%E1%86%BC%208cee6857aa1541c58564b721bdf1cff7/Untitled.png)

타입연산자는 인터페이스의 속성 아닌, 값의 집합(타입의 범위)에 적용된다

→ p.46와 연결에서 이해한게 맞는지 궁금해요

→ 타입의 속성에 대해서는 집합과 반대로, 타입의 값에 대해 볼때는 집합대로

### p. 53 ~ 56

|                | 타입 선언                                       | 타입 단언                                         |
| -------------- | ----------------------------------------------- | ------------------------------------------------- |
| 타입 부여      | const alice : Person = { name : ‘Alice’ }       | const bob = { name : ‘bob’ } as Person            |
| 타입 결과      | Person                                          | Person                                            |
| 타입 체커      | 할당되는 값이 해당 인터페이스를 만족하는지 검사 | 강제로 타입을 지정한 거라 타입 체크 오류를 무시함 |
| 타입 속성 추가 | 불가능, 잉여 속성 체크 동작                     | 가능                                              |
| 사용법         | 화살표함수의 리턴 타입 명시할때                 | 개발자가 판단하는 타입이 더 정확할 때 ex. DOM타입 |

- 잉여 속성 체크 : 타입이 명시된 변수에 `객체 리터럴` 을 할당할때 ts는 해당 타입의 속성이 있는지, 그 외의 속성은 없는지 확인(p.61)

→ 타입 선언과 단언을 정리해봤어요

### p. 57

몽키패치 : 런타임 프로그램의 어떤 기능을 수정해서 사용하는 기법, js에서는 주로 프로토타입을 변경하는 것이 해당된다.

유래 : 게릴라 패치 → 고릴라 패치(발음의 유사성) → 몽키 패치(고릴라가 너무 거대하고 위협적으로 들려 작고 귀여운 몽키로 )

### p. 90

자바스크립트의 객체는 키와 값의 모음. 키는 값은 어떤 것이든 될 수 있다.

```tsx
const x = {};
x[[1, 2, 3]] = 2; // 키를 객체로 사용해도 가능
console.log(x); // {'1,2,3': 2};
```

자바스크립트는 ‘해시 가능’ 객체라는 표현이 없기 때문에 만약, 문자열이 아닌 더 복잡한 객체를 키로 사용하려 한다면, 내부적으로 toString 메서드가 호출되어 객체를 문자열로 반환하여 키로 사용

- 해시 : 키를 테이블에 저장할 때 키 값을 Hash Method를 이용해 계산한 후, 데이터의 고유한 숫자값인 결과값을 배열의 인덱스로 사용하여 저장하는 방식

숫자를 키로 사용할 수 없어서 객체의 키로 숫자로 사용한다면, 문자열로 변환.

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

타입스크립트는 숫자 키를 허용하며 문자열 키와는 다른 것으로 인식 (p.91)

```tsx
// lib.es5.d.ts

interface Array<T> {
  ...
  [n: number]: T; // 숫자 키
}`

`const xs = [1, 2, 3];
const x0 = xs[0]; // OK
const x1 = xs['1'];
// ~~~~ Element implicitly has an 'any' type
//      because index expression is not of type 'number' ( 숫자 키와 문자열 키가 다름 )
```
