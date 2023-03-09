### Overview

바로 들어가기에 앞서 타입스크립트 컴파일러는 두 가지 역할을 수행한다는 사실을 알아야 합니다.

- 최신 타입스크립트/자바스크립트를 브라우저에서 동작할 수 있도록 구버전의 자바스크립트로 트랜스파일 하는 역할.
- 코드의 타입 오류를 체크하는 역할

놀라운 점은 이 두가지가 서로 완벽히 독립적이라는 것입니다. 무슨말일까요? 타입스크립트가 자바스크립트로 변환될 때 코드 내의 타입은 아무런 영향을 주지 않습니다. 또한 그 자바스크립트가 실행 될 때는 더더욱 영향을 미치지 않겠죠. 이런게 어떻게 가능할까요?

### 타입 오류가 있는 코드도 컴파일이 가능하다

컴파일은 타입 체크와 독립적으로 동작하기 때문에, 타입 오류가 있는 코드도 컴파일이 가능합니다. 실제로 해보니 타입 에러는 발생시키지만 결과물 또한 제대로 나옵니다. 좀 황당하긴 합니다.

```typescript
let x = "hello";
x = 1234;
```

타입스크립트 오류는 C나 자바 같은 언어들의 경고와 비슷하다고 보면 됩니다. 문제가 될 만한 부분을 알려주긴 하지만, 그렇다고 빌드는 멈추지 않습니다. 만약 오류가 있을 때 컴파일하지 않으려면 tsconfig.json에 noEmitOnError를 설정할 수 있습니다.

### 런타임에는 타입 체크가 불가능하다

다음 코드를 한번 볼까요? instanceof는 런타임에 실행될 것이지만 Rectangle 자체는 타입입니다. 그렇기 때문에 에러를 보여줍니다. Rectangle은 타입이고, 여기서는 값으로 쓰였기 때문에 에러라고 하네요. 무슨말일까요?

```typescript
interface Square {
  width: number;
}
interface Rectangle extends Square {
  height: number;
}
type Shape = Square | Rectangle;

function calculateArea(shape: Shape) {
  if (shape instanceof Rectangle) {
    // error TS2693: 'Rectangle' only refers to a type, but is being used as a value here.
    return shape.width * shape.height; // error TS2339: Property 'height' does not exist on type 'Shape'.
  }
  return shape.width * shape.width;
}
```

런타임 시점에는 타입은 사라지게 됩니다. 따라서 Rectangle는 아무런 역할을 할 수 없습니다. 타입스크립트의 타입은 ‘제거 가능(erasable)’ 하다는 것을 명시해야 합니다.

```typescript
function calculateArea(shape) {
  if (shape instanceof Rectangle) {
    // Rectangle가 뭔지 모르게 됨
    return shape.width * shape.height;
  }
  return shape.width * shape.width;
}
```

앞의 코드를 명확히 하려면 런타임에 타입 정보를 유지하는 방법이 필요합니다. 책에서는 3가지 방법을 설명하고 있습니다.

하나의 방법은 height 속성 자체를 체크해 보는 것입니다. 속성 체크를 통해 런타임에 접근 가능해지고, 타입 체커 역시 shape의 타입을 Rectangle로 알아서 판단할 수 있기 때문에 오류가 사라집니다.

```typescript
function calculateArea(shape: Shape) {
  if ("height" in shape) {
    return shape.width * shape.height;
  }
  return shape.width * shape.width;
}
```

또다른 방법으로는 런타임에 접근 가능한 타입 정보를 명시적으로 저장하는 ‘태그’ 기법이 있습니다. 여기서 Shape 타입은 ‘태그된 유니온(tagged union)의 한 예입니다. 이 기법은 런타임에 타입 정보를 손쉽게 유지할 수 있기 때문에 흔하게 사용됩니다.

```typescript
interface Square {
  kind: "square";
  width: number;
}
interface Rectangle {
  kind: "rectangle";
  width: number;
  height: number;
}
type Shape = Square | Rectangle;

function calculateArea(shape: Shape) {
  if (shape.kind == "rectangle") {
    shape; // (parameter) shape: Rectangle
    return shape.width * shape.height;
  } else {
    shape; // (parameter) shape: Square
    return shape.width * shape.width;
  }
}
```

마지막 방법으로 타입(런타임 접근 불가)과 값(런타임 접근 가능)을 둘 다 사용하는 기법도 있습니다. 바로 타입을 클래스로 만들면 됩니다. Square와 Rectangle을 클래스로 만들면 오류를 해결할 수 있습니다. 아래와 같이 클래스로 만들면 타입도 되기 때문에 타입과 값으로 모두 사용이 가능해집니다.

```typescript
class Square {
  constructor(public width: number) {}
}

class Rectangle extends Square {
  constructor(public width: number, public height: number) {
    super(width);
  }
}

type Shape = Square | Rectangle;

function calculateArea(shape: Shape) {
  if (shape instanceof Rectangle) {
    shape; // (parameter) shape: Rectangle
    return shape.width * shape.height;
  } else {
    shape; // (parameter) shape: Square
    return shape.width * shape.width;
  }
}
```

