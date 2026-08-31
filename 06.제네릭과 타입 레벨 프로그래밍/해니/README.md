### 아이템 50. 제네릭이 타입 간 함수라고 생각하기

값 영역에서 반복 줄이는 게 함수, 타입 영역에서 그 역할 하는 게 제네릭
-> 그냥 타입계 함수라고 보면 됨.

```ts
type MyPartial<T> = { [K in keyof T]?: T[K] };
//        ^함수명  ^매개변수    ^본문
type Result = MyPartial<Person>; // 호출
```

`T`는 매개변수, `MyPartial<Person>`은 호출, 나온 타입이 반환값

중요한 건 순수 함수라는 점이다.
같은 입력엔 항상 같은 출력, 부수 효과 없음 -> 하나씩 짚어보는 디버깅이 통함

#### extends는 상속이 아니라 매개변수 타입 애노테이션

```ts
type MyPick<T extends object, K extends keyof T> = { [P in K]: T[P] };
```

키워드 때문에 상속처럼 읽히는데 하는 일은 `function f(x: number)`의 `: number`랑 똑같음
-> 받을 수 있는 범위를 좁히는 것! 여기선 T가 객체, K가 그 객체의 키

제약 안 걸면 `T[P]`가 성립 안 해서 에러가 제네릭 정의부 안쪽에서 터진다.
제약 걸면 호출 지점에서 터짐. 후자가 나음.
-> 잘못 쓴 사람이 잘못 쓴 자리에서 빨간 줄 보는 게 맞으니까

#### 유니온 넣으면 쪼개짐

일반 함수랑 갈라지는 지점이 여기인 것 같다.
유니온 넣으면 인자 하나가 아니라 여러 번 호출된 것처럼 동작함.

```ts
type Elem<T> = T extends (infer U)[] ? U : never;

type A = Elem<string[] | number[]>; // string | number
// 사실상 Elem<string[]> | Elem<number[]> 로 분배됨
```

`{ [K in keyof T] }` 같은 동형 매핑 타입도 마찬가지 -> `Partial<A | B>`는 `Partial<A> | Partial<B>`
분배 막고 싶으면 `[T] extends [any[]]`처럼 대괄호로 감싸면 됨. (자세한 건 아이템 53)

제네릭 새로 만들 땐 유니온, `never`, `any` 세 개 넣어보고 결과 확인하는 습관 들이면 좋을 듯 하다.
특히 `never`는 유니온의 빈 집합이라 분배 결과가 통째로 증발하는 경우 많음

#### 타입 세계의 as any

값 영역에서 `as any`로 에러 뭉개듯, `K & keyof T` 같은 인터섹션을 "에러 사라지니까" 붙이는 순간 그건 골치 아프게 된다.
타입 검사 통과했다는 게 정확성을 보증 못 하게 된다.
원인은 대부분 제약이 없어서니까 -> 인터섹션 말고 `K extends keyof T`로 시그니처를 고치는 게 맞는 수순이다.

### 아이템 51. 불필요한 타입 매개변수 지양하기

제네릭의 황금 법칙에 나온 제네릭을 잘 사용하는 방법

- 타입 매개변수는 두 번 등장
- 타입 매개변수는 여러 값의 타입을 연관시키기 위해 사용됨

-> 타입 매개변수가 한 번만 등장한다면, 정말로 필요한 것인지 신중하게 생각해봐야 함

근데 이 법칙은 제네릭 사용이 잘못 되었을 때 어떻게 수정해야 하는지 충분한 가이드를 제공해주지 않는다.

```ts
function identity<T>(arg: T): T {
  return arg;
}
```

위의 예제는 `<T>` 선언 이후에 2번 등장했으므로 올바르게 사용했다고 볼 수 있다.

```ts
function third<A, B, C>(a: A, b: B, c: C): C {
  return c;
}
```

위의 예제에서, A와 B는 선언 외에 한 번만 등장했기에 올바르게 사용된 것이 아니다.

```ts
function getProperty<T, K extends keyof T>(obj: T, key: K) {
  return obj[key];
}
```

```ts
function getProperty<T, K extends keyof T>(obj: T, key: K) {
  console.log(obj[key]);
}
```

이 둘의 차이점은 반환 타입이다. 1번 예제인 경우 반환 타입이 T[K]로 찍히며, 시그니처를 제대로 써보면 K가 두 번 등장한다.

하지만 2번 예제인 경우 K는 한 번만 사용하므로, 잘못 사용된 것처럼 보인다.

