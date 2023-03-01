
### 아이템 45 devDependencies에 typescript와 @types 추가하기

---

npm은 자바스크립트 라이브러리 저장소(npm 레지스트리)이고, 프로젝트가 의존하고 있는 라이브러리들의 버전을 지정하는 방법(package.json)을 제공한다. 

npm은 세 가지 종류의 의존성을 구분하여 관리하고, 각각의 의존성은 package.json 파일 내의 별도 영역에 들어 있다.

- dependencies 
현재 프로젝트를 실행하는데 필수적인 라이브러리들이 포함
- devDependencies 
현재 프로젝트를 개발하고 테스트 하는데 사용되지만, 런타임에는 필요없는 라이브러리들이 포함
- peerDependencies
런타임에 필요하긴 하지만, 의존성을 직접 관리하지 않는 라이브러리들이 포함

타입스크립트는 개발 도구일 뿐 타입 정보는 런타임에 존재하지 않기 때문에 타입스크립트와 관련된 라이브러리는 일반적으로 devDependencies에 속한다. 

타입스크립트 프로젝트에서 공통적으로 고려해야할 의존성 두 가지가 있다.

**첫 번째, 타입스크립트 자체 의존성**
타입스크립트를 시스템 레벨로 설치할 수 있지만, 팀원들 모두가 항상 동일한 버전을 설치한다는 보장이 없기 때문에 devDependencies에 넣는것이 좋다.

**두 번째, 타입 의존성(@types)**
@types 라이브러리는 타입 정보만 포함하고 있으며 구현체는 포함하지 않는다. 라이브러리 자체가 dependencies에 들어가 있어도 @types 의존성은 devDependencies에 있어야 한다.

### 아이템 46 타입 선언과 관련된 세 가지 버전 이해하기

---

의존성 관리는 개발자들에게 매우 힘들다. 타입스크립트를 사용할 때도 의존성 관리가 힘들고 복잡하기 때문에 **라이브러리 버전, 타입 선언 버전, 타입스크립트의 버전**을 고려해야한다. 

라이브러리를 업데이트하는 경우, 해당 @types 역시 업데이트를 해야한다.

### 아이템 47 공개 API에 등장하는 모든 타입을 익스포트하기

---

공개 API 타입 정보를 숨기려 하지말고 라이브러리 사용자를 위해 익스포트하기 쉽게 만드는 것이 좋다.

### 아이템 48 API 주석에 TSDoc 사용하기

---

익스포트된 함수, 클래스, 타입에 주석을 달 때는 TSDoc 형태로 사용하자. TSDoc 형태의 주석을 달면 편집기가 주석 정보를 표시해 준다. 또한 타입 정보가 코드에 있기 때문에 TSDoc에는 타입 정보를 명시하면 안된다. 

### 아이템 49 콜백에서 this에 대한 타입 제공하기

---

자바스크립트의 this는 다이나믹 스코프로 호출하는 객체에 따라 그 의미가 변경된다. 따라서 this는 계속 변경될 수 있는 것으로 타입을 미리 설정을 해두지 않는다면 추후에 혼란이 올 수 있다. 

### 아이템 50 오버로딩 타입보다는 조건부 타입을 사용하기

---

**조건부 타입이란?**

`T extends U ? X : Y` , 타입은 T가 U에 할당될 수 있으면 타입은 X가 되고 그렇지 않다면 타입이 Y가 된다는 것을 의미한다. 

예를 들면,

```jsx
// 제네릭 `T`는 `boolean` 타입으로 제한.
// 제네릭 T에 true가 들어오면 string 타입으로, false가 들어오면 number 타입으로 data 속성을 타입 지정
interface isDataString<T extends boolean> {
   data: T extends true ? string : number;
   isString: T;
}

const str: isDataString<true> = {
   data: '홍길동', // String
   isString: true,
};

const num: isDataString<false> = {
   data: 9999, // Number
   isString: false,
};
```

오버로딩 타입이 작성하기는 쉽지만, 조건부 타입은 개별 타입의 유니온으로 일반화하기 때문에 타입이 더 정확해 진다. 그러므로 오버로딩 타입보다 조건부 타입을 사용하는것이 좋다.

### 아이템 51 의존성 분리를 위해 미러 타입 사용하기

---

`@types/node`가 필요하지 않는 집단에게는 Buffer 하나 때문에 `@types/node`를 설치해야한다.

```jsx
type ParseCSVHanlder = (contents: string | Buffer) => { [column: string]: string }[];

const parseCSV: ParseCSVHanlder = (contents) => {
  if (typeof contents === "object") {
    return parseCSV(contents.toString("utf-8"));
  }
  return [{ column: contents }];
};
```

하지만, 실제 필요한 선언부만 추출하여 라이브러리에 넣는 것을 고려해 볼 수 있다.

```jsx
// "Buffer" 대신 선언한 타입, 별도로 선언하여 구조적 타이핑 적용
type CSVBuffer = {
  toString: (encoding: string) => string;
};

type ParseCSVHanlder = (contents: string | CSVBuffer) => { [column: string]: string }[];

const parseCSV: ParseCSVHanlder = (contents) => {
  if (typeof contents === "object") {
    return parseCSV(contents.toString("utf-8"));
  }

  return [{ column: contents }];
};
```

### 아이템 52 테스팅 타입의 함정에 주의하기

---

- 단지 함수의 실행으로만 오류가 발생하지 않는지 체크하면 안된다. **반환값에 대한 타입 체크**도 해야하여 실행 결과에 대한 테스트도 해야한다.
- 함수 타입의 동일성과 할당 가능성의 차이점을 알아야한다.
- 타입 관련 테스트에서 any를 주의해야 한다. 더 엄격한 테스트를 위해 dtslint 같은 도구를 사용하는 것이 좋다.

### References

---

- [https://inpa.tistory.com/entry/TS-📘-타입스크립트-조건부-타입-완벽-이해하기](https://inpa.tistory.com/entry/TS-%F0%9F%93%98-%ED%83%80%EC%9E%85%EC%8A%A4%ED%81%AC%EB%A6%BD%ED%8A%B8-%EC%A1%B0%EA%B1%B4%EB%B6%80-%ED%83%80%EC%9E%85-%EC%99%84%EB%B2%BD-%EC%9D%B4%ED%95%B4%ED%95%98%EA%B8%B0)
- [https://1-blue.github.io/posts/이펙티브-타입스크립트-6장/#-item-51--의존성-분리를-위해-미러-타입-사용하기-](https://1-blue.github.io/posts/%EC%9D%B4%ED%8E%99%ED%8B%B0%EB%B8%8C-%ED%83%80%EC%9E%85%EC%8A%A4%ED%81%AC%EB%A6%BD%ED%8A%B8-6%EC%9E%A5/#-item-51--%EC%9D%98%EC%A1%B4%EC%84%B1-%EB%B6%84%EB%A6%AC%EB%A5%BC-%EC%9C%84%ED%95%B4-%EB%AF%B8%EB%9F%AC-%ED%83%80%EC%9E%85-%EC%82%AC%EC%9A%A9%ED%95%98%EA%B8%B0-)
