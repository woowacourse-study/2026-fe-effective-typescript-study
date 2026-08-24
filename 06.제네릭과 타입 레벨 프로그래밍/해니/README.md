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
