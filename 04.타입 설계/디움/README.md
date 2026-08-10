## 아이템 29

효과적으로 타입을 설계하려면 -> 유효한 상태만 표현할 수 있는 타입을 만드는 것이 중요하다.

### 페이지 요청 상태

``` ts
interface State {
  pageText: string;
  isLoading: boolean;
  error?: string;
}
```

각 속성이 독립적이기 때문에 모든 조합이 허용된다.

``` ts
{
  isLoading: true,
  error: '요청 실패',
  pageText: '이전 페이지 내용'
}
```

타입상으로는 오류가 없지만, 의미상으로는 모순이다.

> 무효한 상태 : 타입은 허용하지만 프로그램에서는 말이 안 되는 상태

### 태그된 유니온으로 해결

``` ts
interface RequestPending {
  state: 'pending';
}

interface RequestError {
  state: 'error';
  error: string;
}

interface RequestSuccess {
  state: 'ok';
  pageText: string;
}

type RequestState =
  | RequestPending
  | RequestError
  | RequestSuccess;
```

가능한 상태를 각각 별도의 타입으로 정의한다 -> 만들 수 있는 상태는 3가지뿐이다.

> [!note] 
> 여기서 말하고자 하는 것 : 사용하는 곳에서 방어로직을 구현하지 말고, 애초에 타입 선언에서 무효한 상태가 나올 수 없도록 하자.

## 아이템 30

> 엄격하게 생성하고, 너그럽게 사용하기

-> 함수의 매개변수는 타입의 범위가 넓어도 되지만, 결과를 반환할 때는 타입의 범위가 구체적이어야 한다.

-> 입력은 유연하지만 결과는 예측 가능하게 만들라는 의미인 것 같다.

### 간단한 예시

문자열 숫자를 실제 숫자로 바꾸는 함수:

```ts
function toNumber(value: number | string): number {
  return Number(value);
}
```

입력은 숫자와 문자열 둘 다 받지만, 반환값은 항상 number이다.

만약 반환 타입이 너그럽다면:
``` ts
function badToNumber(
  value: number | string
): number | string {
  return value;
}

const result = badToNumber('10');

result + 5;
```

result가 string인지 number인지 확인해야 한다.

## 아이템 31

타입스크립트의 타입 구문 시스템은 간결하고, 구체적이며, 쉽게 읽을 수 있도록 설계되었다

-> 함수의 입력과 출력의 타입을 코드로 표현하는 것이 주석보다 더 낫다.

타입 구문은 타입 체크 과정이 있기 때문에 무조건 일치하게 된다. 괜히 불필요하게 주석으로 작성하지말자.

특정 매개변수를 설명하고 싶다면 `@param` 구문을 사용하면 된다.
``` ts
// es-toolkit 예시
/**
* Sorts an object's values based on multiple property names/paths and their corresponding order directions.
*
* @template T The object type
* @param collection The object to sort values from
* @param iteratees The property names/paths to sort by
* @param orders The sort orders
* @returns Returns the new sorted array

* @example
* const obj = {
  * a: { name: 'fred', age: 48 },
  * b: { name: 'barney', age: 34 }
* };
*
* // Sort by name in ascending order
* orderBy(obj, ['name'], ['asc']);
* // => [{ name: 'barney', age: 34 }, { name: 'fred', age: 48 }]
*/
```

매개변수를 변경하지 않는다는 주석 대신 -> `readonly` 사용

## 아이템 32

타입 별칭에는 `null` 이나 `undefined`를 포함하지 않는 것이 좋다.

```ts
type User = { id: string; name: string; } | null;
```
`User`라는 타입 별칭을 보고 '사용자'라고 생각을 할 뿐, '사용자일 수도?'라고 생각하지 않는다.

null을 써야 한다면 `User | null`  형태로 사용하면 된다.

객체 내부의 속성이 null이거나 undefined인 경우는 문제 없다.

## 아이템 33

결국, 특정 값의 null 여부가 다른 값에까지 관련되지 않도록 설계를 하라는 말인 것 같다.

`undefined`를 포함하는 객체는 다루기 어렵기 때문에 권장하지 않는다.

클래스를 생성할 때도 `null`로 초기화하지 말고, 값이 있을 때만(null이 아닐 때만) 클래스를 생성해 `null`로 인해 발생하는 버그를 막자.

## 아이템 34

``` ts
interface Layer {
	layout: FillLayout | LineLayout | PointLayout;
	paint: FillPaint | LinePaint | PointPaint;
}
```
이렇게 타입을 설계하면 layout과 paint 속성이 서로 짝이 맞아야 하는 Layer의 의도를 벗어난다.

-> 각각의 타입을 인터페이스로 분리하고 합쳐야 한다. 유효한 타입만을 정의해야 한다.

각각의 유효한 타입을 모델링해야 타입스크립트가 코드의 정확성을 체크하는 데 도움이 된다.

결국 선택적 속성을 추가할 때 해당 속성이 없는 상태와 존재하는 상태가 모두 유효한지, 속성 간의 관계가 깨지지는 않는지를 함께 고려해야 한다.

## 아이템 35

타입의 범위는 **가능한 값의 집합**이라는 것을 명심하자.

이 관점에서 `string`타입의 범위는 너무 넓다.

-> 예를 들어, 1월 ~ 12월 중 하나의 값이 들어와야 하지만 Octobar처럼 오타가 발생해도 정상적으로 동작한다.

string 타입이어도 리터럴 유니온 등 더 좁은 타입을 사용하는 것이 오류를 막을 수 있다.

