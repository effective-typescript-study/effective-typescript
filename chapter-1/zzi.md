
### 아이템 1 타입스크립트와 자바스크립트의 관계 이해하기

---

- 모든 자바스크립트는 타입스크립트이지만, 모든 타입스크립트가 자바스크립트는 아니다.
- 타입스크립트의 타입 시스템은 런타임에 오류를 발생시킬 코드를 미리 찾아낸다. 하지만 모든 오류를 찾아내지는 않는다.
- 자바스크립트의 런타임 동작을 모델링하는 것은 타입스크립트 타입 시스템의 기본 원칙이다. 하지만 다른 언어였다면 런타임 오류가 될 만한 코드가 있고, 반대로 정상 동작하는 코드에 오류를 표시하기도 한다.

### 아이템 2 타입스크립트 설정 이해하기

---

타입스크립트 설정은 커맨드 라인을 이용하는 것 보다는 tsconfig.json을 사용하는 것이 좋고, noImplicitAny와 strictNullCheck는 true로 설정하는것이 좋다.

- noImplicitAny : 변수들이 미리 정의된 타입을 가져야 하는지 여부
- strictNullCheck : null과 undefined가 모든 타입에서 허용되는지 확인

### 아이템 3 코드 생성과 타입이 관계없음을 이해하기

---

- **타입 오류가 있는 코드도 컴파일이 가능하다.**
    
    타입스크립트 컴파일러는 두 가지 역할을 수행하고, 이 두 가지가 서로 완벽히 독립적으로 이루어진다.
    
    - 최신 타입스크립트/자바스크립트를 브라우저에서 동작할 수 있도록 구버전의 자바스크립트로 트랜스파일 한다.
    - 코드의 타입 오류를 체크한다.
    
    **컴파일은 타입 체크가 독립적으로 동작하기 때문에, 타입 오류가 있는 코드도 컴파일이 가능하다.**
    
    > 트랜스파일이라고 했다가 컴파일이라고 해서 헷갈렸는데 여기서의 컴파일은 첫 번째 경우를 말하는 것 같다 ! (compile이라는 용어가 더 큰 범주에 속하게 되고, 그 안에 transpile이라는 개념이 있게 되는거라 틀린 말은 아님) 
    트랜스파일 : **한 언어로 작성된 소스 코드를 비슷한 수준의 추상화를 가진 다른 언어로 변환**
    컴파일 : **한 언어로 작성된 소스 코드를 다른 언어로 변환**
    > 
    > 
    > typescript는 **transpiler or compiler?** 
    > 
    > > typescript 의 경우는 javascript로 변환되는데, 뭉뚱 그려 많은 사이트에서는 컴파일링 된다고는 하지만, 거의 **같은 수준의 추상화** 가 되는것이라 더 정확히 말하면 트랜스파일링 된다고 말할 수 있다.
    > > 
- **런타임에는 타입 체크가 불가능하다.**
    
    ```jsx
    interface Square {
        width: number;
    }
    
    interface Rectangle extends Square {
        height: number;
    }
    
    type Shape = Square | Rectangle;
    
    function caculateArea(shape: Shape){
        if (shape instanceof Rectangle) {
            return shape.width * shape.height;
        }
    }
    ```
    
    ⇒ instanceof 체크는 런타임에 일어나지만, Rectangle은 타입이기 때문에 런타임에 아무런 역할을 할 수 없다. 
    
    instanceof 대신에 in을 사용하여 해결할 수 있다.
    
    ```jsx
    function caculateArea(shape: Shape){
        if ('height' in shape) {
            return shape.width * shape.height;
        }
    }
    ```
    
    또 다른 해결법은 런타임에 접근이 가능한 타입 정보를 명시적으로 저장하는 ‘태그’ 기법이 있다.
    
    ```jsx
    interface Square {
    		kind: 'square';
        width: number;
    }
    
    interface Rectangle extends Square {
    		kind: 'rectangle';
        height: number;
    }
    
    type Shape = Square | Rectangle;
    
    function caculateArea(shape: Shape){
        if (shape.kind === 'rectangle') {
            return shape.width * shape.height;
        }
    }
    ```
    
- **타입 연산은 런타임에 영향을 주지 않는다.**
    
    ```jsx
    function asNumber(val: number | string): number {
    	return val as number;
    }
    
    // 변환된 js 코드
    function asNumber(val) {
    	return val
    }
    ```
    
    변환된 js를 보면 코드에 아무런 정제 과정이 없다. 값을 정제하기 위해서는 런타임의 타입을 체크해야 하고 자바스크립트 연산을 통해 변환을 수행해야 한다.
    
    ```jsx
    function asNumber(val: number | string): number {
    	return typeof(val) === 'string' ? Number(val) : val;
    }
    ```
    
