## 아이템 59

```ts
type Shape =
  | { kind: 'circle'; radius: number }
  | { kind: 'square'; size: number };

function getArea(shape: Shape): number {
  switch (shape.kind) {
    case 'circle':
      return Math.PI * shape.radius ** 2;

    case 'square':
      return shape.size ** 2;
  }
}
```

모양에 따라 넓이를 구하는 함수이다.

```ts
type Shape =
  | { kind: 'circle'; radius: number }
  | { kind: 'square'; size: number }
  | { kind: 'triangle'; width: number; height: number };
```

`triangle`이 추가된 경우 기존 `getArea`를 수정해야 한다.

모든 switch를 지난 후 마지막에는 `never` 타입만 남는다 -> 이 점을 이용해 빠진 것을 검사하자

새로운 경우를 추가하면 기존 코드가 자동으로 깨지기 때문에 검사가 가능하다.

## 아이템 60

```ts
const obj = {
  one: 'uno',
  two: 'dos',
  three: 'tres',
};
for (const k in obj) {
  const v = obj[k];
}
```

k의 타입은 string이지만 obj 객체에는 'one', 'two', 'three'만 존재하기 때문에 오류가 발생한다.

이때 `as keyof typeof obj` 타입 단언을 사용하면 오류는 해결되지만, 타입 단언은 그저 타입 체크를 무효화하기 위한 수단이기 때문에 다른 방법을 사용해야 한다.

k의 타입이 `string`으로 추론되는 이유는 **구조적 타이핑**때문이다. 다른 속성이 존재할 수 있기 때문에 `string`으로 추론하는 것이다.

타입 문제 없이 객체의 키와 값을 순회하고 싶다면 `Object.entries`를 사용하면 된다.

```ts
function foo(abc: ABC) {
  for (const [k, v] of Object.entries(abc)) {
    //        ^? const k: string
    console.log(v);
    //          ^? const v: any
  }
}
```

키와 값의 타입을 다루려면 객체보다는 `Map`타입을 사용하면 더욱 간단하다.

## 아이템 61

객체와 해당 객체를 사용해 그리는 함수가 있을 때
1. 변경될 때마다 그리기
2. 새로운 값과 비교하기
둘 다 이상적인 방법이 아니다.

새로운 속성이 추가될 때 개발자가 직접 함수를 고치도록 유도하는 것이 좋다.

-> 타입 체커가 오류를 발생시키도록 하자

실패에 열림(실패해도 허용할건지)과 실패에 닫힘(실패하면 거부할거지) 중 어떤 것에 해당하는지 인지하는 것이 중요하다.

## 아이템 62 가변 함수 모델링을 위해 나머지 매개변수와 튜플 타입 사용하기

```ts
interface RouteQueryParams {
  '/': null,
  '/search': { query: string; language?: string; }
  // ...
}
```

경로에 따라 그에 맞는 쿼리 매개변수를 가지는 인터페이스가 있다.

루트 페이지는 아무런 매개변수를 받지 않기 때문에 이를 사용하는 함수에서 `params` 매개변수에는 옵셔널(?)과 any 타입을 사용할 수 있다.

```ts
function buildURL(route: keyof RouteQueryParams, params?: any) {
  return route + (params ? `?${new URLSearchParams(params)}` : '');
}
```

`any`타입은 타입 검사를 회피하기 때문에 사용하지 않는 것이 좋다는 것을 계속 강조해왔다.

```ts
function buildURL<Path extends keyof RouteQueryParams>(
  route: Path,
  params: RouteQueryParams[Path]
) {
  return route + (params ? `?${new URLSearchParams(params)}` : '');
}
```

따라서 route 매개변수를 제네릭으로 만들고, params는 route에 따라 결정되도록 구체화해주었다.

이 경우 타입 체크가 정상적으로 작동할 것처럼 보인다. 하지만 루트 페이지가 또 문제이다.

-> 루트 페이지는 아무값도 받지 않기 때문에 루트 페이지인 경우에만 params에 `null`을 지정해줘야 한다.

그냥 `null`을 추가해주면 아무런 문제가 없어 보인다. 실제로 호출부에서 `null`을 추가하면 아무런 문제 없이 정상 동작한다. 하지만 매번 `null`을 붙이는 것은 번거롭고 원하는 설계가 아닐 것이다.

이를 해결하기 위해 조건부 타입과 나머지 매개변수를 활용할 수 있다.

```ts
function buildURL<Path extends keyof RouteQueryParams>(
  route: Path,
  ...args: (
      RouteQueryParams[Path] extends null
      ? []
      : [params: RouteQueryParams[Path]]
    )
) {
  const params = args ? args[0] : null;
  return route + (params ? `?${new URLSearchParams(params)}` : '');
}
```

여기서 튜플 타입을 쓰는 이유는 매개변수의 개수까지 타입으로 모델링하기 위함이다.

## 아이템 63 배타적 OR를 모델링하기 위해 선택적 never 속성 사용하기

배타적 OR : A 또는 B. 둘 다는 안됨

포함적 OR : A 또는 B. 혹은 둘 다

타입스크립트는 구조적 타입 시스템이기 때문에 타입스크립트의 OR(|)는 포함적 OR이다.

하지만 배타적 OR를 원하는 경우가 있을 것이다. 이때는 **선택적 never**를 사용할 수 있다.

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

`never` 타입에는 어떤 값도 할당할 수 없기 때문에 이런 설계가 가능하다.

## 아이템 64 공식 명칭에는 상표 붙이기

```ts
interface Vertor2D{
	type: '2d';
	x: number;
	y: number;
}
```

여기서 사용된 `type` 속성은 태그의 역할을 한다. 이러한 명시적 태그는 객체 타입에만 추가할 수 있다.

타입 시스템에서만 동작하면서도 명시적 태그와 동일한 역할을 하는 것이 **상표**이다.

```ts
type AbsolutePath = string & {_brand: 'abs'};
```

태그된 유니온은 어떤 종류인지 판별하기 위해 사용한다면, 상표는 의미적으로 다른 값을 섞지 못하게 구분하기 위해 사용한다.

주로 ID 종류 구분, 검증된 문자열, 절대경로/상대경로, 정렬된 배열, 이메일 주소, 특정 단위 같은 걸 실수로 섞지 못하게 하는 용도로 사용한다.
