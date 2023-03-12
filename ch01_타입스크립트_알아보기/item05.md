### 타입스크립트는 점진적이고 선택적이다 - any 타입

코드에 타입을 조금씩 추가할 수 있기 때문에 점진적이며, 언제든지 타입 체케를 해제할 수 있기 때문에 선택적이다. 이 기능들의 핵심은 바로 any 타입니다.

다음 예시를 볼까요? as any를 붙이지 않은 녀석은 오류라고 인식합니다. 이런 오류를 바로 해결할 수 있는 간단한 방법이 바로 as any 입니다.

```tsx
let age: number;
age = "12"; // Type 'string' is not assignable to type 'number'.(2322)
age = "12" as any;
```

이렇듯 타입 선언을 추가하는 게 귀찮아서 혹은 에러를 고치기 귀찮아서 any 타입이나 타입 단언문(as any)을 사용하고 싶은 유혹에 빠질 수 있습니다.

하지만 정말 특별한 경우가 아니라면 any를 사용하게 되면 타입스크립트의 수많은 장점을 살릴 수 없고, any를 사용했을 때 위험성을 알고 있어야 합니다.

### any 타입에는 타입 안정성이 없다

as any를 통해 string을 할당 받은 age는 더 이상 number가 아닙니다. 하지만 타입 체커는 이후에도 계속 number 타입으로 판단할 것이고 이는 혼란을 초래하게 됩니다.

```tsx
let age: number;
age = "12" as any;
age += 1; // "121"
```

### any는 함수 시그니처를 무시해버린다

birthDate는 Date 타입을 매개변수로 받기를 원했습니다. 하지만 any 타입을 사용하면 calculateAge의 시그니처를 무시하게 됩니다.

```tsx
function calculateAge(birthDate: Date): number {
  //...
}

let birthDate: any = "1990-01-19";
calculateAge(birthDate);
```

### any 타입에는 언어 서비스가 적용되지 않는다

타입스크립트가 좋은 점은 자동완성 기능과 적절한 도움말을 제공한다는 점입니다. 그러나 any 타입을 사용하면 아무런 도움을 받을 수 없습니다.

이름 변경 기능은 또 다른 언어 서비스입니다. 편집기에서 first를 선택하고, Rename Symbol을 클릭해서 firstName으로 바꿀 수 있습니다.

```tsx
interface Person {
  first: string;
  last: string;
}

const formatName = (p: Person) => `${p.first} ${p.last}`;
const formatNameAny = (p: any) => `${p.first} ${p.last}`;
```

바뀐 코드입니다. 하지만 any 타입으로 지정된 first는 바뀌지 않았습니다. 이처럼 언어 서비스를 제대로 누리기 위해서, 그리고 동료의 협업과 생산성을 위해서는 any 타입을 지양해야 합니다.

```tsx
interface Person {
  firstName: string;
  last: string;
}

const formatName = (p: Person) => `${p.firstName} ${p.last}`;
const formatNameAny = (p: any) => `${p.first} ${p.last}`;
```

### any 타입은 코드 리팩터링 때 버그를 감춘다

다음과 같은 코드가 있다고 가정해봅니다. 아이템(item)의 타입이 무엇인지 알기 어려우니 any 타입을 사용했습니다.

```tsx
interface ComponentProps {
  onSelectItem: (item: any) => void;
}

function renderSelector(props: ComponentProps) {
  const item = { id: 1, name: "itme 1" };
  props.onSelectItem(item);
}

let selectedId: number = 0;
function handleSelectItem(item: any) {
  selectedId = item.id;
}
renderSelector({ onSelectItem: handleSelectItem });
```

onSelectItem에 아이템 객체를 필요한 부분(id)만 전달하도록 개선해보겠습니다. 잘 수정했다고 생각하고 타입 체크도 통과했습니다. 하지만 handleSelectItem는 여전히 any 매개변수로 받습니다. 따라서 selectedId = item.id 도 문제가 없다고 나옵니다. 결국 나중에는 런타임 오류나 예상하지 못한 undefined가 나올 것입니다.

```tsx
interface ComponentProps {
  onSelectItem: (id: number) => void; // 개선
}

function renderSelector(props: ComponentProps) {
  const item = { id: 1, name: "itme 1" };
  props.onSelectItem(item.id); // 개선
}

let selectedId: number = 0;
function handleSelectItem(item: any) {
  selectedId = item.id;
}
renderSelector({ onSelectItem: handleSelectItem });
```

이처럼 any가 아니라 구체적인 타입을 사용했다면, 타입 체커가 오류를 발견했을 것입니다.

### any는 타입 설계를 감춰버린다

애플리케이션 상태 같은 객체를 정의하려면 꽤 복잡합니다. 수많은 속성의 타입을 일일이 작성해야 하는데 any 타입을 사용하면 간단히 끝나기 때문에 유혹에 빠지기 쉽습니다.

물론 이때도 any 타입을 사용하면 안됩니다. 상태 객체의 설계를 감춰버리기 때문에 불분명해져버립니다. 설계가 잘 되었는지, 어떻게 되어있는지 전혀 알 수 없게됩니다.

### any는 타입시스템의 신뢰도를 떨어뜨린다

타입 체커가 실수를 잡아주고 코드의 신뢰를 위해 타입스크립트를 도입했는데 런타임에 타입 오류를 발견하게 된다면 타입 체커를 신뢰할 수 없을 것입니다. any 타입을 쓰지 않으면 런타임에 발견될 오류를 미리 잡을 수 있고 신뢰도를 높일 수 있습니다.

### 마치면서

이번 시간에는 any 타입을 지양해야 하는 여러가지 이유에 대해 살펴봤습니다. 실제 개발을 하다보면 너무 많은 속성을 일일히 타입을 지정해줘야 하는 경우가 간혹 있기 때문에 any 타입으로 간단히 해결하고 싶은 유혹이 있습니다. 하지만 그렇다고 해도 any 타입을 사용하면 안되겠네요.
