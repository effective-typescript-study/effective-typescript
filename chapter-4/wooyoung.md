# 타입 설계

타입시스템의 장점은 데이터 타입을 명확히 알 수 있어 코드를 이해하기 쉽다는 것이다.

## 아이템 28 유효한 상태만 표현하는 타입을 지향하기

효과적으로 타입을 설계하려면 유효한 유효한 상태만 표현하는 타입을 지향해야한다.

```tsx
interface RequestPending {
  state: "pending";
}
interface RequestError {
  state: "error";
  error: string;
}
interface RequestSuccess {
  state: "ok";
  pageText: string;
}
type RequestState = RequestPending | RequestError | RequestSuccess;

interface State {
  currentPage: string;
  requests: { [page: string]: RequestState };
}

function getUrlForPage(p: string) {
  return "";
}

function renderPage(state: State) {
  // state의 각각의 상태를 명시적으로 지정함
  const { currentPage } = state;
  const requestState = state.requests[currentPage];
  switch (requestState.state) {
    case "pending":
      return `Loading ${currentPage}...`;
    case "error":
      return `Error! Unable to load ${currentPage}: ${requestState.error}`;
    case "ok":
      return `<h1>${currentPage}</h1>\n${requestState.pageText}`;
  }
}

async function changePage(state: State, newPage: string) {
  state.requests[newPage] = { state: "pending" };
  state.currentPage = newPage;
  try {
    const response = await fetch(getUrlForPage(newPage));
    if (!response.ok) {
      throw new Error(`Unable to load ${newPage}: ${response.statusText}`);
    }
    const pageText = await response.text();
    state.requests[newPage] = { state: "ok", pageText };
  } catch (e) {
    state.requests[newPage] = { state: "error", error: "" + e };
  }
}
```

---

## 아이템 29 사용할 때는 너그럽게, 생성할 때는 엄격하게

매개변수의 재사용을 위해서 기본 형태(반환 타입)을 도입하는 것이 좋다.

사용하기 편리한 API일수록 반환 타입이 엄격하다.

---

## 아이템 30 문서에 타입 정보를 쓰지 않기

주석은 코드화 동기화 되지 않기 때문에, 타입 구문으로 타입 정보를 작성한다면 코드가 변경되면 정보가 정확하게 동기화 된다.

---

## 아이템 31 타입 주변에 null 값 배치하기

null과 null 이 아닌 값을 분리해서 타입을 체크해야한다.

"", null, undefined, 0, NaN → false값 반환

```tsx
if (!null) {
  console.log("null일때 실행되는 코드");
}
```

---

## 아이템 32 유니온의 인터페이스보다는 인터페이스의 유니온을 사용하기

인터페이스 유니온 타입을 사용하는게 더 정확하다.

→ 태그된 유니온

```jsx
type FillPaint = unknown;
type LinePaint = unknown;
type PointPaint = unknown;
type FillLayout = unknown;
type LineLayout = unknown;
type PointLayout = unknown;

interface FillLayer {
  type: "fill"; // '태그', 런타임에 어떤 타입의 layer가 사용되는지 판단할때 쓰임
  layout: FillLayout;
  paint: FillPaint;
}
interface LineLayer {
  type: "line";
  layout: LineLayout;
  paint: LinePaint;
}
interface PointLayer {
  type: "paint";
  layout: PointLayout;
  paint: PointPaint;
}

type Layer = FillLayer | LineLayer | PointLayer; // 인터페이스의 유니온

const drawLayer = (layer: Layer) => {
  if (layer.type === "fill") {
    // const {paint, layout} = layer;
    const { paint } = layer; // Type is FillPaint
    const { layout } = layer; // Type is FillLayout
  } else if (layer.type === "line") {
    const { paint } = layer; // Type is LinePaint
    const { layout } = layer; // Type is LineLayout
  } else {
    const { paint } = layer; // Type is PointPaint
    const { layout } = layer; // Type is PointLayout
  }
};
```

---

## 아이템 33 string 타입보다 더 구체적인 타입 사용하기

```tsx
type RecordingType = "studio" | "live";

interface Album {
  artist: string;
  title: string;
  releaseDate: Date;
  /** 이 녹음은 어떤 환경에서 이루어졌는지?*/
  recordingType: RecordingType;
}

const kindOfBlue: Album = {
  artist: "Miles Davis",
  title: "Kind of Blue",
  releaseDate: new Date("1959-08-17"),
  recordingType: "Studio",
  // ~~~~~~~~~~~~ Type '"Studio"' is not assignable to type 'RecordingType'
};
```

장점

1. 타입을 명시적으로 정의함으로써 다른 곳으로 값이 전달되어도 타입 정보가 유지된다.
2. 타입을 명시적으로 정의하고 해당 타입의 의미를 설명하는 주석을 붙여 넣을 수 있다.
3. keyof 연산자로 더욱 세밀하게 객체의 속성체크가 가능하다.

   ```tsx
   function pluck<T, K extends keyof T>(record: T[], key: K): T[K][] {
     return record.map((r) => r[key]);
   }

   type RecordingType = "studio" | "live";

   interface Album {
     artist: string;
     title: string;
     releaseDate: Date;
     recordingType: RecordingType;
   }

   declare let albums: Album[];
   pluck(albums, "releaseDate"); // Type is Date[]
   pluck(albums, "artist"); // Type is string[]
   pluck(albums, "recordingType"); // Type is RecordingType[]
   pluck(albums, "recordingDate");
   // ~~~~~~~~~~~~~~~ Argument of type '"recordingDate"' is not
   //                 assignable to parameter of type ...
   ```

---

## 아이템 34 부정확한 타입보다는 미완성 타입을 사용하기

실수가 발생하기 쉽고 잘못된 타입은 타입이 없는 것보다 못하다.

---

## 아이템 35 데이터가 아닌, API와 명세를 보고 타입을 만들기

예시 데이터가 아니라 명세를 참고해 타입을 생성해야한다.

---

## 아이템 36 아이템 해당 분야의 용어로 타입 이름 짓기

이름을 지을 때에는 포함된 내용나 계산 방식이 아니라 데이터 자체가 무엇인지 고려해야한다

INodeList보다는 Directory가 더 의미있다. 왜냐하면 Directory는 구현의 측면이 아니라 개념적인 측면에서 디렉터리를 생각하게 하기 때문이다.

→ 어떤 변수를 NodeList로 구현하니까 INodeList라고 이름을 짓는 것보다 변수가 디렉터리 데이터를 담고있으면 Directory로 짓는게 더 낫다.

---

## 아이템 37 공식 명칭에는 상표를 붙이기

타입스크립트는 구조적 타이핑(덕 타이핑)을 사용하기 때문에 값을 구분하기 위해서는 공식 명칭인 상표를 붙일 수 있다.

상표 기법은 타입시스템에서 동작하지만 런타임에 상표를 검사하는 것과 동일한 효과를 얻을 수 있다.

```tsx
type Meters = number & { _brand: "meters" };
type Seconds = number & { _brand: "seconds" };

const meters = (m: number) => m as Meters;
const seconds = (s: number) => s as Seconds;

const oneKm = meters(1000); // Type is Meters
const oneMin = seconds(60); // Type is Seconds

// 산술 연산 후에는 상표가 없어진다.
const tenKm = oneKm * 10; // Type is number
const v = oneKm / oneMin; // Type is number
```
