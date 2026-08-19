## 아이템 43

`any`를 넓은 스코프에서 사용하면 타입이 전파되어 이후의 동작까지도 영향을 미치게 된다.

-> 가능한 좁은 스코프에서 사용해라. 함수의 반환 타입을 명시해 any 타입이 바깥에 영향을 미치는 것을 막는 것과 같은 관점이다.

`@ts-ignore`는 바로 **다음 줄의 TS 에러를 무시**한다.

`@ts-expect-error`는 **다음 줄에는 타입 에러가 발생할 거라고 예상**한다. 실제 에러가 있으면 무시하지만, 에러가 없어지면 TypeScript가 오히려 에러를 발생시킨다.

-> 차라리 이 지시어를 사용해 오류가 수정되었음을 확인하자

객체 또한 같은 관점에서 최소한의 스코프에만 `any`를 사용하자.


## 아이템 44

대부분의 상황에서는 any보다 구체적인 타입으로 표현할 수 있을 것이다. 하지만 필연적인 경우에 any 타입을 그대로 쓰지않는 것을 권장한다.

```ts
function getLength(array: any[]) {
	return array.length;
}
```
만약 배열이라면 `any[]` 만 써도 함수 내의 .length 타입이 체크되고, 반환 타입이 number로 추론되고, 함수 호출 시 매개변수가 배열인지 체크된다.

`any`를 어쩔 수 없이 사용하더라도 `any`그 자체를 사용하지 말고 `() => any`, `{[id: string]: any}` 처럼 형태라도 구체적으로 사용해야 한다.

## 아이템 45

안전하게 작성된 함수 구현체와 타입 시그니처 중 하나만 선택해야 한다면 타입 시그니처를 선택해야 한다.

-> 내부를 안전하게 만들기 위해 반환 타입을 `unknown` 으로 선언하면, 모든 호출자가 반복해서 타입 단언을 해야 한다. 차라리 함수 내부에 타입 단언을 사용하고 사용자에게 정확한 타입을 제공하는 것이 낫다.

``` tsx
function isMountainPeak(value: unknown): value is MountainPeak {
  return (
    typeof value === "object" &&
    value !== null &&
    "name" in value &&
    typeof value.name === "string" &&
    "continent" in value &&
    typeof value.continent === "string" &&
    "elevationMeters" in value &&
    typeof value.elevationMeters === "number" &&
    "firstAscentYear" in value &&
    typeof value.firstAscentYear === "number"
  );
}

export async function fetchPeak(
  peakId: string
): Promise<MountainPeak> {
  const value = await checkedFetchJSON(
    `/api/mountain-peaks/${peakId}`
  );

  if (!isMountainPeak(value)) {
    throw new Error("잘못된 MountainPeak 응답");
  }

  return value; // 여기서는 MountainPeak로 좁혀짐
}
```

또다른 방법으로는 함수 단일 오버로딩이 있다.

-> 근본적으로 타입 단언과 다를 바가 없어 데이터 검증은 수행해야 한다.

타입 단언문을 사용했으면 해당 로직은 단위 테스트를 꼭 수행하자.

## 아이템 46

`any`와 `unknown` 모두 모든 값을 받을 수 있다.

하지만 `any`는 검사 없이 다른 타입으로 사용이 가능하고, `unknown`은 값을 사용하기 전에 반드시 타입을 좁혀야 한다.

`unknown` 구체화하기
- `typeof`, `instanceof`, `in` 등의 조건 검사
- 사용자 정의 타입 가드 -> 사용자 정의 타입 가드도 타입 단언만큼 위험하다! (올바르게 구현했는지 알 수 있는 방법이 없기 때문인 것 같음)

제네릭`<T>`를 사용하는 거보다 `unknown`을 사용하자. 제네릭을 사용하면 안전해 보이지만 이것 또한 타입 단언과 다를 바 없다.

일반적인 상황에서는 `unknown`을 사용하되, `null, undefined`를 사용하지 않는 경우에만 `{}`를 사용하면 된다.

## 아이템 47

몽키 패치란 런타임에 객체에 속성을 추가하는 것. 즉, 원래 없던 속성을 기존 객체에 추가하는 것이다.

사실 몽키 패치를 사용해야 하는 경우가 있는지는 잘 모르겠다.

> [!note] 
> 1. 서버가 HTML에 초기 데이터를 삽입할 때
> 2. 오래된 외부 스크립트가 전역 객체를 제공할 때

이런 경우가 있지만 요즘 React/Next를 사용할 때 몽키 패치를 사용할 일은 거의 없다고 보면 된다.

혹시 사용을 하게 된다면 간단하게 `as any` 타입 단언을 사용하지 말고 별도의 객체로 분리하거나 **타입 보강**을 사용하면 된다.

