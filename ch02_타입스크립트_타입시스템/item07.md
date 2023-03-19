### 타입을 값들의 집합이라고 생각하기

‘**할당 가능한 값들의 집합**’을 ‘타입’ 혹은 ‘타입의 범위’ 라고 생각할 수 있습니다. 예를 들면 다음과 같습니다.

- 모든 숫자값의 집합(ex. 24, 34.2)는 number 타입에 해당됨.
- 가장 작은 집합은 공집합이며 타입스크립트에서는 never 타입에 해당됨.
- 그 다음으로 작은 집합은 한 가지 값만 포함하는 타입으로 ‘유닛 타입’ 혹은 ‘리터럴 타입’
- 두 개 혹은 세 개로 묶으려면 유니온(union) 타입을 사용

다음은 유닛 타입 예시입니다. 유닛 타입은 한 가지 값만을 포함합니다.

```tsx
type A = "A";
type B = "B";
type Twelve = 12;
```

다음은 유니온(union) 타입 예시입니다. 유니온 타입은 값 집합들의 합집합으로 보면 됩니다.

```tsx
type AB = "A" | "B";
type AB12 = "A" | "B" | 12;
```

타입스크립트 오류에서 ‘할당 가능한’ 이라는 문구를 자주 볼 수 있습니다. 집합의 관점에서 보자면, ‘~의 원소’ 또는 ‘~의 부분 집합’을 의미합니다. 아래에서 ‘C’는 ‘AB’의 부분 집합이 아니므로 오류입니다.

```tsx
type AB = "A" | "B";

const a: AB = "A";
const c: AB = "C"; // Type '"C"' is not assignable to type 'AB'.(2322)
```

> ✍️ 집합의 관점에서 보면 타입 체커가 하는 역할은 **하나의 집합이 다른 집합의 부분 집합인지 검사하는 것**이라고 볼 수 있습니다.

타입을 값들의 집합이라고 생각해보면 통과되는 이유를 쉽게 알 수 있습니다.

```tsx
type AB = "A" | "B";
type AB12 = "A" | "B" | 12;

const ab: AB = Math.random() < 0.5 ? "A" : "B";
const ab12: AB12 = ab; // {'A', 'B'} 는 {'A', 'B', 12}의 부분 집합이기 때문에 통과

type CD = "C" & "D"; // 교집합 될 만한게 없으니까 never
```

단순한 값의 유니온, 인터섹션은 의미가 쉽지만 값이 아닌 경우라면 그 의미가 다시 생각해봐야 될 거 같습니다.

```tsx
type CD = {a: 'a'} & {b: 'b'}
const cd: CD = {a: 'a', b: 'b'}. // 교집합 될만한게 없어보이는데 이게 된다고?
```

### 인터페이스 intersection(교집합) 생각해보기

& 연산자는 두 타입의 인터섹션(intersection, 교집합)을 계산합니다. 언뜻보면 PersonSpan 타입은 Person과 Lifespan의 공통 속성이 없기 때문에 교집합이 없어서 공집합(never)로 예상하기 쉽습니다.

그러나 타입 연산자는 **인터페이스의 속성이 아닌, 값의 집합에 적용**됩니다. (→ 이 의미가 중요하네)

```tsx
interface Person {
  name: string;
}

interface Lifespan {
  birth: Date;
  death?: Date;
}

type PersonSpan = Person & Lifespan;
/* 
{
	name: string;
	birth: Date;
	death?: Date;
  ... 그 외 가능 ...
} 
*/
```

그니까 생각해보면 {name, birth} 를 가지고 있는 값은 Person도 속하고, Lifespan도 속하니까 교집합이 성립한다는 거네요. 만약 {name} 만 있으면 이건 Person은 성립하는데 Lifespan은 성립하지 않으니까 안되는거고요.

```tsx
const ps: PersonSpan = {
  name: "Alan",
  birth: new Date("1912/06/23"),
};

// Property 'birth' is missing in type '{ name: string; }' but required in type 'Lifespan'.
const ps: PersonSpan = {
  name: "Alan",
};
```

> ✍️ 인터섹션 타입의 값은 각 타입 내의 속성을 모두 포함하는 것이 일반적인 규칙이다. 뭔가 처음 보면 어리둥절하지만 의미를 알고보니 왜 인터섹션인지 알겠네요.

### 인터페이스 union(합집합) 생각해보기

이번에는 인터페이스 유니온(union, 합집합)에 대해 생각해봅시다. 생각해보면 유니온은 Person만 포함하면 되고, Lifespan만 포함해도 됩니다. 즉, {name} 혹은 {birth}, {name, birth} 상관 없습니다.

```tsx
interface Person {
  name: string;
}

interface Lifespan {
  birth: Date;
  death?: Date;
}

type PersonSpan = Person | Lifespan;

const ps: PersonSpan = {
  birth: new Date(),
};
```