### 아이템 52. 오버로딩 타입보다는 조건부 타입 사용하기

```ts
declare function double<T extends string | number>(x: T): T;

const num = double(12);
// ^? const num: 12
const str = double("x");
//^?  const str: "x"
```

위의 예제는 너무 과하다. 타입이 과하다.
string 타입을 매개변수로 넘기면 string이 반환되어야 한다.
리터럴 문자 x를 넘긴다고 해서 동일한 문자열이 나와야 한다는 것은 아니다. 얘의 더블은 xx여야 한다.

그래서 정확성을 포기하는 것보다 정밀함을 포기하는 것이 낫다.

```ts
declare function double<T extends string | number>(
  x: T,
): T extends string ? string : number;
```

이렇게 조건부 타입을 유니온으로 분리해서 단일 오버로딩을 하는 게 낫다.

### 아이템 53. 조건부 타입에 적용되는 유니온 영역을 제어하는 방법 숙지하기

```ts
declare function double<T extends number | string>(
  x: T,
): T extends string ? string : number;
```

이 경우 조건부 타입에 유니온 적용해서 원하는 결과 얻을 수 있지만, 상황에 따라 이런 패턴을 적용할 수 없을 수도 있다.

```ts
type Comparable<T> = T extends Date
  ? Date | number
  : T extends number
    ? number
    : T extends string
      ? string
      : never;

declare function isLessThan<T>(a: T, b: Comparable<T>): boolean;
```

`isLessThan`의 의도는 첫 번째 인자의 타입에 따라 두 번째 인자로 올 수 있는 타입을 제한하는 것이다. -> `Date`면 `Date | number`, `number`면 `number`,`string`이면 `string`

문제는 `T`에 유니온이 들어올 때 생긴다.

```ts
let dateOrStr = Math.random() < 0.5 ? new Date() : "A";
//  ^? let dateOrStr: Date | string

isLessThan(dateOrStr, "B"); // 통과된다
```

`dateOrStr`은 런타임에 `Date`일 수도, `string`일 수도 있다. 그렇다면 두 번째
인자는 **두 경우 모두에서** 유효해야 하므로, 각 경우의 인터섹션인
`(Date | number) & string`, 즉 `never`가 되어 어떤 값도 받지 못해야 맞다.

그런데 실제로는 `"B"`가 통과한다. 조건부 타입의 **분배** 때문이다.

Comparable<Date | string>
-> Comparable<Date> | Comparable<string>
-> (Date | number) | string

인터섹션이 아니라 유니온으로 펼쳐지면서, `string`이 허용 범위에 들어와 버린다.

#### 튜플로 분배 막기

```ts
type Comparable<T> = [T] extends [Date]
  ? Date | number
  : [T] extends [number]
    ? number
    : [T] extends [string]
      ? string
      : never;
```

분배가 일어나려면 `T extends U ? X : Y`에서 왼쪽이 가공되지 않은 타입 매개변수 그 자체여야 한다. `[T]`처럼 튜플로 한 번 감싸면 더 이상 `T` 자체가 아니라 `[T]`라는 별개의 튜플 타입이므로 분배 조건을 만족하지 않는다.

이제 `Date | string`이 쪼개지지 않고 통째로 비교된다.

- `[Date | string] extends [Date]` -> false
- `[Date | string] extends [number]` -> false
- `[Date | string] extends [string]` -> false
- -> `never`

```ts
isLessThan(dateOrStr, "B");
//                    ~~~ Argument of type 'string' is not
//                        assignable to parameter of type 'never'
```

의도대로 유니온 타입에 대한 비교가 차단된다.

---

### 아이템 54. DSL과 문자열 사이의 관계는 템플릿 리터럴 타입으로 모델링하기

문자열을 타입으로 표현하는 방법은 사실 세 단계다.

`string`은 너무 넓고, 유한한 집합은 유니온으로 충분하다.

```ts
type MedalColor = "gold" | "silver" | "bronze";
```

문제는 그 사이다.
`data-`로 시작하는 속성이나 `tag#id` 형태의 선택자처럼 **개수는 무한한데 규칙은 명확한** 문자열은 유니온으로 적을 수가 없다. 이때 쓰는 게 템플릿 리터럴 타입이다.

```ts
type PseudoString = `pseudo${string}`;

const a: PseudoString = "pseudoscience"; // OK
const b: PseudoString = "science"; // Error
```

`string`과 리터럴 유니온 사이의 빈 공간을 메워주는 도구이다.

