# 4장  타입설계

- 객체의 속성이름을 함수의 매개변수로 받을 때
    - string 타입보단 `keyof T`를권장 (182) → 굉장히 좋은 방식같음
- ts branding 기법 (202) 
`T & {_brand: “name”}`
    
    ```jsx
    type Meters = number & {_brand: 'meters'};
    type Seconds = number & {_brand: 'seconds'};
    const meters = (m: number) => m as Meters;
    const seconds = (s: number) => s as Seconds;
    const onekm = meters (1000);
    //타입이 Meters
    const onevin = seconds(60); 
    // 타입이 Seconds
    
    //number 타입에 상표를 붙여도 산술 연산 후에는 상표가 없어지기 때문에 
    //실제로 사용하기에는 무리가 있습니다.
    
    const tenkm = oneKm * 10;
    const v = oneKm / oneMin;
    //타입이 number
    //타입이 number
    
    // 타입 스크립트는 덕타이핑 기반이기 때문에 
    // 값을 구분하기위해 공식명칭이 필요한경우 {_brand: "name"} 사용
    ```