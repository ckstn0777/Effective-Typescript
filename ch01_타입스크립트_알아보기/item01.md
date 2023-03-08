### Overview

- 타입스크립트는 사용 방식 면에서 다른 언어와 다른 독특한 언어라고 볼 수 있다. 파이썬 처럼 인터프리터로 동작하는 것도 아닌것이, 자바 처럼 저수준 언어로 컴파일 되는 것도 아니다. 또 다른 고수준 언어인 자바스크립트로 컴파일 되며, 실행 역시 타입스크립트가 아닌 자바스크립트로 이뤄지는 구조이다.
- 따라서 타입스크립트와 자바스크립트 관계는 필연적이며 밀접한 관계. 관계를 잘 이해하는 것이 중요하다.

<br>

### 타입스크립트는 자바스크립트의 상위 집합인가?

책에서는 “타입스크립트는 자바스크립트의 `상위 집합`”이다. 라며 처음 소개하고 있다.

- 당장 js 파일을 ts로 바꾼다고 해도 크게 달라지는 건 없을 것이다. 타입오류는 좀 발생
  할 수 있지만 그 자체로 큰 이슈는 없다.
- 이러한 특성은 자바스크립트에서 타입스크립트로 마이그레이션 하는데 엄청난 이점이 된다. 다른 언어에서는 상상도 못할 일.

이처럼 모든 자바스크립트는 타입스크립트가 될 수 있지만 그 반대는 성립하지 않는다. 이는 타입스크립트가 타입을 명시하는 추가적인 문법을 가지고 있기 때문이다. 당연하게도 아래 타입스크립트 코드는 일반적인 node로 실행하면 에러가 발생할 것이다. 이는 `: string` 이라는 타입이 추가되었기 때문에.

```typescript
function greet(who: string) {
  console.log("Hello", who);
}
```

### 타입 추론

타입스크립트 컴파일러는 타입스크립트 뿐만 아니라 일반 자바스크립트 프로그램에도 유용하다. 아래 예시는 분명 타입스크립트가 아닌 자바스크립트 처럼 보인다. 하지만 타입스크립트의 타입 체커는 문제점을 바로 찾아낸다.
이는 city변수가 문자열이라는 걸 알려주지 않아도 `타입 추론`을 통해 자동으로 알 수 있기 때문이다.

```typescript
let city = "new york city";
console.log(city.toUppercase()); // Property 'toUppercase' does not exist on type 'string'. Did you mean 'toUpperCase'?
```

### 타입과 사용자 의도

타입 시스템의 목표 중 하나는 런타임에 오류를 발생시킬 코드를 미리 찾아내는 것이다. 하지만 자바스크립트 런타임에서는 오류가 발생하지 않을 코드에서 타입스크립트는 에러를 발생시키기도 한다.

책에서는 이런 현상에 대해 타입스크립트가 오류 뿐만 아니라 의도와 다르게 동작할만한 코드를 오류로 판단한다고 한다.

```typescript
const states = [
  // 이미 타입추론으로 타입이 정의 되어있긴 함
  { name: "Alabama", capital: "Montogomery" },
  { name: "Alask", capital: "Juneau" },
  { name: "Alabama", capital: "Phoenix" },
];

for (const state of states) {
  console.log(state.capitol); // Property 'capitol' does not exist on type '{ name: string; capital: string; }'. Did you mean 'capital'?
}
```

하지만 타입스크립트가 자동으로 사용자 의도를 예측하는 것보다 직접 타입 명시를 통해 의도를 명확히 하는 것이 좋다고 한다. 만약 명시적으로 나의 의도를 타입으로 정의하지 않았다면 아래에서 에러를 발생시키지 않았을 것이다.

```typescript
interface State {
  name: string;
  capital: string;
}

const states: State[] = [
  { name: "Alabama", capital: "Montogomery" },
  { name: "Alask", capitol: "Juneau" }, // Type '{ name: string; capitol: string; }' is not assignable to type 'State'
  { name: "Alabama", capital: "Phoenix" },
];
```

### 타입스크립트는 자바스크립트의 런타임 동작을 모델링한다

엄격한 타입 규칙을 사용하는 언어라면 꿈도 못꿀 일을 타입스크립트는 오류 없이 가능하다고 판단할 수 있습니다. 아래 예시처럼 말이죠. 이는 타입스크립트가 자바스크립트의 런타임 동작을 베이스로 동작하기 때문입니다.

```typescript
const x = 2 + "3"; // ?
const y = "2" + 3; // ?
```

하지만 정상 동작하는 코드에 오류를 표시하기도 합니다. 이런 경우는 아무래도 명확히 이상한 코드라고 인식해서 오류라고 표시하는 듯 합니다.

```typescript
const a = null + 7; // The value 'null' cannot be used here.
const b = [] + 7; // Operator '+' cannot be applied to types 'never[]' and 'number'
```

그리고 또 한가지 신기한 점. 타입 체크는 통과했는데 런타임에 오류가 발생할 수 있습니다. 생각해보면 당연하겠네요. 타입스크립트라고 해서 names의 length가 얼마인지 판단해서 아래 코드가 오류인지 판단하진 않겠죠. 직접 자바스크립트로 실행해봐야 알 수 있는 내용입니다.

```typescript
const names = ["Allice", "Bob"];
console.log(names[2].toUpperCase());

// Uncaught TypeError: Cannot read properties of undefined (reading 'toUpperCase')
```

<br>

### 마치면서

어찌보면 당연하다고 생각하고 넘어갔을 내용을 다시 한번 책을 통해 정리하게 되니까 알송달송하기도 하고 쉬운거 같으면서 어려운거 같은 느낌입니다. 그래도 책 자체는 타입스크립트 개발자라면 도움이 될 만한 내용이네요.
