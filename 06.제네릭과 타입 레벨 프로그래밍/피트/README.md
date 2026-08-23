## TS스터디 23일차: 아이템 50

### 아이템 50: 제네릭을 타입 간 함수라고 생각하기

제네릭 타입은 타입의 영역에서 반복을 줄이는 역할을 수행한다.

#### 제네릭 타입을 인스턴스화한다?

함수를 호출하는 것과 달리, 제네릭 타입을 인스턴스화한다는 표현이 있다.

제네릭 타입의 빈칸인 타입 매개변수에 실제 타입을 넣어 새로운 구체적인 타입을 만드는 것이다.

클래스 생성의 맥락으로 이해하면 좋을 듯

```ts
type Box<T> = {
  value: T;
};

type StringBox = Box<string>;
```

`Box<T>`라는 제네릭 타입에 `string`을 넣어 `Box<string>`이라는 구체적인 타입을 만든다.

`string | number | symbol`의 내장 별칭은 `PropertyKey`

제네릭 타입을 선언할 때도 매개변수의 타입을 좁히는 방식이 권장된다.

```ts
type MyPick<T extends object, K extends keyof T> = {
  [P in K]: T[P];
};
```

타입 매개변수의 이름으로는 한 글자를 사용하는 것이 관례이다.

이름을 지을 때 가장 중요한 규칙은 이름의 길이와 그 범위가 일치해야 한다는 점이다.

제한된 스코프에서는 짧은 이름을 사용하고, 장기간 사용되는 전역 변수에는 긴 이름을 사용한다.

#### 실제로 제네릭 타입이 유니온 타입처럼 동작한다?

제네릭은 유니온 타입처럼 타입을 검사한다.

제네릭을 타입과 타입 간의 함수로 생각하는 것이 좋다.

```ts
type ArrayElement<T> = T extends Array<infer U> ? U : never;

type Result = ArrayElement<string[]>;
// string
```

일반 함수가 값을 입력받아 새로운 값을 반환하듯이, 제네릭 타입은 타입을 입력받아 새로운 타입을 만든다고 이해할 수 있다.

## TS스터디 24일차: 아이템 51~53

### 아이템 51: 불필요한 타입 매개변수 지양하기

제네릭의 황금 법칙이라는 게 있다고 한다.

타입 매개변수는 2번 등장해야 한다.

타입 매개변수가 하나의 함수 시그니처에서 한 번만 쓰이면 역할을 하지 못한다.

타입 매개변수가 한 번만 등장한다면 정말 필요한 것인지 신중히 생각해야 한다.

실제 모든 경우에 적용되는 법칙은 아니지만, 일종의 가이드라고 생각하자.

음... 어쨌든 중요한 건 타입 매개변수가 두 번 이상 등장해야 한다는 점이다.

또한 꼭 필요한 경우에만 함수나 클래스에 타입 매개변수를 추가하는 것도 중요하다.

#### 제네릭이 필요한 경우

입력 배열의 요소 타입과 반환값 타입이 같아야 하는 경우

```ts
// 배열에서 첫 번째 요소를 꺼내는 함수
// 두 위치에서 타입 매개변수가 등장한다.
// items: T[] (입력)
// T | undefined (출력)
function first<T>(items: T[]): T | undefined {
  return items[0];
}

// 결과
const a = first([1, 2, 3]);
// number | undefined

const b = first(['a', 'b']);
// string | undefined
```

#### 제네릭이 필요하지 않은 경우

```ts
function getLength<T extends { length: number }>(value: T): number {
  return value.length;
}
```

위 메서드에서 중요한 것은 타입이 아니라 `length` 속성이다.

따라서 제네릭을 사용하지 않아도 된다.

```ts
function getLength(value: { length: number }): number {
  return value.length;
}
```

> 멋있어 보인다는 이유로 제네릭을 추가하지 말고, 입력·출력·여러 매개변수 사이의 타입 관계를 표현할 필요가 있을 때만 사용하라.

### 아이템 52: 오버로딩 타입보다는 조건부 타입 사용하기

정확한 제네릭 메서드를 모델링하기 위해서는 오버로딩 타입 또는 조건부 타입을 사용할 수 있다.

무엇이 무조건 좋고 나쁜 것은 아니지만, 일반적으로는 조건부 타입을 사용하는 것이 좋다.

진짜 어려워서 모르겠다...

```ts
function double(value: string | number): string | number {
  return typeof value === 'string' ? value + value : value * 2;
}
```

위 함수는 구현은 멀쩡하지만 `string | number` 유니온 타입을 반환한다.

이를 해결하기 위해서 오버로딩 타입을 사용할 수 있다.

```ts
function double(value: string): string;
function double(value: number): number;

function double(value: string | number): string | number {
  return typeof value === 'string' ? value + value : value * 2;
}
```

그러나 오히려 타입이 좁혀져서 유니온 값을 전달하면 에러가 발생한다.

그래서 조건부 타입을 사용하면 해결할 수 있다.

```ts
function double<T extends string | number>(
  value: T,
): T extends string ? string : number;
```

유니온 타입 매개변수를 전달해도 각각에 조건부 검사가 수행되기 때문에 반환값도 정상적인 유니온 타입으로 나온다.

> 입력 타입과 반환 타입의 관계가 규칙적이고 유니온에도 적용되어야 한다면 조건부 타입을 고려하고, 서로 완전히 다른 호출 방식이라면 오버로딩이나 별도의 함수를 사용하는 편이 좋다.

### 아이템 53: 조건부 타입에 적용되는 유니온 영역을 제어하는 방법 숙지하기

아이템 52의 패턴을 적용할 수 없는 경우를 설명한다.

애초에 매개변수가 유니온 타입이라면?
