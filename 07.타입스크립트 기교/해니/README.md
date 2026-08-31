### 아이템 59. 완전성 체크를 위해 never 타입 사용하기

타입 체커는 하면 안 되는 일은 잘 잡아준다.

반대로 해야 하는데 빠뜨린 일은 그냥 넘어간다.

유니온에 멤버를 추가하고 그걸 처리하는 분기를 안 만드는 경우가 그렇다.

분기를 하나씩 처리하다 보면 남은 후보가 줄어들고, 전부 처리했다면 마지막에는 `never`만 남는다.

-> 마지막 지점에서 값이 `never`인지 확인하면 빠진 분기를 컴파일 타임에 잡을 수 있다.

```ts
function assertNever(value: never): never {
  throw new Error(`처리하지 않은 값: ${value}`);
}
```

이 함수를 `default`에 두면 새 멤버가 추가됐을 때 오류가 난다.

`const check: never = event`나 `event satisfies never`로도 같은 검사가 가능하다.

주의할 점은 함수의 **반환 타입을 명시해야 한다**는 것이다.

반환 타입을 적지 않으면 분기가 빠져도 오류 대신 반환 타입에 `undefined`가 섞이는 식으로 넘어간다.

`switch`문 전용 기법도 아니다. 가위바위보처럼 경우의 수를 빠짐없이 따져야 할 때도 쓸 수 있다.

> 유니온의 모든 경우를 다뤘는지 확인하려면 `never`를 쓴다.

### 아이템 60. 객체를 순회하는 노하우 이해하기

`for-in`으로 객체를 순회하면 키가 `string`으로 나온다.

그래서 그 키로 값을 꺼내면 오류가 발생한다.

`as keyof typeof obj`로 단언하면 오류는 사라지지만, 이건 문제를 없앤 게 아니라 검사를 끈 것이다.

`let k: keyof typeof obj` 형태의 반복자를 써도 마찬가지다.

키가 `string`으로 추론되는 이유는 **구조적 타이핑** 때문이다.

함수가 객체를 매개변수로 받는다고 하면, 선언된 속성만 가진 객체가 들어온다는 보장이 없다.

속성이 더 많은 객체도 들어올 수 있으니 실제 키가 선언된 키보다 많아질 수 있다.

-> 키를 좁히는 게 오히려 거짓말이라 `string`을 고를 수밖에 없다.

타입까지 신경 쓸 필요 없이 키와 값만 돌면 되는 경우에는 `Object.entries`를 쓰면 된다.

다만 키는 `string`, 값은 `any`로 나오니 안전하다고 하긴 어렵다.

더 정밀하게 가려면 키 배열을 `as const`로 선언해두고 그걸 순회하면 된다.

대신 객체와 키 배열을 어긋나지 않게 유지하는 건 개발자의 몫이다.

키와 값의 타입을 제대로 다루고 싶다면 객체보다 `Map`이 낫다.

> 키가 `string`으로 나오는 건 버그가 아니라 구조적 타이핑의 결과다.

### 아이템 61. 레코드 타입을 사용해 값 동기화하기

옵션 객체가 있고 값이 바뀌면 화면을 다시 그리는 상황을 생각해보자.

어떤 속성은 바뀌면 다시 그려야 하고, 어떤 속성은 그럴 필요가 없다.

무엇이 바뀌든 일단 다시 그리면 정확하지만 불필요하게 자주 그린다. -> 실패에 열린 접근

다시 그릴 속성만 골라 검사하면 빠르지만 새 속성이 추가됐을 때 조용히 누락된다. -> 실패에 닫힌 접근

둘 다 이상적이지 않다.

특히 후자는 잘못돼도 아무 신호가 없다는 점이 위험하다.

주석으로 "속성 추가하면 여기도 고쳐주세요"라고 적어두는 것도 결국 지켜지지 않는다.

이럴 때는 판단 기준을 타입으로 만들어두면 된다.

```ts
const SHOULD_REDRAW: Record<keyof ChartProps, boolean> = {
  xs: true,
  ys: true,
  color: true,
  onClick: false,
};
```

`Record<keyof T, boolean>`은 키를 하나도 빠뜨릴 수 없다.

-> 속성이 추가되는 순간 오류가 나고, 추가한 사람이 다시 그릴지 여부를 반드시 결정하게 된다.

59번의 `never`와 결이 같다. 사람이 기억해서 챙기던 일을 타입 체커에게 넘기는 것이다.

> 정확도를 희생하며 성능을 추구해서는 안 된다.

### 아이템 62. 가변 함수 모델링을 위해 나머지 매개변수와 튜플 타입 사용하기

경로마다 필요한 쿼리 매개변수가 다른 상황을 생각해보자.

```ts
interface RouteQueryParams {
  "/": null;
  "/search": {
    query: string;
    language?: string;
  };
  // ...
}
```

URL을 만드는 함수는 경로와 매개변수를 함께 받는다.

`params`를 옵셔널 `any`로 두면 어떤 경로에 어떤 값을 넘겨도 통과한다.

-> 검사를 포기한 것이나 마찬가지다.

`route`를 제네릭으로 잡고 `params`의 타입이 그 경로에서 나오게 하면 둘 사이가 연결된다.

```ts
function buildURL<Path extends keyof RouteQueryParams>(
  route: Path,
  params: RouteQueryParams[Path],
) {
  return route + (params ? `?${new URLSearchParams(params)}` : "");
}
```

여기까지는 좋은데 루트 경로가 걸린다.

