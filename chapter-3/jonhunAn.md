# 3장 타입추론

- 구조분해(deserialize) 처리 시 모든 객체의 타입을 체크함으로 장점 (111)
- eslint `no-inferrable-types` 옵션 사용 → 불필요한 타입명시 확인 가능 (115)
- 타입 확장에 대한 이해
    
    ```jsx
    interface Vector3 { x: number; y: number; z: number; }
    function getComponent (vector: Vector, axis: 'x' | 'y' | 'z') {
    return vector[axis];
    }
    
    let x = 'x';
    let vec = fx: 10, y: 20, z: 301;
    getComponent (vec, x) ;
    // ~'string' 형식의 인수는 '"X"| "y" | "2"'
    // 형식의 매개변수에 할당될 수 없습니다.
    
    //런타임환경에는 오류가 없지만 편집기에서는 오류를 나타냄
    //x의  타입이 할당 시점에서 타입 넓히기로 string 타입으로 잡기때문
    ```
    
    - [https://github.com/InfraBlockchain/blockchat-server/pull/509/files/9f00e4dde74824d924c1923a856a17be98dbd55e](https://github.com/InfraBlockchain/blockchat-server/pull/509/files/9f00e4dde74824d924c1923a856a17be98dbd55e) (policy.service.ts) 왜 안되는지 얘기해보기
- const vs as const 차이 (122~123)
    
    ```jsx
    const v1 ={
    x: 1,
    y: 2,};
    // 타입은 { x: number; y: number; }
    
    const v2 = {
    X: 1 as const,
    y: 2,
    };
     // 타입은 { x: 1; y: number; }
    const v3 = {
    X: 1, 
     y: 2,
    } as const; 
    // 타입은 { readonly x: 1; readonly y: 2; }
    ```
    
    - as const 사용 → 최대한 좁은 타입으로 추론함
    - as const로 할당받은 변수는 deep-copy , const 할당변수는 shallow-copy (144)
- 타입좁히기 (124~125)
    - 분기처리,
    - instance of → 제일많이사용 try catch에서 Error 타입 체크
    - **‘a’ in a A → 근데 이건 같은 프로퍼티 있으면 문제생김**
    - **Array.isArray()**
    - **null 은 object 형임**
- `is` 사용자 정의 타입가드 (126)
    
    ```jsx
    const jacksons = [Jackie', 'Tito','Jermaine', 'Marlon', 'Michael'];
    const members = ['Janet', 'Michael'). map
    who => jackson5. find(n => n === who)
    ); // 타입은 (string | undefined)[]
    
    filter 함수를 사용해 undefined를 걸러 내려고 해도 잘 동작하지 않을 겁니다.
    const members = ['Janet', 'Michael']. map(
    who => jackson5. find (n => n === who)
    ).filter (who => who !== undefined);
    // 타입이 (string 1 undefined)[]
    
    이럴 때 타입 가드를 사용하면 타입을 좁힐 수 있습니다.
    function isDefined<T>(x: T | undefined): x is T 
    { return × !== undefined;
    }
    
    const members = ['Janet', 'Michael'].map(
    who => jackson5. find (n => n === who) ).filter (isDefined); 
    // 타입이 string[]
    ```
    
- 안전한 타입으로 속성을 추가하려면 객체전개 `{…A, …B}` 사용 (131)