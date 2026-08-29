## 아이템 59 완전성 체크를 위해 never 타입 사용하기

switch-case를 사용할때 혹시라도 누락된 케이스가 있는지 확인하기 위해서 never타입을 사용할 수 있다.

never타입은 공집합이기 때문에 switch-case 최하단으로 내려왔을 때 그 값이 never가 아니라면 case가 누락된 것이다.

책에서는

```ts
function assertUnreachable(value: never): never {
  throw new Error(`Missed a case! ${value}`);
}
```

이런 함수를 switch-case의 default:에 배치하여서 런타임에도 체크 가능하게끔 해놨다.

이런 패턴은 반환값이 있는 경우 조금 더 빛을 발한다 따라서 함수의 반환값에 원하는 값을 선언해놓자
그래야 never 패턴이 의마기 생긴다.

```ts
default:
const exhaustiveCheck: never = shape;
// ~~~~~~~~~~~~~~~ Type 'Line' is not assignable to type 'never'.
throw new Error(`Missed a case: ${exhaustiveCheck}`);
```

```ts
    default:
      shape satisfies never
      //    ~~~~~~~~~ Type 'Line' does not satisfy the expected type 'never'.
      throw new Error(`Missed a case: ${shape}`);
```

assertUnreachable 외에 이렇게 2가지 방식으로도 체크가 가능하다

> 가위바위보 같은 모든 경우의 수를 따져야할 때도 never가 빠진 경우의 수를 체크해주는 역할을 해서 아주 유용하다

## 아이템 60 객체를 순회하는 노하우 이해하기

객체를 순회할 때는 `for-of`와 `Object.entries`를 사용할 수 있다.
Object.entries는 타입 별로 신경 안쓰고 단순히 객체의 키와 값을 순회할 때 쓴다 -> 안정성이 조금 떨어지더라도 에러는 안 냄
대신 안정성을 조금 더 신경쓰려면 for-of를 쓰면 된다.

for-of 내부에서 ['a','b','c'] as const;처럼 key의 타입을 좁혀서 사용할 수 있다.
하지만 이건 개발자가 key를 잘 일치시켜줘야한다.

정리해보면 불변 객체를 순회할 때는 for of를 쓰고, 값이 변하는 객체를 순회할 때는 Object.entries를 쓰면 된다.
하지만 Object.entries도 키와 값의 타입을 다루기는 까다롭기 때문에 Map이 더 적절한 방안이 될 수 있다.

## 아이템 61 레코드 타입을 사용해 값 동기화하기

> 어떤 타입에 속성이 추가됐을 때, 그 타입과 관련된 다른 코드도 반드시 같이 수정되어야 한다면, Record<keyof T,...>으로 컴파일러가 그 수정을 강제하게 만들라는 의미

열린 실패: 새로운 상황이 추가되었을 때 더 넓게 허용하는 방향으로 동작함

실제로는 아래와 같이 써볼 수도 있음

```ts
const messages: Record<RequestStatus, string> = {
  idle: "대기 중",
  loading: "불러오는 중",
  success: "완료",
  error: "오류가 발생했습니다",
};
```

이렇게 하면, 4가지 상태를 빼먹지 않을 수 있음!

## 아이템 62 가변 함수 모델링을 위해 나머지 매개변수와 튜플 타입 사용하기

함수의 한 인자에 따라 나머지 인자의 개수나 타입이 달라진다면 단순한 optional parameter로는 관계를 정확하게 표현하기 어렵다.

이때 조건부 타입으로 tuple 타입을 선택하고 이를 rest parameter에 적용할 수 있다.

```ts
function buildURL<Path extends keyof RouteQueryParams>(
  route: Path,
  ...args: RouteQueryParams[Path] extends null
    ? []
    : [params: RouteQueryParams[Path]]
) {}
```

[]는 추가 인자가 없음을, [params: T]는 타입 T의 인자가 하나 필요함을 의미한다.
따라서 첫 번째 인자의 타입에 따라 함수의 나머지 인자 개수와 타입을 연결할 수 있다.

결론은 인자가 2개이상 있는데, 첫 번째 인자에 따라서 나머지 매개변수들이 변한다면 튜플 타입 + rest를 사용하자!

## 아이템 63 배타적 OR를 모델링하기 위해 선택적 never 속성 사용하기

타입스크립트의 타입 레벨 OR는 "포함적 OR"이다. 이게 뭔소리임?

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

여기서는 타입 오류가 나지 않는다. 구조적 타이핑 때문이다.
하지만 우리가 원하는 것은 thing1에는 shirtColor만 올 수 있게 하고, thing2에는 hairColor만 올 수 있게 하는것이다. (배타적 OR)

표준적인 방법은 `선택적 never`타입을 추가하는 것이다.
사실 더 좋은 방법은 태그된 유니온인데, 태그를 명시적으로 추가할 수 없거나 추가하고 싶지 않은 경우에는 선택적 never를 쓰면 된다.

```ts
interface OnlyThingOne {
  shirtColor: string;
  hairColor?: never;
}
interface OnlyThingTwo {
  hairColor: string;
  shirtColor?: never;
}
```

## 아이템 64 공식 명칭에는 상표 붙이기

태그된 유니온 대신 상표(brand)라는게 있다.

태그된 유니온은 1개의 값이 여러개의 상태가 될 수 있을 때 유용하다.
하지만 상표는 런타임에서는 사실상 동일한데, 타입 수준에서만 의미적으로 구분하고 싶을 때 주로 사용한다.

meter, seconds 같은 것도 사실상 다 같은 number인데 TS적으로만 구분을 할 수 있다.

이를 구분하는 방법은 불가능한 타입을 선언해놓고, 이를 타입가드를 이용해서 좁히고 난 다음에 사용하는 방식이다.

일반적으로는 타입이 number & { \_brand: 'meters' }; 이걸로 될 수 없으니까 타입가드를 이용해서 검증을 한 단계 거친다는 말이다.

```ts
string
// 그냥 문자열

Email
// 이메일임이 보장된 문자열

T[]
// 그냥 배열

SortedList<T>
// 정렬됨이 보장된 배열

string
// 그냥 문자열

AbsolutePath
// 절대 경로임이 보장된 문자열
```

이렇게 의미적으로 좁히는 것이다.
