### 타입스크립트의 심벌은 타입 공간이나 값 공간 중의 한 곳에 존재한다

예를 들어볼까요? 처음 Cylinder은 타입으로 쓰였습니다. 하지만, 두번째 Cylinder는 값으로 쓰였습니다.

```tsx
interface Cylinder {
  radius: number;
  height: number;
}

const Cylinder = (radius: number, height: number) => ({ radius, height });
```

상황에 따라서는 Cylinder는 타입으로 쓰일 수 있고, 값으로 쓰일 수도 있습니다. 이런 점이 가끔 혼란을 야기합니다.

```tsx
const c: Cylinder = {
  // 타입으로 쓰임
  radius: 2,
  height: 3,
};

Cylinder(2, 3); // 함수로 쓰임

function calculateVolume(shape: unknown) {
  if (shape instanceof Cylinder) {
    // 여기서의 Cylinder는 타입이 아닌 값에 대해서 연산합니다.
    shape.radius;
    // 오류 : '{}' 형식에 'radius' 속성이 없습니다.
  }
}
```

> ✍️ 컴파일 과정에서 타입 정보는 제거되기 때문에, 심벌이 사라진다면 그것은 타입에 해당될 것입니다.

타입스크립트 코드에서 타입과 값은 번갈아 나올 수 있습니다. 타입 선언(:) 또는 단언문(as) 다음에 나오는 심벌은 타입인 반면, = 다음에 나오는 모든 것은 값입니다.

```tsx
interface Person {
  first: string;
  last: string;
}

const p: Person = { first: "J", last: "JA" };
```

### class와 enum은 상황에 따라 타입과 값 두 가지 모두 가능한 예약어이다

다음 예시를 볼까요? 여기서 `Cylinder`는 타입으로 쓰였습니다. 클래스가 타입으로 쓰일 때는 형태가 사용되는 반면, 값으로 쓰일 때는 생성자가 사용됩니다.

```tsx
class Cylinder {
  radius = 1;
  height = 1;
}

function calculateVolumn(shape: unknown) {
  if (shape instanceof Cylinder) {
    shape; // 정상, 타입은 Cylinder
    shape.radius; // 정상, 타입은 number
  }
}
```

### 타입에서 쓰일 때와 값에서 쓰일 때 다른 기능을 하는 연산자 - typeof

타입의 관점에서 typeof는 값을 읽어서 타입스크립트의 타입을 반환합니다. 값의 관점에서는 typeof는 자바스크립트 런타임의 typeof 연산자가 됩니다.

```tsx
interface Person {
  first: string;
  last: string;
}

const p: Person = { first: "J", last: "JA" };

type T1 = typeof p; // 타입은 Person
const v1 = typeof p; // 값은 "object"
```

혼란하다 혼란해… 책에서 뭘 말하고 싶은건지 잘 모르겠음… 이건 패스..

```tsx
class Cylinder {
  radius = 1;
  height = 1;
}
const v = typeof Cylinder; // 값이 "function"
type T = typeof Cylinder; // 타입이 typeof Cylinder

declare let fn: T;
const c = new fn(); // 타입이 Cylinder

type C = InstanceType<typeof Cylinder>; // 타입이 Cylinder
```

타입의 속성을 얻을 때는 `obj.field`가 아니라 반드시 `obj['field']`를 사용해야 합니다.

```tsx
const first: Person["first"] = p["first"]; // string 타입
```

### **두 공간 사이에서 다른 의미를 가지는 코드 패턴**

- 값에서 &와 | 는 ADN와 OR를 의미하는 비트 연산입니다. 타입에서는 인터섹션과 유니온을 의미합니다.
- const는 새 변수를 선언하지만, as const는 리터럴 또는 리터럴 표현식의 추론된 타입을 바꿉니다.
- extends는 서브클래스, 서브타입 또는 제너릭 타입의 한정자를 정의할 수 있습니다.
- 값으로 쓰이는 this는 자바스크립트의 this 키워드입니다. 타입으로 쓰이는 this는 일명 다형성 this라고 불리는 this의 타입스크립트 타입입니다. (??)
- in은 루프 또는 매핑된 타입에 등장합니다.

보통 타입스크립트에서 이런식으로 코드를 작성하면 되지 않을까 했는데 에러가 난적이 있을 것입니다. 여기서는 값의 관점에서 Person과 string이 해석되었기 때문입니다. 즉, Person이라는 변수명과 string이라는 이름을 가지는 두 개의 변수를 생성하려고 한 것입니다.

```tsx
function email({
  person: Person, // 오류: ~~바인딩 요소 'Person'에 암시적으로 'any' 형식이 있습니다.
  subject: string, // 'string' 식별자가 중복되었습니다.
  // 오류: ~~바인딩 요소 'Person'에 암시적으로 'any' 형식이 있습니다.
  subject: string, // 'string' 식별자가 중복되었습니다.
  // 오류: ~~바인딩 요소 'Person'에 암시적으로 'any' 형식이 있습니다.
});
```

문제를 해결하려면 아래처럼 타입과 값을 구분해야 합니다.

```tsx
function email({
  person,
  subject,
  body,
}: {
  person: Person;
  subject: string;
  body: string;
}) {
  //...
}
```

### 마치면서

이번 아이템에서도 중요한 내용을 다루고 있군요. 타입스크립트가 헷갈리는것 중 하나가 바로 타입과 값의 구분이 모호할 수 있다는 것일 거 같습니다. 하지만 이번 아이템을 이해하고 나면 그런 모호함은 좀 풀릴 것으로 생각합니다.