#### 1. 인덱스 시그니처의 구멍 막기

`string` 인덱스 시그니처는 잉여 속성 체크의 이점을 전부 없앤다.

```ts
interface Checkbox {
  id: string;
  checked: boolean;
  [key: string]: unknown;
}

const cb: Checkbox = {
  id: "subscribe",
  checked: true,
  chekced: false, // 오타인데 통과된다
};
```

`checked`를 잘못 친 건데 인덱스 시그니처가 다 받아주니 에러가 안 난다. 값은 `undefined`가 되어서 런타임에 가서야 알게 된다.

애초에 열어주려던 건 "아무 속성"이 아니라 "`data-`로 시작하는 속성"이었다. 그럼 키를 그렇게 적으면 된다.

```ts
interface Checkbox {
  id: string;
  checked: boolean;
  [key: `data-${string}`]: unknown;
}
// 'data-testid' -> OK,  chekced -> Error
```

의도한 만큼만 열리고 나머지 보호막은 그대로 남는다.

#### 2. 진가는 제네릭 + 추론과 조합할 때

`querySelector`는 태그로 쿼리하면 구체적 타입을 주지만, ID를 붙이면 포기한다.

```ts
document.querySelector("img"); // HTMLImageElement | null
document.querySelector("img#spectacular-sunset"); // Element | null <- 너무 넓다
```

`tag#id` 형태를 이해하도록 오버로딩할 수 있다.

```ts
type HTMLTag = keyof HTMLElementTagNameMap;

declare global {
  interface ParentNode {
    querySelector<TagName extends HTMLTag>(
      selectors: `${TagName}#${string}`,
    ): HTMLElementTagNameMap[TagName] | null;
  }
}

document.querySelector("img#spectacular-sunset"); // HTMLImageElement | null
```

핵심은 `TagName`이 **넘긴 문자열에서 추론된다**는 점이다.

1. `"img#spectacular-sunset"`을 `` `${TagName}#${string}` `` 패턴에 맞춰본다
2. `#` 앞이 `TagName`으로 추론된다 -> `"img"`
3. 그 결과가 `HTMLElementTagNameMap["img"]`로 이어진다 -> `HTMLImageElement`

문자열 값 하나가 타입 계산의 입력이 된 것이다. 이게 "문자열 사이의 관계를 모델링한다"는 말의 실체다.

#### 3. 부족한 부분 - 공백 문제

CSS에서 공백은 "~의 자손"인데, 위 오버로드는 그걸 모른다.

```ts
document.querySelector("div#container img");
// HTMLDivElement | null <- 틀렸다. 실제로 찾는 건 img다
```

`${string}`이 욕심쟁이(greedy)라서 `"container img"`를 통째로 삼키고 `TagName`을 `"div"`로 추론해버리기 때문이다.

해결은 결합자가 든 선택자를 **먼저** 잡아 넓은 타입으로 되돌리는 오버로드를 위에 두는 것이다. 오버로드는 위에서부터 매칭되기 때문이다.

```ts
declare global {
  interface ParentNode {
    // 1) 공백이 있으면 여기서 먼저 잡힌다
    querySelector(selectors: `${string} ${string}`): Element | null;
    // 2) 순수한 tag#id 형태만 여기로
    querySelector<TagName extends HTMLTag>(
      selectors: `${TagName}#${string}`,
    ): HTMLElementTagNameMap[TagName] | null;
  }
}
```

`>`, `+`, `,` 같은 결합자도 같은 식으로 하나씩 막아야 한다.

이 부분이 어렵게 느껴졌던 건 개념이 어려워서가 아니었다.
템플릿 리터럴 타입은 문자열을 **파싱하는 게 아니라 그냥 패턴을 맞춰볼 뿐**이라, CSS 문법을 이해하지 못하고 예외를 손으로 막는 수밖에 없다. 그 모양새가 지저분해 보였던 것이다.

#### 4. 주의

관계가 **단순할 때** 가장 효과적이다.

복잡한 문자열 파싱을 타입 수준에서 구현하기 시작하면 컴파일러 성능이 나빠지고 에러 메시지를 사람이 읽을 수 없게 된다. 유니온끼리 조합할 때도 조심해야 한다. `${A}${B}`는 A × B개의 리터럴을 만들기 때문에 100 × 100이면 10,000개가 된다.

그리고 복잡한 걸 만들었다면 아이템 55의 테스트가 필수다.

---

### 아이템 55. 타입에 대한 테스트 작성하기

런타임 테스트는 "이 입력에 이 출력이 나오는가"를 본다. 타입 테스트는 **"컴파일러가 내 의도대로 추론하는가"** 를 본다.

검사가 눈에 보이지 않기 때문에, 테스트를 썼다고 착각하기 쉽다는 게 이 아이템의 핵심이다.

#### 1. 호출만 하는 테스트는 반쪽

```ts
declare function map<U, V>(array: U[], fn: (value: U) => V): V[];

