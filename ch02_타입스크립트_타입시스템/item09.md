### 타입 선언과 타입 단언

타입스크립트에서 변수에 값을 할당하고 타입을 부여하는 방법은 두 가지가 있다. 타입 선언과 타입 단언. 확인해보면 둘 다 Person 타입을 가지고 있다.

```tsx
interface Person {
  name: string;
}

const alice: Person = { name: "Alice" }; // 타입 선언
const bob = { name: "Bob" } as Person; // 타입 단언
```

하지만 이 둘은 완전히 다르다. 타입 선언은 그 값이 선언된 타입임을 명시하도록 하는 반면, 타입 단언은 타입스크립트가 추론한 타입이 있더라도 무조건 Person 타입으로 간주하게 한다.

따라서 타입 단언보다 타입 선언을 사용하는 것이 좋다. 아래 코드를 보자.

```tsx
const alice: Person = {}; // Property 'name' is missing in type '{}' but required in type 'Person'.(2741)
const bob = {} as Person;
```

- 타입 선언은 할당되는 값이 해당 인터페이스에 만족하는 검사한다.
- 타입 단언은 강제로 타입을 지정했으니 타입 체커에게 오류를 무시하라고 하는 것과 같다.

타입 선언과 단언의 차이는 속성을 추가할 때도 마찬가지이다. 타입 선언문에서는 잉여 속성 체크가 동작했지만, 단언문에서는 적용되지 않는다.

```tsx
const alice: Person = { name: "Alice", age: 12 }; // Type '{ name: string; age: number; }' is not assignable to type 'Person'.
// Object literal may only specify known properties, and 'age' does not exist in type 'Person'.(2322)

const bob = { name: "Bob", age: 12 } as Person; // 이것도 에러 안남. 원래는 에러 남
```

근데 궁금한게 왜 아래 경우는 에러가 남? 이건 좀 선넘었나? 흠… {} as Person도 에러가 안나는데…

```tsx
const bob = { age: "Bob" } as Person; // 흠... 근데 이건 에러나더라
```

### 화살표 함수에서의 타입 선언과 타입 단언

화살표 함수의 타입 선언은 추론된 타입이 모호할 때가 있습니다. 우리는 people이 Person[]이길 원합니다. 현재는 `{ name: string; } []` 이라고 나옵니다.

```tsx
const people = ["alice", "bob", "jan"].map((name) => ({ name }));
```

이럴때 그냥 타입 단언을 쓰면 문제가 해결되는 것 처럼 보입니다.

```tsx
const people = ["alice", "bob", "jan"].map((name) => ({ name } as Person)); // Person[]
```

그러나 타입 단언을 사용하면 앞선 예제처럼 문제가 발생하게 됩니다.

```tsx
const people = ["alice", "bob", "jan"].map((name) => ({} as Person)); // ??
```

그렇다면 바람직한 방법은 무엇일까요? 타입 선언을 사용해볼까요? 괜찮아보이지만 좀 번잡해보입니다. 개선해봅시다.

```tsx
const people = ["alice", "bob", "jan"].map((name) => {
  const person: Person = { name };
  return person;
});
```

변수 대신 화살표 함수의 반환 타입을 선언했습니다. 아하… map 함수의 반환 값이 ({ name })이니까 이 녀석의 반환 타입을 Person으로 지정할 수 있군요. 하지만 (name): Person은 반환타입만 명시하고, name의 타입을 명시하지 않았습니다. (근데 자동 추론하는거 같은데… string으로)

```tsx
const people = ["alice", "bob", "jan"].map((name): Person => ({ name }));
```

다음 코드는 최종적으로 원하는 타입을 직접 명시하고, 타입스크립트가 할당문의 유효성검사를 하게됩니다. (근데 위에랑 뭔 차이지?)

```tsx
const people: Person[] = ["alice", "bob", "jan"].map(
  (name): Person => ({ name })
);
```

### 타입 단언이 꼭 필요한 경우

타입 단언은 타입 체커가 추론한 타입보다 여러분이 판단하는 타입이 더 정확할 때 의미가 있습니다. 예를 들어, DOM 앨리먼트에 대해서는 타입스크립트보다 저희가 더 정확하게 알고 있을 겁니다.

```tsx
document.querySelector("#myButton")!.addEventListener("click", (e) => {
  e.currentTarget; // EventTarget | null

  const button = e.currentTarget as HTMLButtonElement;
  button.textContent = "button";
});
```

### ! 단언문

위에서 저는 에러가 나서 ! 를 사용했습니다. 여기서 ! 는 단언문처럼 생각해야 합니다. 타입 체커는 알지 못하지만 그 값이 null이 아니라고 확신할 수 있을 때 사용해야 합니다. 그렇지 않다면 null인 경우를 체ㅔ크하는 조건문을 사용해야 합니다.

```tsx
document.querySelector("#myButton")!;
```

### 타입 단언문으로 임의의 타입 간 변환은 할 수 없다

단, A가 B의 부분 집합인 경우 타입 단언문을 사용해 변환할 수 있다. 아 위에서 궁금했던 것도 그런 이유에서구나.

```tsx
const bob = {} as Person;
const bob = { age: "Bob" } as Person; // 흠... 근데 이건 에러나더라
```

{} 는 Person의 부분집합인가? 그렇다. 근데 확실한건 { age: ‘Bob’ } 은 Person의 부분집합은 아니네.

음… 근데 { name: ‘Bob’, age: 12 }는? 이것도 아닌거 같은데…

아무튼 굳이 저 오류를 해결하고 싶으면 unknown을 사용하면 된다. unknown은 모든 타입이 될 수 있다. 하지만 unknown을 사용하는 이상 무언가 위험한 동작을 하고 있다는 것을 알아야 합니다.

```tsx
const bob = { age: "Bob" } as unknown as Person;
```
