4장은 실제 코드를 작성할 때 유용한 것들이 많이 포함되어 있었다. 

### 아이템 28 유효한 상태만 표현하는 타입을 지향하기

---

유효한 상태와 무효한 상태를 둘 다 표현하는 타입은 혼란을 초래하기 쉽기 때문에 유효한 상태만 표현하는 타입을 지향해야 한다. 

### 아이템 29 사용할 때는 너그럽게, 생성할 때는 엄격하게

---

매개변수는 타입의 범위가  넓어도 되지만, 결과를 반환할 때는 타입의 범위가 더 구체적이어야 한다. 

### 아이템 30 문서에 타입 정보를 쓰지 않기

---

- 타입스크립트의 타입 구문 시스템은 간결하고, 구체적이며, 쉽게 읽을 수 있도록 설계되었기 때문에 함수의 입력과 출력의 타입을 코드로 표현하는 것이 주석보다 더 나은 방법이다.
- 변수명에 타입 정보를 넣지 않도록 한다.
예를 들어 변수명을 `ageNum` 보다는 `age`로 하고, 그 타입이 `number`임을 명시하는 게 좋다.
하지만 단위가 있는 숫자들은 예외, (`timeMs`는 `time`보다 훨씬 명확하고, `temperatureC`는 `temperature`보다 훨씬 명확하다.)

### 아이템 31 타입 주변에 null 값 배치하기

---

strictNullChecks 설정을 하지 않는다면 설계적 결함을 발견하지 못할 수도 있다.

예를 들면, 아래의 코드는 최댓값이나 최솟값이 0인 경우, 값이 덧씌워져 버린다. 또한 `nums` 배열이 비어져있다면 함수는 `[undefined,undefined]`를 반환한다.`undefined` 를 포함하는 객체는 다루기 어렵고 절대 권장하지 않는다. 

```tsx
function extent(nums: number[]){
  let min, max;
  for (const num of nums){
    if (!min){
      min = num
      max = num
    } else {
      min = Math.min(min,num)
      max = Math.max(max,num)
    }
  }
  return [min, max]
}
```

위의 코드의 해결법은 (!) null 아님 단언을 사용하던가 if 구문으로 체크할 수 있다.

```tsx
function extent(nums: number[]){
  let result: [number, number] | null = null;
  for (const num of nums){
    if (!result){
      result = [num,num]
    } else {
      result = [Math.min(num,result[0]),Math.max(num,result[1])]

    }
  }
  return result
}

// 1. (!) null 아님 단언 사용
const [min,max] = extent([0,1,2])!

// 2. if 구문으로 체크
const range = extent([0,1,2])!
if (range) {
	const [min, max] = range;
	const span = max - min;
}
```

- API 작성시에는 반환 타입을 큰 객체로 만들고 반환 타입 전체가 null 이거나 null이 아니게 만들어야 한다.
- 클래스를 만들 때는 필요한 모든 값이 준비되었을 때 생성하여 null이 존재하지 않도록 하는 것이 좋다.

> ➡️ strictNullChecks 설정을 하면 코드가 좀 더 복잡해지고, 어떻게 하나 하나 null 체크를 하지 싶었는데 이 챕터를 보고나니 어떤식으로 체크를 해야하는지 도움이 되었고, 프로젝트에도 적용해 볼 생각이다.
> 

### 아이템 32 유니온의 인터페이스보다는 인터페이스의 유니온을 사용하기

---

- 유니온의 인터페이스보다는 인터페이스의 유니온이 더 정확하고 타입스크립트가 이해하기 좋다.
    
    ```tsx
    // Bad
    interface Layer {
    	layout: FillLayout | LineLayout | PointLayout;
    	paint: FillPaint | LinePaint | PointPaint;
    }
    
    // Good
    interface FillLayer {
    	layout: FillLayout;
    	paint: FillPaint;
    }
    
    interface LineLayer {
    	layout: LineLayout;
    	paint: LinePaint;
    }
    
    interface PointLayer {
    	layout: PointLayout;
    	paint: PointPaint;
    }
    
    type = FillLayer | LineLayer | PointLayer;
    ```
    
