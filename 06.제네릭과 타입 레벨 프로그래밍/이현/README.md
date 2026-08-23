# 이번 장의 목표는 제네릭이 필요한 상황과 필요하지 않은 상황을 구분하는 것이다!

## 아이템 50 제네릭이 타입 간 함수라고 생각하기

값의 영역에서 반복되는 코드 줄이기는 함수이고, 타입의 영역에서 이 역할을 하는 것이 `제네릭`이다.

걍 타입계의 함수화라고 보는 것이 맞다.

값의 영역에서 as any를 사용하는 것처럼 타입 영역에서도 내장된 `PropertyKey` 인터섹션 기법이 존재한다.
이것도 마찬가지로 사용하지 않는 편이 좋다.

제네릭을 썼으면 주석을 달아주는 것이 좋다.
@template 이런식으로

> 제네릭 타입이 유니온 타입처럼 동작한다 -> 자세한 내용은 아이템 53

제네릭 함수라는 것이 존재한다. function pick<T,K> 이런식으로 쓸 수 있다. 이렇게 쓰면 함수가 호출되는 시점에 값으로부터 타입 매개변수를 추론할 수 있다.
제네릭 함수는 호출할 때 타입을 지정해주는 것이다.

제네릭 타입을 "타입과 타입 간 함수"로 생각하는 편이 좋다.

## 아이템 51 불필요한 타입 매개변수 지양하기

매개변수는 2번 등장해야한다.

```ts
function identity<T>(arg: T): T {
  return arg;
}
```

이건 매개변수가 2번 등장한 올바른 예제이다.
선언 이후 2번 미만으로 등장한다면 정말 올바른 제네릭 사용인지 다시 생각해봐야 한다.

```ts
declare function parseYAML<T>(input: string): T;
```

얘도 1번만 등장하니까 잘못된 제네릭 사용이다.

```ts
function printProperty<T, K extends keyof T>(obj: T, key: K) {
  console.log(obj[key]);
}
```

여기서 T는 2번 사용되지만, K는 1번만 사용되고 있다.
함수가 return이나 그런걸 하지 않기 떄문에 사실상 입력 타입만 제한하고 쓰이는 용도로 끝나서 1번이라고 하는 것 같다.

```ts
function printProperty<T>(obj: T, key: keyof T) {
  console.log(obj[key]);
}
```

따라서 출력용으로는 이렇게 해도 큰 상관이 없다.

그런데 속성 값을 반환해야한다면 이야기가 달라진다.

```ts
function getProperty<T, K extends keyof T>(obj: T, key: K) {
  return obj[key];
}
```

여기서는 K가 반환까지 관여하기때문에 실제로 유의미하게 2번 쓰이는 것이다.

```ts
function getProperty<T>(obj: T, key: keyof T) {
  return obj[key];
}
```

만약 이렇게 쓰게 된다면?
사욜하는 곳에서는 타입을 좁혀야 될 것이다.

```ts
const name = getProperty(user, "name");
// string | number
```

keyof는 유니온이기 때문에 K와 T가 extends로 직접 연결되지 않는 이상 타입이 정확해지지 않는다.

결론은 유의미한 제네릭 매개변수가 선언 이후 2번이상 쓰였는지 확인하는 것이다. 이게 바로 제네릭의 `황금 법칙`

제네릭 사용 법칙
첫 번째: 제네릭 사용 하지말라 = 불필요하면 쓰지말아라 제발

## 아이템 52 오버로딩 타입보다는 조건부 타입 사용하기

```ts
declare function double<T extends string | number>(
  x: T,
): T extends string ? string : number;
```

이렇게 반환할때 타입에도 삼항 연산자를 쓸 수 있다.
오버로딩이 꼭 필요한 상황이 아니라면 먼저 조건부 타입을 떠올려보도록 하자.

## 아이템 53 조건부 타입에 적용되는 유니온 영역을 제어하는 방법 숙지하기

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

이 코드의 의미는 Date는 Date | number랑 비교가능
number는 number와 비교가능
string은 string이랑 비교가능이라는 뜻이다.

잘 작동할 것처럼 보이지만, 여기에 `유니온`이 개입되는 순간 에러를 뿜어낸다.

```ts
let dateOrStr = Math.random() < 0.5 ? new Date() : "A";
//  ^? let dateOrStr: Date | string
isLessThan(dateOrStr, "B"); // ok, but should be an error
```

dateOrStr은 Date | string 이다.
Date | string이라는 것은 Comparable에 넣었을 때
Date | number | string이 된다는 것이다.

그래서 위 예제에서는 에러가 안난다. 그러나 우리는 에러를 내야한다 Date 객체가 들어왔을 때 string과 비교할 수 있으면 안된다.

그래서 우리는 이걸 튜플 타입으로 해결할 수 있다.

```ts
type Comparable<T> = [T] extends [Date]
  ? Date | number
  : [T] extends [number]
    ? number
    : [T] extends [string]
      ? string
      : never;
```

기존의 문제는 제네릭 T의 매개변수로 Date | string이 오면 Comparable<Date> | Comparable<string>으로 된다는 것이었다.

하지만 튜플 타입으로 해버리면
[Date | string] 전체를 나머지와 비교하는 것이기 때문에 올바르게 never타입이 된다.

> never | 무언가 -> never사라짐
> 분배 조건부 타입 관점에서 never가 들어오면 얘는 멤버개수가 0인 유니온처럼 행동해서 결과가 never가 된다.
