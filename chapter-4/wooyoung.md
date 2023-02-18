### p. 155

테이블(코드의 타입)을 보여주면 순서도(코드의 로직)은 별로 필요하지 않다

→ 정확하게 무슨 뜻일까?

### p.175

인터페이스 유니온 예시

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
type Layer = FillLayer | LineLayer | PointLayer;

const drawLayer = (layer: Layer) => {
  if (layer.type === "fill") {
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

타입 분기 이후 layer가 포함된 동일한 코드가 반복되는것이 어수선해보임

→ 그러면 어떻게 간단하게 할 수 있을까?

### p.198

이름을 지을 때에는 포함된 내용나 계산 방식이 아니라 데이터 자체가 무엇인지 고려해야한다

INodeList보다는 Directory가 더 의미있다. 왜냐하면 Directory는 구현의 측면이 아니라 개념적인 측면에서 디렉터리를 생각하게 하기 때문이다.

→ 구현의 측면? 개념적인 측면? 말이 잘 이해가 안간다.