- **런타임의 타입은 선언된 타입과 다를 수 있다.**
API를 사용하는 비동기 함수 등에서 명시된 것과 다른 타입을 받아올 가능성이 있다. 그러므로 선언된 타입이 언제든지 달라질 수 있다는 것을 명심해야 한다.
- **타입스크립트 타입으로는 함수를 오버로드할 수 없다.**
타입스크립트에서는 타입과 런타임의 동작이 무관하기 때문에, 함수 오버로딩은 불가능합니다. 하나의 함수에 대해 여러 개의 선언문을 작성할 수 있지만, 구현체는 오직 하나뿐이다.
- **타입스크립트 타입은 런타임 성능에 영향을 주지 않는다.**
타입과 타입 연산자는 자바스크립트 변환 시점에 제거되기 때문에, 런타임의 성능에 아무런 영향을 주지 않는다.

### 아이템 4 구조적 타이핑에 익숙해지기

---

자바스크립트는 본질적으로 **덕 타이핑 기반**이다. 만약 어떤 함수의 매개변수 값이 모두 제대로 주어진다면, 그 값이 어떻게 만들어졌는지는 신경 쓰지 않고 사용한다. 타입스크립트는 이를 모델링 하기 위해 **구조적 타이핑**을 사용한다. 

- **덕 타이핑**
객체가 어떤 타입에 부합하는 변수와 메서드를 가질 경우, 객체를 해당 타입에 속하는 것으로 간주하는 방식
- **구조적 타이핑**
실제 구조와 정의에 의해 결정되는 타입 시스템
    
    ⇒ 구조적 타이핑에 단점으로, **의도하지 않았지만 동일한 타입을 가지는 경우 의도치 않게 동일한 유형으로 간주될 수 있다.**
    
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
    const v: NamedVector = { x: 3, y: 4, name: 'mgh' }
    calculateLength(v)
    ```
    
    위 예제는 명목적 타입 시스템을 기준으로 Vector2D를 사용하도록 의도한 코드이다. 하지만 구조적 타이핑 언어에서는 전혀 다른 개념으로 이해해야 한다. **Vector2D의 속성에 해당하는 값이 값을 넣는 타입에 속성으로 존재하는가** 로 이해해야 한다.
    
    타입은 봉인되어 있지 않다. 즉, [함수 매개변수 속성]이 [매개변수타입에 선언된 속성]만 가지고, 다른 속성은 가지지 않을 것이라고 생각해서는 안된다. 
    
    ⭐️ 구조적 타이핑을 제대로 이해 한다면 오류인 경우와 오류가 아닌 경우의 차이를 알 수 있고, 더욱 견고한 코드를 작성할 수 있다!
    

### 아이템 5 any 타입 지양하기

---

- any 타입에는 타입 안정성이 없다.
- any는 함수 시그니처를 무시해 버린다.
- any 타입에는 언어 서비스가 적용되지 않는다.
타입스크립트 언어 서비스는 자동완성 기능과 적절한 도움말을 제공한다. 그러나 any 타입인 심벌을 사용하면 아무런 도움을 받지 못한다.
- any 타입은 코드 리팩터링 때 버그를 감춘다.
- any는 타입 설계를 감춰버린다.
설계가 명확히 보이도록 타입을 일일이 작성하는 것이 좋다.
- any는 타입시스템의 신뢰도를 떨어뜨린다.
타입 체커가 사람의 실수를 잡아주고 코드의 신뢰도를 높인다. 하지만 런타임에 타임 오류를 발견하게 된다면 타입 체커를 신뢰할 수 없게된다. any 타입을 쓰지 않으면 런타임에 발견될 오류를 미리 잡을 수 있고 신뢰도를 높일 수 있다.

### References

---

- [https://ideveloper2.tistory.com/166](https://ideveloper2.tistory.com/166)
- [https://vallista.kr/덕-타이핑과-구조적-타이핑/](https://vallista.kr/%EB%8D%95-%ED%83%80%EC%9D%B4%ED%95%91%EA%B3%BC-%EA%B5%AC%EC%A1%B0%EC%A0%81-%ED%83%80%EC%9D%B4%ED%95%91/)