`'/'`의 매개변수 타입이 `null`이라 `buildURL('/', null)`처럼 쓸모없는 값을 매번 넘겨야 한다.

문제는 매개변수의 타입이 아니라 **개수**다.

-> 개수까지 타입으로 표현해야 한다.

나머지 매개변수 자리에 튜플 타입을 놓으면 그게 가능하다.

`[]`는 더 받을 인자가 없다는 뜻이고, `[params: T]`는 `T` 하나를 받는다는 뜻이다.

조건부 타입으로 둘 중 하나를 고르게 하면 된다.

```ts
function buildURL<Path extends keyof RouteQueryParams>(
  route: Path,
  ...args: RouteQueryParams[Path] extends null
    ? []
    : [params: RouteQueryParams[Path]]
) {
  const params = args.length > 0 ? args[0] : null;
  return route + (params ? `?${new URLSearchParams(params)}` : "");
}
```

호출부에서 보면 경로에 따라 아예 다른 함수처럼 동작한다.

```ts
buildURL("/");
buildURL("/search", { query: "typescript", language: "ko" });
```

튜플 요소에 이름을 붙여두면 에디터 힌트에도 `params`로 표시된다.

> 앞쪽 인자가 뒤쪽 인자의 개수나 타입을 결정한다면 나머지 매개변수와 튜플 타입으로 모델링한다.

### 아이템 63. 배타적 OR를 모델링하기 위해 선택적 never 속성 사용하기

일상적으로 "A 또는 B"라고 하면 보통 둘 중 하나만 고르라는 뜻으로 받아들인다.

하지만 TS의 `|`는 둘 중 하나면 된다는 뜻이지, 하나만이어야 한다는 뜻은 아니다.

```ts
interface ThingOne {
  shirtColor: string;
}
interface ThingTwo {
  hairColor: string;
}
type Thing = ThingOne | ThingTwo;

const bothThings = {
  shirtColor: "red",
  hairColor: "blue",
};

const thing1: ThingOne = bothThings; // ok
const thing2: ThingTwo = bothThings; // ok
```

두 속성을 다 가진 객체가 양쪽 모두에 할당된다.

필요한 속성만 갖추면 나머지는 따지지 않는 **구조적 타이핑**의 결과다.

정말로 둘 중 하나만 허용하고 싶다면, 반대쪽 속성은 없어야 한다는 사실까지 타입에 적어줘야 한다.

이때 쓰는 게 선택적 `never`다.

```ts
interface OnlyThingOne {
  shirtColor: string;
  hairColor?: never;
}
interface OnlyThingTwo {
  hairColor: string;
  shirtColor?: never;
}
type ExclusiveThing = OnlyThingOne | OnlyThingTwo;
```

`never`에는 어떤 값도 할당할 수 없으니 `hairColor`에 실제 값이 들어오면 막힌다.

선택적이기 때문에 속성 자체를 갖지 않는 경우는 그대로 통과한다.

-> 딱 우리가 원하던 동작이다.

속성이 늘어나면 일일이 적기 번거로우니 헬퍼로 만들어둘 수도 있다.

```ts
type XOR<T1, T2> =
  | (T1 & { [K in Exclude<keyof T2, keyof T1>]?: never })
  | (T2 & { [K in Exclude<keyof T1, keyof T2>]?: never });
```

물론 구분 필드를 붙일 수 있는 상황이라면 태그된 유니온이 먼저다.

태그를 붙이기 어려운 경우에 선택적 `never`를 꺼내면 된다.

> 유니온은 포함적 OR다. 배타적으로 만들려면 선택적 `never`를 쓴다.

### 아이템 64. 공식 명칭에는 상표 붙이기

태그된 유니온도 공짜는 아니다.

실제 객체에 구분 필드를 하나 더 넣어야 하고, `string`이나 `number` 같은 기본형에는 아예 붙일 수 없다.

런타임에는 없고 타입 수준에만 존재하는 표식이 **상표**다.

```ts
type AbsolutePath = string & { _brand: "abs" };
```

`string`과의 인터섹션이라 실제 값은 그냥 문자열이다. 런타임 비용이 없다.

대신 `_brand`를 가진 문자열은 자연스럽게 만들어지지 않으니, 이 타입의 값을 얻으려면 검사를 한 번 거쳐야 한다.

```ts
function isAbsolutePath(path: string): path is AbsolutePath {
  return path.startsWith("/");
}
```

이렇게 해두면 절대 경로가 필요한 함수에 검증되지 않은 문자열을 넘길 수 없다.

숫자도 마찬가지다. `Meters`와 `Seconds`처럼 상표를 붙여두면 단위가 다른 값을 섞는 실수를 막을 수 있다.

다만 값을 연산하면 상표가 떨어져 나가므로 다시 붙여줘야 한다.

주로 쓰는 곳은 검증된 문자열(이메일, 절대 경로), 종류가 다른 ID, 정렬이 보장된 배열처럼 **타입은 같지만 의미가 다른** 값들이다.

클래스의 `private` 필드나 `unique symbol`로 상표를 만드는 방법도 있다.

상표 심벌이 밖으로 노출되지 않으면 그 타입의 값을 임의로 만들 수 없어서 더 안전하다.

TS는 구조적 타입 시스템인데, 상표는 그 위에서 명목적 타이핑을 흉내 내는 기법이라고 볼 수 있다.

> 런타임에는 같은 값이라도 의미가 다르다면 상표를 붙여 타입 수준에서 구분한다.