## 아이템 48

> **TypeScript의 타입 검사를 통과했다고 해서 런타임 안전성이 완전히 보장되는 것은 아니다.**

무결성이 깨졌다는 것은 정적 타입과 실제 런타임 값이 달라졌다는 것을 의미한다.

```ts
const xs = [1, 2, 3];
const x = xs[3];

// 정적 타입: number
// 런타임 값: undefined
```

###  대표적인 무결성 함정

#### `any`

`any`는 타입 검사를 무효화한다.

```ts
function logNumber(x: number) {
  console.log(x.toFixed());
}

const value: any = "hello";
logNumber(value); // 런타임 오류
```

가능하면 `unknown` 사용하거나,  `any`가 필요하면 사용 스코프를 제한해야 한다.

#### 타입 단언과 타입 가드

```ts
const value: number | null = null;
logNumber(value as number); // 타입 오류만 제거
```

`as` 또한 타입 검사를 무효화하기 때문에 런타임에서 오류가 발생한다.

 -> 조건문을 통해 타입 좁히기를 수행하자

#### 배열과 객체 조회

TypeScript는 인덱스가 실제로 존재하는지 확인하지 않는다.

```ts
const names: Record<string, string> = {
  "007": "James Bond",
};

const name = names["008"];
// 정적 타입: string
// 런타임 값: undefined
```

값의 타입에 명시적으로 `undefined`를 추가하거나

```ts
const names: Record<string, string | undefined> = {};
```

또는 다음 옵션을 사용한다.

```ts
{
  "noUncheckedIndexedAccess": true
}
```

#### 클래스 계층의 이중변동성

자식 클래스가 부모보다 좁은 매개변수를 받도록 재정의해도 TypeScript가 허용할 수 있다.

```ts
class Parent {
  foo(value: string | number) {}
}

class Child extends Parent {
  foo(value: number) {
    value.toFixed();
  }
}

const parent: Parent = new Child();
parent.foo("hello"); // 타입 오류 없음, 런타임 실패
```

자식 메서드의 시그니처를 부모와 동일하게 유지해야 한다.

#### 배열·객체의 가변성

```ts
function addAnimal(animals: Animal[]) {
  animals.push(new Fox());
}

const hens: Hen[] = [new Hen()];
addAnimal(hens); // Hen[]에 Fox가 들어감
```

`Hen[]`을 `Animal[]`로 취급하는 것은 읽기만 할 때는 괜찮지만, 수정하면 문제가 된다.

```ts
function inspectAnimals(animals: readonly Animal[]) {
  // animals.push(...) 불가능
}
```

함수의 매개변수를 변경하지 않도록 `readonly` 를 추가하거나, 요소를 직접 추가하게 하는 대신 새로운 값을 반환하도록 해야 한다.

#### 할당 가능성과 선택적 속성

TypeScript 객체 타입은 선언된 속성만 가질 수 있는 타입이 아니다.(구조적 타이핑)

```ts
interface Person {
  name: string;
}

interface AgedPerson extends Person {
  age?: number;
}

const original = {
  name: "Kim",
  age: "42 years",
};

const person: Person = original;     // 추가 속성 age가 가려짐
const aged: AgedPerson = person;     // 허용됨
aged.age?.toFixed();                 // 실제 age는 string
```

aged로 할당하는 부분에서 무결성이 깨진다.

-> 선택적 속성을 무분별하게 추가하지 않기, 충돌하기 쉬운 `type`, `value`, `age`보다 구체적인 이름 사용, 서로 다른 의미라면 `ageInYears`, `formattedAge`처럼 분리

## 아이템 49

`any` 타입을 좁은 스코프에서 사용할지라도 코드에 아무런 영향이 없는 것은 아니다. 

-> `noImplicitAny` 설정을 켜두어도 마찬가지다.

명시적 any 타입이나 서드 파티 타입 선언으로 인해 `any`가 코드 내에 남아있을 수 있다.

이때,  npm의 `type-coverage`를 통해 사용된 `any` 를 추적할 수 있다.

`--detail` 플래그를 붙이면 `any`를 사용한 곳을 모두 출력해준다.

> [!note]
> 사용하려면 type-coverage 패키지를 설치해야 한다.
> 
> --strict를 붙이면 안전하지 않은 타입 단언도 잡아낸다.

`tpye-coverage`를 CI에 추가하면 바로 인지할 수 있다.

애초에 처음부터 `any`를 사용하지 않으면 `type-coverage`를 사용해서 얻는 이익은 적다.

-> 다만 직접 작성하지 않아도 유입될 수 있기 때문에 유입되는 `any`를 방지하는 목적으로 사용해도 좋다.