### 타입 연산은 런타임에 영향을 주지 않는다

아래와 같은 코드가 있다고 가정해봅시다. 타입체커는 통과하지만 잘못된 방법을 사용했습니다. 왜그럴까요?

```typescript
function asNumber(val: number | string): number {
  return val as number; // as number는 타입 단언문입니다.
}
```

바로 변환된 자바스크립트를 보면 알 수 있습니다. 타입이 다 날아가버렸습니다. 결국 의도와는 다르게 무엇도 아닌 함수가 되버렸네요.

```typescript
function asNumber(val) {
  return val;
}
```

이를 의도에 맞게 고치자면 다음과 같습니다. 변한된 자바스크립트도 동일한 코드임을 알 수 있을 것입니다.

```typescript
function asNumber(val: number | string): number {
  return typeof val === "string" ? Number(val) : val;
}
```

### 런타임 타입은 선언된 타입과 다를 수 있다

아래 코드를 봅시다. 마지막의 default를 통과해서 console이 실행되는 경우가 있을까요? 단순히 보기에는 없을거 같아보입니다. 하지만 변환된 자바스크립트 코드를 보면 : boolean은 런타임에 제거됩니다. 이제 이런 사실은 쉽게 알 수 있을 것입니다.

```typescript
function setLightSwitch(value: boolean) {
  switch (value) {
    case true:
      turnLightOn();
      break;
    case false:
      trunLightOff();
      break;
    default:
      console.log("실행될 수 있을까요?");
  }
}
```

하지만 생각치도 못하게 순수 타입스크립트에서도 마지막 코드를 실행하는 방법이 존재합니다. `/light`를 요청하면 그 결과로 LightApiResponse이 반환될 것을 예상하고 있지만 실제로는 그렇지 않을 수 있습니다. 만약 lightSwitchValue가 사실 문자열이었다면 그대로 런타임에는 setLigthSwitch가 실행되면서 default까지 실행될 것입니다.

```typescript
interface LightApiResponse {
  lightSwitchValue: boolean;
}
async function setLight() {
  const response = await fetch("/light");
  const result: LightApiResponse = await response.json();
  setLigthSwitch(result.lightSwitchValue);
}
```

> 타입스크립트에서는 런타임 타입과 선언된 타입이 맞지 않을 수 있음에 명심하고 혼란스러운 상황을 가능한 피해야 합니다.

### 타입스크립트 타입으로는 함수를 오버로드 할 수 없다

동일 이름에 매개변수만 다른 여러 버전의 함수를 함수 오버로딩이라고 합니다. 아 저도 헷갈려서 자바스크립트에서 함수 오버로딩이 가능한가 싶어서 찾아보니 가능하진 않고 비슷하게는 쓸 수 있네요.

```typescript
function CatStrings(p1, p2, p3) {
  var s = p1;
  if (typeof p2 !== "undefined") {
    s += p2;
  }
  if (typeof p3 !== "undefined") {
    s += p3;
  }
  return s;
}

CatStrings("one"); // result = one
CatStrings("one", 2); // result = one2
CatStrings("one", 2, true); // result = one2true
```

아무튼 타입스크립트에서도 함수 오버로딩은 불가능합니다.

```typescript
function add(a: number, b: number) {
  return a + b;
}

function add(a: string, b: string) {
  // Duplicate function implementation.(2393)
  return a + b;
}
```

그렇다고 아예 지원을 하지 않는 것은 아닙니다. 다만 제한이 있습니다. 바로 온전한 구현체는 하나라는 것입니다. 매개변수 타입에 따라 선택되는 add 함수가 다른것을 볼 수 있습니다.

```typescript
function add(a: number, b: number): number;
function add(a: string, b: string): string;
function add(a, b) {
  return a + b;
}

const num = add(1, 2); // function add(a: number, b: number): number (+1 overload)
const str = add("1", "2"); // function add(a: string, b: string): string (+1 overload)
```

### 타입스크립트 타입은 런타임 성능에 영향을 주지 않는다

타입과 타입 연산자는 자바스크립트 변환 시점에 제거되기 때문에, 런타임의 성능에 아무런 영향을 주지 않습니다. 대신 ‘빌드타임’ 오버헤드가 있습니다. 오버헤드가 커지면, 빌드 도구에서 ‘트랜스파일만(transpile only)’을 설정하여 타입 체크를 건너뛸 수 있습니다.

### 마치면서

이번 아이템은 꽤나 중요하고 배울 점이 많다고 생각됩니다. 특히 타입스크립트를 처음 시작하는 분들께 많은 도움이 될 내용인거 같습니다. 물론 저도 여러모로 많은 도움이 되었네요. 타입스크립트라고 무조건 만능은 아니라는 점. 필요에 따라 자바스크립트 런타임 환경을 중요하게 볼 줄 알아야 한다는 점이 그렇겠네요.