- 여러 개의 선택적 필드가 동시에 값이 있거나 동시에 없는경우 여러 개의 속성을 하나의 객체로 모으는것이 더 나은 설계이다.
    
    ```tsx
    // Bad
    interface Person {
    	name: string;
    	// 다음은 둘 다 동시에 있거나 동시에 없다
    	placeBirth?: string;
    	dateBirth?: Date;
    }
    
    // Good
    interface Person {
    	name: string;
    	birth?: {
    		place: string;
    		date: Date;
    	}
    }
    ```
    

### 아이템 33 string 타입보다 더 구체적인 타입 사용하기

---

string은 any와 비슷한 문제를 갖고 있기 때문에 모든 문자열을 할당할 수 있는 string 타입보다는 문자열 리터럴 타입의 유니온을 사용하면 좋다. 

```tsx
// Bad
interface Album {
	title: string;
	releaseDate: string;
	recordingType: string;
}

// Good
type RecordingType = 'studio' | 'live';

interface Album {
	title: string;
	releaseDate: Date;
	recordingType: RecordingType;
}
```

### 아이템 34 부정확한 타입보다는 미완성 타입을 사용하기

---

타입을 구체적으로 작성할 수록 버그를 많이 잡고 타입스크립트가 제공하는 도구를 활용할 수 있게 됩니다. 

하지만 타입 선언의 정밀도를 높이는 일은 주의를 기울여야 한다. 실수가 발생하기 쉽고 잘못된 타입은 차라리 타입이 없는거 보다 못할 수 있기 때문이다. 
또한 타입 선언을 더 구체적이라고 하여도, 자동 완성을 방해한다면 타입스크립트 경험을 해치기 때문에 주의를 기울여야 한다. 

### 아이템 35 데이터가 아닌, API와 명세를 보고 타입 만들기

---

데이터에 드러나지 않는 예외적인 경우들이 문제가 될 수 있으므로 데이터보다는 명세로부터 코드를 생성하는 것이 좋다. 

### 아이템 36 해당 분야의 용어로 타입 이름 짓기

---

 가독성을 높이고, 추상화 수준을 올리기 위해서 해당 분야의 요어를 사용해야 한다. 

```tsx
interface animal {
	name: string;
}

interface animal {
	commonName: string;
}
```

`name`은 매우 일반적인 용어이다. 동물의 학명인지 일반적인 명칭인지 알 수가 없다. `commonName`으로 더 구체적인 용어로 대체했다.

### 아이템 37 공식 명칭에는 상표를 붙이기

---

타입스크립트의 구조적 타이핑 때문에 가끔 코드가 이상한 결과를 낼 수 있다. 

```tsx
interface Vector2D {
	x: number;
	y: number;
}

function calculateNorm(p: Vector2D) {
	return Math.sqrt(p.x * p.x + p.y * p.y);
}

calculateNorm({x: 3, y: 4}); // 정상
const vec3D ={x: 3, y: 4, z: 5}
calculateNorm(vec3D); // 정상!
```

`calculateNorm` 함수가 3차원 벡터를 허용하지 않게 하려면 상표(brand)를 붙이면 된다.

```tsx
interface Vector2D {
	_brand: '2d';
	x: number;
	y: number;
}

function vec2D(x: number, y: number): Vector2D {
	return {x, y, _brand: '2d'}
}

function calculateNorm(p: Vector2D) {
	return Math.sqrt(p.x * p.x + p.y * p.y);
}

calculateNorm(vec2D({x: 3, y: 4})); // 정상
const vec3D ={x: 3, y: 4, z: 5}
calculateNorm(vec3D); // Error..
```

만약 `vec3D`에 `_brand: '2d'`라고 추가하는 것과 같은 악의적인 사용을 막을 수는 없다. 하지만 단순한 실수를 방지하기에는 좋다.
