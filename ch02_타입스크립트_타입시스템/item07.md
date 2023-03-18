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
const cd: CD = {a: 'a', b: 'b'}. // 교집합 될만한게 없어보이는데 이게 된다고? ㅇㅇ
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

// Property 'birth' is missing in type '{ name: string; }'
// but required in type 'Lifespan'.
const ps: PersonSpan = {
  name: "Alan",
};
```

> ✍️ 인터섹션 타입의 값은 각 타입 내의 속성을 모두 포함하는 것이 일반적인 규칙이다. 뭔가 처음 보면 어리둥절하지만 의미를 알고보니 왜 인터섹션인지 알겠네요.

### 인터페이스 union(합집합) 생각해보기

이번에는 인터페이스 유니온(union, 합집합)에 대해 생각해봅시다. 생각해보면 Person만 포함하면 되고, Lifespan만 포함해도 됩니다. 즉, {name} 혹은 {birth}, {name, birth} 상관 없습니다.

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

하지만 이런경우를 생각해볼까요? 참고 : [https://velog.io/@soulee\_\_/TypeScript-Union-Type](https://velog.io/@soulee__/TypeScript-Union-Type)

이상하게도 name만 되고, age, skill은 안되는 것을 알 수 있습니다. 왜 그런걸까요? 타입스크립트 관점에서는 `introduce() 함수`를 호출하는 시점에 `Person`타입이 올지 `Developer`타입이 올지 알 수가 없기 때문에 어느 타입이 들어오든 간에 오류가 안 나는 방향으로 타입을 추론하게 된다고 합니다.

그래서 name만 되고 나머지에 대해서는 안되는 것이군요… 좀 어렵네요… ㅠ

```tsx
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

이번에는 이런 상황에 대해 생각해봅시다. keyof Union이 never인 것도 위 상황과 같아보이지 않나요? name만 가능합니다. 이 역시 타입스크립트 관점에서 어떤 타입이 올지 예측 불가능하기 때문에 오류가 안나는 방향으로 타입을 추론한 결과인듯합니다.

```tsx
type a = keyof (Person | Developer); // "name"
```

만약 확실히 Person이거나 Developer인 상황에서는 {name, age}도 될 것이고, {name, skill}도 될 것입니다. 오히려 이 경우에는 name만 있을 때 에러가 발생합니다.

```tsx
type PersonDeveloper = Person | Developer;

const person: PersonDeveloper = {
  name: "person a",
  age: 12,
};
```
