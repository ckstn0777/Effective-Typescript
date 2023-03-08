### 타입스크립트 설정에 대하여

아래 코드는 오류 없이 타입 체커를 통과할 수 있을까요? 통과 될 수 있을거 같기도 하고 아닐거 같기도 합니다. 사실 정답은 타입스크립트 설정에 따라 에러가 될 수 있고 아닐 수도 있습니다.

```typescript
function add(a, b) {
  return a + b;
}
add(10, null);
```

타입스크립트 컴파일러는 매우 많은 설정을 가지고 있고 현재는 거의 100개에 이른다고 하네요. 보통 tsconfig.json 설정 파일에서 설정을 관리하게 됩니다.

저는 지금 플레이그라운드에서 간단하게 실습하고 있는데 여기서 ‘noImplicitAny’라는 설정을 체크하게 되면 위 코드는 에러로 잡힙니다. error: ‘Parameter 'a', ‘b’ implicitly has an 'any' type.’

### 엄격한 타입스크립트 설정

타입스크립트를 제대로 사용하려면, `noImplicitAny`와 `strictNullChecks`를 이해해야 합니다.

noImplicitAny는 변수들이 미리 정의된 타입을 가져야 하는지 여부를 제어합니다. 위 코드에서 a, b는 암시적으로 any라는 타입을 가지게 됩니다. any 타입을 매개변수로 사용하게 되면 타입 체크는 무력해집니다. 있으나마나 해지는 것이지요. 그래서 되도록이면 noImplicitAny를 true로 설정하는 것이 좋습니다.

strictNullChecks는 null과 undefined가 모든 타입에서 허용되는지 확인하는 설정입니다. true가 되어있다면 아래 코드는 에러를 발생시킬 것입니다.

```typescript
const x: number = null; // Type 'null' is not assignable to type 'number'
```

만약 null을 허용하고 싶으면 명시적으로 null을 허용한다고 작성해야 합니다.

```typescript
const x: number | null = null;
```

이런 모든 체크를 설정하고 싶다면 `strict` 설정을 하면 됩니다. 타입스크립트에서는 strict 설정을 권장하며 이를 설정하면 대부분의 타입 오류를 잡아낼 수 있습니다.

### 마치면서

2장에서는 아주 기본적인 타입스크립트 설정에 대해 배울 수 있었습니다. 아주 많은 설정이 있지만 그 중 일부에 대해 알아봤네요. strict은 무조건 true로 해놓고 써야 하며 이를 true로 해서 쓰지 않는다면 타입스크립트를 쓰는 의미가 없다는 말을 들어보긴 했습니다. 그러한 내용을 설명하고 있네요.
