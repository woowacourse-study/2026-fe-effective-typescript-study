## 아이템 50

제네릭을 사용하면 유연해지지만, 그에 반해 복잡해진다.

결국 제네릭을 **함수**라고 생각하라는 말인 것 같다.

타입을 입력받아 새로운 타입을 반환하는 함수라고 생각하면 좀 더 쉽게 이해할 수 있다.

```ts
type MyPartial<T> = {
  [K in keyof T]?: T[K];
};

type Result = MyPartial<Person>;
```

`T`는 매개변수, `MyPartial<Person>` 는 함수 호출, 만들어진 타입은 반환값이라 생각하면 될 듯

```ts
type MyPick<T extends object, K extends keyof T> = {
  [P in K]: T[P];
};
```

쉽게 생각하면 T는 객체, K는 객체의 키이다.

-> 매개변수의 타입을 좁혀서 사용하라는 얘기인 것 같다.

-> `extends`를 사용해 제네릭 타입 함수가 받을 수 있는 입력 타입의 범위를 제한한다. 이를 통해 잘못된 타입 사용을 제네릭 내부가 아니라 호출 지점에서 발견할 수 있다.

제네릭이 짧고 단순하다면 `T`, `K`를 사용해도 되지만, 복잡해지면 의미 있는 이름을 사용해야 한다.

`A | B` 같은 유니온 타입을 입력했을 때의 동작도 고려해야 한다.

#### 의문점

제네릭 함수가 호출되는 시점에 값으로부터 타입 매개변수를 추론할 수 있다.

-> `function pick(): Pick<T, K>`으로 반환 타입을 지정해놨는데 당연한 것 아닌가?

-> 타입 추론은 반환 타입을 지정했기 때문에 일어나는 게 아니다. 함수 인자의 타입으로부터 `T`, `K`를 먼저 추론한 다음, 그 타입들을 반환 타입에 대입하는 것이다.

## 아이템 51

제네릭 함수를 작성하면 유연한 함수를 작성할 수 있지만, 타입 매개변수를 과도하게 사용하거나 불필요한 제한을 걸면 오히려 역효과를 일으킨다

-> 타입 추론이 어려워지고, 함수 사용이 복잡해진다.

### 타입 매개변수는 두 번 등장해야 한다

```ts
function identity<T>(arg: T): T {
  return arg;
}
```

`<T>` 선언 이후 2번 등장했기에 올바르게 사용했다고 볼 수 있다.

```ts
function third<A, B, C>(a: A, b: B, c: C): C {
  return c;
}
```

A와 B는 선언 외에 한 번만 등장했기에 제네릭으로 표현할 이유가 없다.

제네릭 타입 매개변수는 두 번 등장해야 한다는 말은

-> 둘 이상의 위치에서 타입 간 관계를 표현할 때 사용해야 한다라는 뜻인 것 같다.

## 아이템 52

```ts
declare function double<T extends string | number>(x: T): T;

const num = double(12);
// const num: 12
const str = double('x');
// const stre: 'x'
```

해당 경우는 타입을 오히려 너무 과하게 구체적으로 만들었다. 매개변수가 string 타입이면 string 타입이 반환되어야 하지만 리터럴 문자열 타입이 반환되었다.

-> 실제 반환값은 'xx'이므로 결과와 타입이 맞지 않다.

```ts
// 일반 오버로딩
function double(x: string): string;
function double(x: number): number;
function double(x: string | number): string | number;

function double(x: string | number): string | number {
  if (typeof x === 'string') {
    return x + x;
  }
  return x + x;
}

// 제네릭을 사용한 단일 오버로딩
function double<T extends string | number>(
  x: T,
): T extends string ? string : number;

function double(x: string | number): string | number {
  if (typeof x === 'string') {
    return x + x;
  }
  return x + x;
}

const num = double(12);
console.log(num); // 24
```

제네릭을 사용해서 단일 오버로딩을 정의해 사용할 수 있다.

## 아이템 53

```ts
declare function double<T extends number | string>(
  x: T,
): T extends string ? string : number;
```

```ts
type Comparable<T> = T extends Date
  ? Date | number
  : T extends number
    ? number
    : T extends string
      ? string
      : never;

declare function isLessThan<T>(a: T, b: Comparable<T>): boolean;

let dateOrStr = Math.random() < 0.5 ? new Date() : 'A';
// Date | string

isLessThan(dateOrStr, 'B');
```

이 경우 문제가 발생한다.

`dateOrstr`이 `Date`일 수도 있는데 `isLessThan`함수에서 `Date < 'B'`와 같은 비교를 하게 된다.

즉, 의도하지 않은 조합까지 허용될 수 있다.

이 경우 Tuple을 사용해 분배를 막으면 된다.

```ts
type Comparable<T> = [T] extends [Date]
  ? Date | number
  : [T] extends [number]
    ? number
    : [T] extends [string]
      ? string
      : never;
```

TS에서 분배가 일어나려면 `T extends U ? X : Y`형태여야 한다. 왼쪽이 가공되지 않은 타입 매개변수 그 자체여야 한다.

하지만 튜플로 감싸게 되면 `T`자체가 아닌 가공된 `[T]`라는 튜플 타입이 되어버려 분배 조건을 만족하지 않는다.

## 아이템 54

```ts
interface Checkbox {
  id: string;
  checked: boolean;
  [key: string]: unknown;
}

interface Checkbox {
  id: string;
  checked: boolean;
  [key: `data-${string}`]: unknown;
}
```

