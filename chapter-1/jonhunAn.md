# 1장 타입스크립트 이해하기

타입스크립트 타입 시스템은 열려있다.(23~25p.)

```jsx
interface Vector3D {
x: number;
y: number;
z: number;
}

const v: Vertor3D = {x:3, y:4, z:5};

calculateLength(v);

function calculateLength(v: Vector3D){
let length = 0;
for (const axis of Object.keys(v)){
const coord = v[axis]; //axis error 
....

// axis any 타입으로 잡음 
ex) const vec3D: Vector3D = {.... address: "12311"}
```

```jsx
class C {
  foo: string;
    constructor(foo: string) {
        this.foo = foo;
    }
}

const c = new C('instance of C');
const d: C = { foo: 'object literal' }; // 정상

// 그래서 구조적으로는 필요한 속성과 생성자가 존재하기 때문에 문제가 없다.
// 만약 C의 생성자에 단순 할당이 아닌 연산 로직이 존재한다면,
// d의 경우는 생성자를 실행하지 않으므로 문제가 발생하게 된다. 이러한 부분이 C타입의 매개변수를 선언하여 C 또는 서브클래스임을 보장하는 C++이나 자바 같은 언어와 매우 다른 특징이다.
```

any 타입을 지양하자(28~29p)

```jsx
interface Person {
first: string;
last: string;
}

// Person property 변경
interface Person {
firstName: string;
last: string;
}

const formatName = (p: Person) =>`${p.firstName} ${p. last}`;
// any 타입에 대한 p 타입의 rename 기능 반영되지않음(실수하기 쉬움)
const formatNameAny = (p: any) => `${p.first} ${p. last}`;
```