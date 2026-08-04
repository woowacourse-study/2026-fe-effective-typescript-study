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