문자열을 인덱스 타입으로 사용하면 잉여 속성 체크의 이점을 모두 잃게 되고, 사용자의 의도와 맞지 않는 동작이 허용되는 오류가 발생한다.

## 아이템 55

순수 함수에 단위 테스트를 작성하는 것처럼, 복잡한 타입과 타입 선언에도 테스트를 작성하라는 얘기이다.

### 1. 호출만 테스트하지 말자

```ts
declare function map<U, V>(array: U[], fn: (value: U) => V): V[];

map(['2017', '2018'], (value) => Number(value));
```

호출만 하면 결과가 제대로 추론되는지는 검사하지 않는다 -> 반환 타입까지 검사해야 한다.

```ts
// 예시
import { expectTypeOf } from 'expect-type';

const result = map(['2017', '2018'], (value) => Number(value));

expectTypeOf(result).toEqualTypeOf<number[]>();
```

### 2. 할당 가능성을 체크해라

```ts
function assertType<T>(value: T) {}

const n = 12;
assertType<number>(n); // 통과
```

n의 타입은 리터럴 `12`지만 통과한다.

### 3. 콜백은 내부에서 추론되는 타입도 검사한다

콜백을 받는 API라면 최종 반환 타입만 보지 말고, 콜백 매개변수가 올바르게 추론되는지도 확인해야 한다.

```ts
expectTypeOf(
  map(['john', 'paul'], function (name, index, array) {
    expectTypeOf(name).toEqualTypeOf<string>();
    expectTypeOf(index).toEqualTypeOf<number>();
    expectTypeOf(array).toEqualTypeOf<string[]>();
    expectTypeOf(this).toEqualTypeOf<string[]>();

    return name.length;
  }),
).toEqualTypeOf<number[]>();
```

검사할 대상

- 함수의 반환 타입
- 콜백의 각 매개변수 타입
- 제네릭 타입 추론 결과
- API에 포함된다면 `this` 타입

직접 만든 타입을 테스트하려면 vitest, expect-type, tsd 같은 라이브러리를 사용하는 것이 좋다.

타입의 구조뿐 아니라 표시 형식까지 테스트하려면 eslint-plugin-expect-type을 사용하자.

## 아이템 56

```ts
type T21 = '2' | '1';
//   ^? type T21 = "2" | "1"

type T123 = '1' | '2' | '3';
//   ^? type T123 = "2" | "1" | "3"
```

일반적으로 요소가 나열된 순서대로 표시되지만, 겹치는 타입의 유니온은 다르게 표시된다.

```ts
interface BlogComment {
  commentId: number;
  title: string;
  content: string;
}

type PartComment = PartiallyPartial<BlogComment, 'title'>;
//   ^? type PartComment =
//          Partial<Pick<BlogComment, "title">> &
//          Omit<BlogComment, "title">
```

객체 형식으로 타입을 보여주지 않고, 구현을 보여준다 -> Resolve를 앞에 붙이면 객체 형식으로 타입을 보여준다.

```ts
type PartiallyPartial<T, K extends keyof T> = Resolve<
  Partial<Pick<T, K>> & Omit<T, K>
>;

type PartComment = PartiallyPartial<BlogComment, 'title'>;
//   ^? type PartComment = {
//          title?: string | undefined;
//          commentId: number;
//          content: string;
//      }
```

복잡하게 합성된 타입을 이해하기 쉽게 펼쳐서 보여주는 기능이라고 생각하면 된다.

하지만 `Date`타입 같은 경우 내부 타입을 풀어서 보여주는 것은 불필요하기에 필요한 경우에만 사용하자

## 아이템 57

```ts
function sum(nums: readonly number[]): number {
  if (nums.length === 0) {
    return 0;
  }

  return nums[0] + sum(nums.slice(1));
}
```

배열 요소들의 합을 구하는 재귀 함수가 있다.

```txt
1 + sum([2, 3])
1 + (2 + sum([3]))
1 + (2 + (3 + sum([])))
1 + (2 + (3 + 0))
```

이런 과정으로 계산되지만, 재귀 호출이 끝난 다음에 작업이 남아 있어 각 호출은 자기 상태를 기억해둬야 하고 콜 스택이 계속 쌓인다.

너무 깊어지면 오버플로가 발생한다.

-> **꼬리 재귀**를 사용해서 해결할 수 있다.

### 꼬리 재귀

```ts
function sum(nums: readonly number[], acc = 0): number {
  if (nums.length === 0) {
    return acc;
  }

  return sum(nums.slice(1), nums[0] + acc);
}
```

꼬리 재귀는 재귀 호출 후에 할 일이 남아 있지 않다. 재귀 호출 자체가 마지막 작업이기 때문에 이전 함수 호출의 값을 유지하지 않아도 된다.

이 개념을 타입스크립트에도 적용할 수 있다.

## 아이템 58

조건부 타입, 템플릿 리터럴 타입, 재귀를 조합하면 복잡한 계산을 할 수 있다.

-> SQL 문자열을 분석해 결과 타입을 추론하는 경우, 모든 것을 직접 구현해야 한다.

이 경우 **코드 생성**을 사용하면 된다.

코드 생성은 일반 프로그램이 스카마나 쿼리를 읽고 `.ts`파일을 만들어 주는 방식이다.

개발자는 복잡한 구현은 코드 생성 도구에게 맡기고 만들어준 타입만 사용하면 된다.

이때 주의해야 할 점은 원본이 변하면 기존 타입 파일도 최신화시켜줘야 한다는 점이다.

-> CI에서 코드 생성과 `git diff`를 실행하는 것이 좋다.
