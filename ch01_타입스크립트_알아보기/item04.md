### 구조적 타이핑(덕 타이핑)이란?

자바스크립트는 본질적으로 `덕 타이핑` 기반입니다. 여기서 덕 타이핑이란, 객체가 어떤 타입에 부합하는 변수와 메서드를 가질 경우 객체를 해당 타입에 속하는 것으로 간주하는 방식을 말합니다. “만약 어떤 새가 오리처럼 걷고, 헤엄치고, 꽥꽥거리는 소리를 낸다면 나는 그 새를 오리라고 부를 것이다” 라는 유래에서 덕 타이핑이란 용어가 쓰이게 되었습니다.

타입스크립트 또한 매개변수 값이 요구사항을 만족한다면 타입이 무엇인지 신경 쓰지 않는 동작을 그대로 모델링합니다. 예시를 살펴볼까요? 아래와 같은 코드가 있습니다.

```typescript
interface Vector2D {
  x: number;
  y: number;
}

function calculateLength(v: Vector2D) {
  return Math.sqrt(v.x * v.x + v.y * v.y);
}
```

그리고 나서 또 다른 타입인 NamedVector를 만들어서 calculateLength를 사용할 수 있을까요? 가능합니다. 왜냐면 NamedVector에서는 x, y라는 타입을 가지고 있기 때문입니다. 따라서 Vector2D로 선언이 되어있어도 사용이 가능합니다. 타입스크립트는 그정도는 이해할만큼 충분히 영리합니다.

```typescript
interface NamedVector {
  name: string;
  x: number;
  y: number;
}

const v: NamedVector = { x: 3, y: 4, name: "Zee" };
calculateLength(v);
```

### 구조적 타이핑(덕 타이핑) 때문에 주의해야할 사항

여기서 ‘구조적 타이핑’이라는 용어가 사용됩니다. 하지만, 이러한 구조적 타이핑 때문에 문제가 발생하기도 합니다. 아래 코드를 볼까요? 우리는 3D 벡터를 기반으로 계산하길 원했지만 calculateLength가 2D 벡터를 기반으로 연산하는 것을 오류로 판단하지 못했습니다.

```typescript
interface Vector2D {
  x: number;
  y: number;
}

interface Vector3D {
  x: number;
  y: number;
  z: number;
}

function calculateLength(v: Vector2D) {
  return Math.sqrt(v.x * v.x + v.y * v.y);
}

function normalize(v: Vector3D) {
  const length = calculateLength(v);
  return {
    x: v.x / length,
    y: v.y / length,
    z: v.z / length,
  };
}

normalize({ x: 3, y: 4, z: 5 });
```

calculateLength가 2D 벡터를 받도록 선언되어있음에도 불구하고 3D벡터를 받는데 문제가 없었던 이유는 무엇일까요? 당연히 구조적 타이핑 관점에서 Vector3D이지만 x와 y가 있어서 Vector2D와 호환되기 때문입니다.

> ✍️ 함수를 작성할 때, 호출에 사용되는 매개변수의 속성들이 매개변수의 타입에 선언된 속성만 가질 거라 생각하기 쉽다. 이러한 타입을 ‘봉인된’ 혹은 ‘정확한’ 타입이라고 불리며, 타입스크립트에서는 좋든 싫든 타입은 ‘**열려(open)**’ 있습니다.

또 다른 혼란스러운 예시를 볼까요? 그냥 자바스크립트 보듯이 보면 별로 문제는 없어보입니다. 하지만 타입스크립트에서는 에러를 발생하고 있습니다. 왜일까요?

```typescript
function calculateLengthL1(v: Vector3D) {
  let length = 0;
  for (const axis of Object.keys(v)) {
    const coord = v[axis]; // ?
    length += Math.abs(coord);
  }
  return length;
}
```

> ⚠️ Element implicitly has an 'any' type because expression of type 'string' can't be used to index type 'Vector3D'. No index signature with a parameter of type 'string' was found on type 'Vector3D'.(7053)

헷갈리기 쉬운데 타입스크립트에서 타입은 열려있다고 했습니다. 따라서 다음 코드도 실행 가능합니다. (!!)

```typescript
const vec3D = { x: 3, y: 4, z: 1, address: "123 Broadway" };
calculateLengthL1(vec3D);
```

구조적 타이핑은 클래스와 관련된 할당문에서도 당황스러운 결과를 보여줍니다. 우리는 첫번째 상황을 원했겠지만 두번째 상황도 가능합니다. 어떻게 d가 C 타입이 될 수 있었던 걸까요?

```typescript
class C {
  foo: string;
  constructor(foo: string) {
    this.foo = foo;
  }
}

const c = new C("instance of C");
const d: C = { foo: "object literal" }; // ?
```

일단 d는 foo 속성을 가질 수 있게 되었습니다. 그리고 호출이 되는 생성자(Object.prototype으로부터 비롯된)를 가집니다. 그래서 구조적으로는 필요한 속성과 생성자가 존재하기 때문에 문제가 없었던 것입니다. 만약 C의 생성자에 단순 할당이 아닌 연산 로직이 존재한다면 문제가 발생될 것입니다.

### 테스트 작성 시 구조적 타이핑이 유리하다

데이터베이스에 쿼리를 하고 결과를 처리하는 함수를 가정해보겠습니다.

```typescript
interface PostgresDB {
  //엄청 길고 복잡함
  //..
  runQuery: (sql: string) => any[];
}
interface Author {
  first: string;
  last: string;
}

function getAuthors(db: PostgresDB): Author[] {
  // authors 리턴 하는 로직
}
```

getAuthors를 테스트하기 위해서는 모킹한 PostgresDB를 생성해야 합니다. 하지만 테스트를 위해 실제 환경에 대한 정보까지 필요하지 않을 수 있습니다. 이럴때 구조적 타이핑 개념을 이용해 비슷한 인터페이스를 정의할 수 있습니다.

```typescript
interface DB {
  runQuery: (sql: string) => any[];
}

function getAuthors(db: DB): Author[] {
  // authors 리턴 하는 로직
}
```

구조적 타이핑 덕분에 PostgresDB의 인터페이스를 명확히 선언할 필요가 없습니다. 추상화(DB)를 함으로써, 로직과 테스트를 특정한 구현(PostgresDB)으로부터 분리한 것입니다.

### 마치면서

오늘은 중요한 개념인 ‘구조적 타이핑(덕 타이핑)’ 에 대해 알아보는 시간이었네요. 덕분에 구조적 타이핑에 대해 확실히 알게 되었고 타입스크립트에서는 이러한 구조적 타이핑으로 인해 혼란스러운 경우가 있을 수 있다는 것을 알게 되었네요.