좁은 타입을 사용해야
- 다른 곳으로 값이 전달되어도 타입 정보가 유지된다.
- 해당 타입의 의미를 설명하는 주석을 붙여 넣을 수 있다.

### pluck 예시
pluck은 객체 배열에서 특정 속성만 뽑는 함수

``` ts
const albums = [
  {
    title: 'Abbey Road',
    artist: 'The Beatles',
    releaseDate: new Date(),
  },
];

pluck(albums, 'title');
// ['Abbey Road']

pluck(albums, 'releaseDate');
// [Date 객체]
```

`any` 또는 `string`을 사용하면 title, artist, releaseDate가 아닌 어떤 문자열이든 허용된다.

```ts
function pluck<T>(
  records: T[],
  key: string
) {
  return records.map(record => record[key]);
}

pluck(albums, 'releaseDate');
// T = Album
```
`T`는 전달받은 객체 한 개의 타입이다. 타입스크립트는 record가 Album이라는 사실을 알지만, key는 여전히 아무 문자열이나 될 수 있어서 오류가 발생한다.

```ts
function pluck<T>(
  records: T[],
  key: keyof T
) {
  return records.map(record => record[key]);
}
```

따라서 키를 실제 객체가 가진 키로 제한해야 한다.

-> `keyof T`는 'title' | 'artist' | 'releaseDate'가 된다.

하지만 아직도 타입이 넓다. 반환타입인 `T[keyof T]`는 객체가 가진 속성 타입의 합집합이다. 즉 `string | Date`이다.

``` ts
function pluck<T, K extends keyof T>(
  records: T[],
  key: K
): T[K][] {
  return records.map(record => record[key]);
}

const dates = pluck(albums, 'releaseDate');
// T = Album
// K = 'releaseDate'
// T[K] = Album['releaseDate'] = Date
```

이 예제에서 제네릭은 함수를 범용적으로 만들기 위해서만이 아니라, **입력한 키에 따라 반환 타입이 달라지는 관계를 정확히 표현하기 위해** 사용된다.

## 아이템 36

JS의 `indexOf`처럼 특수값(실패한 값)이 정상값과 동일한 타입이면 문제가 발생할 수 있다.

-> 타입스크립트가 버그를 찾을 수 있는 기회를 놓치게 된다.

0, -1, ""보다는 null이나 undefined를 사용하는 것이 좋다.

-> 실패 상태를 타입에 표현했기 때문에 실패 처리를 빼먹은 코드를 발견할 수 있다.

하지만 null과 undefined로 특수값을 표현하는 것이 항상 옳은 것은 아니다 -> 이때는 태그된 유니온을 사용하자.

## 아이템 37

결국 속성이 정말 독립적으로 선택적인 경우가 아니라면, 무작정 선택적 속성으로 만들지 말라는 얘기이다.

이전 아이템에서 언급된 '무효한 상태'를 막는 것도 있지만, 이번 아이템에서는 기존 코드를 건들지 않으려고 새 속성을 선택적으로 추가하면, 타입 체커가 수정이 필요한 사용처를 찾아주지 못한다는 것을 강조한다.

반드시 전달해야 하는 값이라면 필수 속성으로 선언하고, 발생한 타입 오류를 따라 모든 사용처를 수정하는 편이 안전하다.

## 아이템 38

같은 타입의 매개변수를 여러 개 나열하면 타입 체커가 각각의 의미를 구분하지 못하기 때문에, 관련된 값끼리 객체 타입으로 묶자!

``` ts
drawRect(25, 50, 75, 100, 1);
```
순서를 실수해도 모든 값의 타입이 number라서 타입스크립트는 오류를 찾지 못한다.

의미 있는 타입으로 묶거나,
``` ts
interface Point {
  x: number;
  y: number;
}

interface Dimension {
  width: number;
  height: number;
}
```

매개변수를 객체 하나로 만들면 된다.
``` ts
interface DrawRectParams {
  x: number;
  y: number;
  width: number;
  height: number;
  opacity: number;
}

function drawRect(params: DrawRectParams) {
  // ...
}
```

## 아이템 39

두 타입 간 차이점을 없애는 것이 더 낫다.

-> 같은 데이터를 표현 방식만 다르게 만든 여러 타입을 유지하지 말고 하나의 타입으로 통일하자

스네이크 케이스를 사용해 DB에 저장하고, TS 코드에서는 카멜 케이스를 사용하는 경우, 두 타입은 의미상 똑같지만 표현 방식이 달라 항상 변환해야 한다.

-> 변환을 빠뜨리면 버그가 발생하기 때문에 하나의 타입으로 통일하는 것이 좋다.

타입을 통일할 수 없는 경우 변환을 즉흥적으로 작성하지 말고, 타입과 변환 함수를 명확히 해야 한다.

무조건 하나의 타입으로 통합하는 것이 좋지는 않다. 구조가 비슷하더라도 의미가 다르면 별도의 타입으로 유지해야 한다.

## 아이템 40

타입을 무조건 구체적으로 만드는 것이 좋은 것은 아니다.

실제 데이터가 가질 수 있는 형태를 정확히 알지 못하는 상태에서 타입을 좁히면, 정상적인 값까지 타입 오류로 처리할 수도 있다

-> 그럼 결국 타입 단언이나 any로 타입 검사를 회피하게 된다.

실제 데이터의 형태를 정확히 알지 못한다면 유효한 값을 모두 허용하는 넓은 타입을 사용하고 추후 단계적으로 좁혀나가야 한다.
