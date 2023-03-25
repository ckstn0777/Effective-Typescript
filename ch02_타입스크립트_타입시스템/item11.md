### 잉여 속성 체크란?

이미 몇번 다뤘던 내용입니다. ‘그 외 속성은 없는지’ 확인을 해서 있다면 에러가 발생합니다. 근데 생각해보면 구조적 타이핑 관점에서는 오류가 발생하지 않아야 합니다. 당연히 Room에 대한 속성은 가지고 있으니까요.

```tsx
interface Room {
  numDoors: number;
  ceilingHeightFt: number;
}
const r: Room = {
  numDoors: 1,
  ceilingHeightFt: 10,
  elephant: "present", // Object literal may only specify known properties, and 'elephant' does not exist in type 'Room'.(2322)
};
```

그래서 그런지 신기하게도 임시 변수를 도입해보면 obj 객체는 Room 타입에 할당이 가능합니다.

obj의 타입은 {numDoors: number; ceilingHeightFt: number; elephant: string;}으로 추론이됩니다. obj 타입은 Room 타입의 부분집합을 포함하므로, Room에 할당 가능하며 타입 체커도 통과합니다. (그러면 위 경우는 다른건가? 같은거 아닌가…?)

```tsx
interface Room {
  numDoors: number;
  ceilingHeightFt: number;
}
const obj = {
  numDoors: 1,
  ceilingHeightFt: 10,
  elephant: "present",
};
const r: Room = obj; // 정상
```

앞의 두 예시에 차이점을 보겠습니다. 첫번째 예시는 구조적 타입 시스템에서 발생할 수 있는 중요한 종류의 오류를 잡을 수 있도록 ‘잉여 속성 체크’라는 과정이 수행되었습니다. **잉여 속성 체크가 할당 가능 검사와는 별도의 과정이라는 것**을 알아야 타입스크립트의 타입 시스템 개념을 정확히 잡을 수 있습니다. (??)

### \***\*초과 프로퍼티 검사 (Excess Property Checks)\*\***

참고 : [https://inpa.tistory.com/entry/TS-📘-타입스크립트-객체-타입-체킹-원리-이해하기](https://inpa.tistory.com/entry/TS-%F0%9F%93%98-%ED%83%80%EC%9E%85%EC%8A%A4%ED%81%AC%EB%A6%BD%ED%8A%B8-%EA%B0%9D%EC%B2%B4-%ED%83%80%EC%9E%85-%EC%B2%B4%ED%82%B9-%EC%9B%90%EB%A6%AC-%EC%9D%B4%ED%95%B4%ED%95%98%EA%B8%B0)

“객체 타입 속성을 검사하는 것을 좀더 전문 용어로 표현하자면 **초과 프로퍼티 검사**라고 한다.”

“객체 리터럴을 변수에 직접 할당할 때나 인수로 전달할 때, *초과 프로퍼티 검사 (excess property checking)*를 받게 된다.”

책에 있는 내용만으로 이해하지 못해서 찾아봤습니다. 이제야 뭔소리인지 이해가 가더군요. 즉, 첫번째 경우는 저희가 직접 변수에 할당 하기 때문에 이때 초과 프로퍼티 검사가 일어난다는 것입니다.

```tsx
const r: Room = {
  numDoors: 1,
  ceilingHeightFt: 10,
  elephant: "present", // Object literal may only specify known properties, and 'elephant' does not exist in type 'Room'.(2322)
};
```

혹은 인수로 전달할 때도 마찬가지입니다.

```tsx
function createRoom(room: Room) {
  //...
}

createRoom({
  numDoors: 1,
  ceilingHeightFt: 10,
  elephant: "present",
});
```

그러나 객체를 다른 변수에 할당하게 되면, 변수 obj 는 초과 프로퍼티 검사를 받지 않기 때문에, 컴파일러는 에러를 주지 않는 것입니다.

```tsx
const obj = {
  numDoors: 1,
  ceilingHeightFt: 10,
  elephant: "present",
};
const r: Room = obj; // 정상
```

<aside>
✍️ 이제야 책에서 나온 “**잉여 속성 체크가 할당 가능 검사와는 별도의 과정이라는 것**을 알아야…” 이 말의 뜻을 알겠군요.

</aside>

### 타입 단언와 잉여 속성 체크

잉여 속성 체크는 타입 단언문을 사용할 때도 적용되지 않습니다. 이 예시가 단언문보다 선언문을 사용해야 하는 단적인 이유 중 하나입니다.

```tsx
const r = {
  numDoors: 1,
  ceilingHeightFt: 10,
  elephant: "present",
} as Room; // 정상
```

### 잉여 속성 체크를 원하지 않는 경우 - 인덱스 시그니처

특별한 경우에, 추가 프로퍼티가 있음을 확신한다면, 인덱스 시그니처를 사용해서 타입스크립트가 추가적인 속성을 예상하도록 할 수 있습니다.

```tsx
interface Room {
  numDoors: number;
  ceilingHeightFt: number;
  [otherOptions: string]: unknown;
}

const r: Room = {
  // 정상
  numDoors: 1,
  ceilingHeightFt: 10,
  elephant: "present",
};
```

### 약한(weak) 타입

선택적 속성만 가지는 타입을 약한 타입이라고 한다. 근데 약한 타입은 좀 특이한거 같습니다.

```tsx
interface LineChartOptions {
  logscale?: boolean;
  invertedYAxis?: boolean;
  areaChart?: boolean;
}

const opts = {}; // 가능
const opts2 = { logscale: true }; // 가능
const opts3 = { logScale: true }; // 이건 왜 불가능?
const opts4 = { logscale: true, logScale: true }; // 가능

const o: LineChartOptions = opts3;
// Type '{ logScale: boolean; }' has no properties in common with type 'LineChartOptions'.(2559)
```

3번째 경우는 왜 에러가 날까요? 객체를 다른 변수에 할당하게 되면 초과 프로퍼티 검사를 받지 않기 때문에 잉여 속성 체크가 안되는거 아니었나요? 그래서 opts4는 되잖아요. 음…

약한 타입에 경우는 타입스크립트가 값 타입과 선언 타입에 공통된 속성이 있는지 확인하는 별도의 체크를 수행한다고 합니다. **공통 속성 체크**는 잉여 속성 체크와 마찬가지로 오타를 잡는데 효과적이며 구조적으로 엄격하지 않습니다. 그러나 잉여 속성 체크와 다르게, 약한 타입과 관련된 할당문마다 수행됩니다. (아… 그래?)

### 마치면서

오늘은 잉여 속성 체크에 대해 알아봤습니다. 처음이랑 마지막에는 다소 혼란스러웠지만 원리를 이해하게 되면서 앓던 이가 빠진 느낌입니다. 오늘도 중요한 개념을 배워가서 좋네요.

약한 타입 공통 속성 체크는 아직도 혼란스럽네요.. 허허…