map(["2017", "2018"], (value) => Number(value)); // 에러만 안 나는지 볼 뿐
```

이 줄이 확인해주는 건 "타입 에러 없이 호출된다" 하나뿐이다. 결과가 `number[]`인지 `unknown[]`인지는 아무도 보지 않는다. 반환 타입까지 명시적으로 확인해야 한다.

```ts
import { expectTypeOf } from "expect-type";

const result = map(["2017", "2018"], (value) => Number(value));
expectTypeOf(result).toEqualTypeOf<number[]>();
```

#### 2. 할당 가능성 ≠ 동일성

직접 만든 헬퍼는 "같은 타입인가"가 아니라 **"넣을 수 있는가"** 를 검사한다. 그래서 통과하면 안 될 게 통과한다.

```ts
function assertType<T>(value: T) {}

// (1) 더 좁은 타입이 통과
const n = 12;
assertType<number>(n); // 통과 - 실제 타입은 리터럴 12

// (2) 잉여 속성이 있어도 통과
const beatles = ["john", "paul", "ringo"];
assertType<{ name: string }[]>(
  beatles.map((name) => ({ name, inYellowSubmarine: name === "ringo" })),
); // 통과

// (3) 매개변수가 적은 함수가 통과 <- 함수 테스트에서 제일 위험
const double = (x: number) => 2 * x;
assertType<(a: number, b: number) => number>(double); // 통과!
```

(2)는 변수를 거치면 잉여 속성 체크가 걸리지 않기 때문이고, (3)은 매개변수가 적은 함수는 많은 함수 타입에 할당 가능하다는 규칙 때문이다. 특히 (3)은 콜백 시그니처를 테스트할 때 사실상 검사를 안 한 것과 같아진다.

-> 직접 만들지 말고 라이브러리를 쓰자. `toEqualTypeOf`는 동일성을 검사한다.

```ts
expectTypeOf(n).toEqualTypeOf<12>(); // 통과
expectTypeOf(n).toEqualTypeOf<number>(); // 실패 <- 원하던 동작
```

#### 3. 콜백 매개변수와 `this`도 검사한다

콜백을 받는 API는 반환 타입만 보면 안 된다. 사용자가 실제로 겪는 건 콜백을 열었을 때 매개변수가 뭘로 뜨는지다.

```ts
expectTypeOf(
  map(["john", "paul"], function (name, index, array) {
    expectTypeOf(name).toEqualTypeOf<string>();
    expectTypeOf(index).toEqualTypeOf<number>();
    expectTypeOf(array).toEqualTypeOf<string[]>();
    expectTypeOf(this).toEqualTypeOf<string[]>();
    return name.length;
  }),
).toEqualTypeOf<number[]>();
```

`this`도 API 표면의 일부다. 선언에서 빼먹으면 사용자 콜백 안의 `this`가 조용히 `any`가 되는데, 에러가 안 나니까 아무도 모른다.

**검사 대상**: 반환 타입 / 콜백 매개변수 타입 / 제네릭 추론 결과 / (있다면) `this`

#### 4. 에러가 나야 하는 경우도 테스트다

통과하는 케이스만 테스트하면, 타입을 `string`으로 되돌려놔도 테스트는 초록불이다. 특히 54처럼 패턴을 좁혀놨다면 잘못된 값이 실제로 거부되는지 확인해야 한다.

```ts
// @ts-expect-error
const bad: PseudoString = "science";
```

`@ts-expect-error`는 그 줄에서 에러가 **안 나면** 오히려 에러를 낸다. 방어막이 살아있는지 감시해주는 셈이다.

#### 5. 도구

- **vitest** : `expectTypeOf` 내장, 런타임 테스트와 같은 파일에서 사용 가능
- **expect-type / tsd** : 단독 타입 테스트
- **eslint-plugin-expect-type** : 타입의 구조뿐 아니라 **에디터에 표시되는 형식**까지 검사

마지막 게 은근히 중요하다. 타입이 논리적으로 맞아도 호버했을 때 `Pick<Omit<Foo, "a">, "b">`처럼 뜨면 쓰는 사람 입장에서는 모르는 타입이다. 결과가 맞는 것과 읽히는 건 다른 문제고, 공개 API라면 후자도 테스트 대상이다.

---

### 아이템 56. 타입이 표시되는 방식 관리하기

라이브러리 제작 시 타입이 에디터에 어떻게 찍히는지도 API 설계의 일부다.

복잡한 제네릭 연산 체인은 추론 결과가 연산 과정 그대로 노출돼 가독성이 떨어진다.

```ts
type Resolve<T> = T extends Function ? T : { [K in keyof T]: T[K] };