하지만 아래 경우를 생각해볼까요? 이번에는 이상하게도 name만 되고, age, skill은 안되는 것을 알 수 있습니다. 왜 그런걸까요? 🤔

타입스크립트 관점에서는 `introduce() 함수`를 호출하는 시점에 `Person`타입이 올지 `Developer`타입이 올지 알 수가 없기 때문에 어느 타입이 들어오든 간에 오류가 안 나는 방향으로 타입을 추론하게 된다고 합니다. 그래서 name만 되고 나머지에 대해서는 안되는 것이군요… 좀 어렵네요… ㅠ

```tsx
// 참고 : https://velog.io/@soulee__/TypeScript-Union-Type

interface Person {
  name: string;
  age: number;
}
interface Developer {
  name: string;
  skill: string;
}
function introduce(someone: Person | Developer) {
  someone.name; // O 정상 동작
  someone.age; // X 타입 오류
  someone.skill; // X 타입 오류
}
```

만약 확실히 Person이거나 Developer인 상황에서는 {name, age}도 될 것이고, {name, skill}도 될 것입니다. 오히려 이 경우에는 name만 있을 때 에러가 발생합니다.

```tsx
type PersonDeveloper = Person | Developer;

const person: PersonDeveloper = {
  name: "person a",
  age: 12,
};
```

### keyof - union, intersection 알아보기

이번에는 이런 상황에 대해 생각해봅시다. 여기서부터 다소 헷갈리긴 합니다…

```tsx
type a = keyof (Person | Developer); // "name"
type b = keyof (Person & Developer); // "name" | "age" | "skill"
```

- 첫번째 union 했을 때 결과가 “name”인 것은 왜 그럴까요? 이것도 위에 경우를 생각해보면 어느정도 이해가 되는 거 같습니다. Person인지 Developer인지 확실하지 않은 상황에서 타입추론은 오류가 나지 않는 방향으로 추론하게 되므로 “name”만 가능해집니다.
- 두번째 intersection의 경우는 생각해보면 name, age, skill은 필수여야 합니다. 근데, keyof Developer 했을 때 결과도 “name” | “skill” 이렇게 나오는거 보니까 keyof (Person & Developer)이 저런식으로 나오는 것도 이해가 되네…

정리해보면 다음과 같습니다. 오웅…. 헷갈려…

```tsx
keyof (A&B) = (keyof A) | (keyof B)
keyof (A|B) = (keyof A) & (keyof B)
```

### 집합 관점에서 extends 키워드

보통 일반적으로 PersonSpan을 선언하는 방법은 extends 키워드를 쓰는 것입니다. 타입의 관점에서 extends는 ‘~의 부분 집합’ 이라는 의미로 받아들일 수 있습니다.

```tsx
interface Person {
  name: string;
}

interface PersonSpan extends Person {
  birth: Date;
  death?: Date;
}
```

이를 서브 타입 관점으로 생각해볼 수도 있습니다. 서브 타입이란 어떤 집합이 다른 집합의 부분집합이란 의미입니다.

```tsx
interface Vector1D {
  x: number;
}
interface Vector2D extends Vector1D {
  y: number;
}
interface Vector3D extends Vector2D {
  z: number;
}
```

### 제너릭 관점에서 extends 키워드

타입스크립트에서 extends 키워드는 2가지 관점에서 생각해볼 수 있습니다. 한가지는 위에서 말했다싶이 확장의 개념이고, 또 다른 한가지는 제너릭 타입에서 한정자로 쓰입니다.

```tsx
function getKey<K extends string>(val: any, key: K) {
  //...
}
```

여기서 string을 상속의 관점에서 생각하면 이해하기 어렵습니다. string의 부분 집합 범위를 가지는 한정자 느낌으로 생각해야 합니다. 따라서 key는 string인 경우에만 매개변수로 받을 수 있습니다.

```tsx
getKey({}, "x");
getKey({}, Math.random() < 0.5 ? "a" : "b");
getKey({}, document.title);
getKey({}, 12); // Argument of type 'number' is not assignable to parameter of type 'string'.(2345)
```

### 마치면서

이번 아이템에 대해서는 다소 어렵고 헷갈리는 부분이 많이 있었습니다. 특히 union과 intersection에 대해서는 단순한 경우라면 문제가 없는데 interface 끼리의 union과 intersection은 인터페이스 속성끼리가 아닌 값의 집합에 적용된다는 것을 처음 알게 되었습니다.

그리고 keyof (A & B), keyof (A | B)와 같이 값이 아닌 타입끼리의 union, intersection은 결과가 또 다르다는 사실을 알고 혼란스러웠습니다… ㅠㅠ

extends도 마찬가지입니다. 처음 타입스크립트를 공부할 때 extends가 2가지 의미가 있다는 사실을 알고 많이 혼란스러웠습니다. 이런 부분도 타입스크립트를 처음 공부하는 분들 입장에서 좀 어렵게 느껴질 수 있을 거 같다는 생각이듭니다.
