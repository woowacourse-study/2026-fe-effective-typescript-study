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