type Padding = { top: number; left: number };
type Margin = { top: number; right: number };
type Combined = Resolve<Padding & Margin>;
// 감싸지 않으면 Padding & Margin 그대로 표시
// 감싸면 { top: number; left: number; right: number } 로 평탄화되어 표시
```

Resolve 같은 매핑 타입으로 감싸면 교차 타입, 매핑 타입 체인 등의 연산 과정이 사라지고 최종 속성 목록만 보인다.

단, 내부 구현이 안 보이게 되어 디버깅이 어려워질 수 있고, 실제로는 같은 타입인데 다르게 표시되어 혼란을 줄 수도 있으므로 공개 API 경계에서만 선택적으로 쓴다.

---

### 아이템 57. 제네릭 타입 반복에는 꼬리 재귀 사용하기

재귀 호출 뒤에 결과를 다시 가공하는 형태는 컴파일러가 각 단계 결과를 스택에 계속 쌓아야 해서 깊은 재귀에서 인스턴스화 한도(약 대략 수백 단계 수준)에 쉽게 도달한다.

```ts
// 일반 재귀 - 호출 후에도 유니온 연산이 남아있음
type Reverse<S extends string> = S extends `${infer First}${infer Rest}`
  ? `${Reverse<Rest>}${First}`
  : S;

// 꼬리 재귀 - Acc에 누적하고 재귀 호출이 마지막 동작
type ReverseTail
  S extends string,
  Acc extends string = ''
> = S extends `${infer First}${infer Rest}`
  ? ReverseTail<Rest, `${First}${Acc}`>
  : Acc;
```

Reverse는 재귀 결과를 받아 First를 앞에 붙이는 연산이 남아있어 스택이 계속 쌓이지만, ReverseTail은 Acc에 이미 계산된 결과를 들고 다니다가 재귀 호출 자체로 끝나므로 TS가 이전 프레임을 버려도 된다.

이 차이 덕분에 꼬리 재귀는 훨씬 긴 문자열이나 배열도 인스턴스화 한도 없이 처리할 수 있다.

---

### 아이템 58. 복잡한 타입 대신 코드 생성 사용하기

TS의 타입 시스템은 튜링 완전하다.

조건부 타입, 템플릿 리터럴 타입, 재귀를 조합하면 타입 레벨에서도 조건문이나 문자열 가공 같은 복잡한 계산을 할 수 있다는 뜻이다.

근데 할 수 있다는 것과 그렇게 푸는 게 좋다는 건 다르다.

SQL 문자열을 분석해 결과 타입을 추론하는 경우를 생각해보면, 필요한 걸 전부 타입 레벨에서 직접 구현해야 한다.

-> 읽기 어려워지고, 틀렸을 때 원인을 찾기도 힘들어진다.

외부 데이터의 경우에는 TS가 데이터의 구조를 정확히 알지 못한다.

그걸 타입으로만 모델링하려고 하면 매우 복잡해지는 경우가 있다.

-> 이럴 때 **코드 생성**을 고려하면 된다!

코드 생성은 일반 프로그램이 스키마나 쿼리를 읽고 `.ts` 파일을 만들어 주는 방식이다.

쉽게 설명하면 스키마를 읽고 타입을 대신 만들어준다고 이해하면 편할 듯싶다.

복잡한 구현은 코드 생성 도구가 맡고, 개발자는 만들어진 타입을 가져다 쓰기만 하면 된다.

주의할 점은 원본이 바뀌면 생성된 타입 파일도 같이 최신화해야 한다는 것이다.

원본과 타입이 어긋나면 타입 체크는 통과하는데 실제 동작에서 깨지게 된다.

-> CI에서 코드 생성과 `git diff`를 실행하는 것이 좋다.

> 타입 레벨 프로그래밍은 강력한 도구이지만 만능은 아니다.
> 복잡한 경우 코드 생성을 고려해야 한다.
