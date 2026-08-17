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
