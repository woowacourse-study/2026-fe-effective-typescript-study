### 아이템 29. 유효한 상태만 타입으로 표현하기

타입을 잘 설계하면 버그의 절반은 애초에 안 생김. 로직이 틀려서가 아니라 "말이 안 되는 조합"을 타입이 허용해서 버그가 나는 경우가 많음

```ts
interface State {
  pageText: string;
  isLoading: boolean;
  error?: string;
}
```

이 타입만 보면 `isLoading: true`인데 `error`도 채워진 상태가 가능함 -> 로딩 중인데 이미 에러났다는 모순

> 왜 문제가 되나?
> if (isLoading) {...} else if (error) {...} else {...} 이런 식으로 분기 짜다가 케이스 하나 빠뜨려도 컴파일러가 아무 말 안 해줌
> 타입이 애초에 말 안 되는 상태까지 허용하고 있어서 그 상태 처리 안 해도 에러가 안 남

**해결 방법 - 태그된 유니온으로 쪼개기**

```ts
interface RequestPending {
  state: "pending";
}
interface RequestError {
  state: "error";
  error: string;
}
interface RequestSuccess {
  state: "ok";
  pageText: string;
}
type RequestState = RequestPending | RequestError | RequestSuccess;
```

state 필드로 판별자를 두면 "로딩 중이면서 동시에 에러"인 상태는 애초에 타입으로 못 만듦
식별 가능한 유니온(태그된 유니온)이라고 부름

페이지 여러 개면 딕셔너리로 관리하는 게 나음

```ts
interface State {
  currentPage: string;
  requests: { [page: string]: RequestState };
}
```

코드는 길어지지만 나올 수 없는 조합 자체가 사라짐

**비행기 조종간 예시**

```ts
interface CockpitControls {
  leftStickAngle: number;
  rightStickAngle: number;
}
```

기장 10도, 부기장 30도면 비행기는 몇 도로 가야 하나? -> 답이 없음. 둘 다 유효한 입력인데 충돌함

> 그럼 평균 내면 되지 않나?
> 차이 작을 땐 되는데 차이 크면(10도 vs 80도) 또 문제. 평균? 큰 쪽 우선? 경고음? 로직으로는 답이 안 나옴
> 진짜 문제는 "충돌을 어떻게 처리할까"가 아니라 애초에 조종간을 독립된 두 값으로 모델링한 거 자체가 잘못임
> 실제 보잉 항공기는 두 조종간이 기계적으로 연결돼 있어서 물리적으로 같은 각도만 가능함

```ts
interface CockpitControls {
  stickAngle: number; // 필드를 하나로 합치면 질문 자체가 사라짐
}
```

> 정리: 충돌 나는 상태를 if문으로 계속 방어하지 말고, 그런 상태 자체를 못 만들게 타입 구조를 바꾸는 게 근본적인 방법

---

### 아이템 30. 엄격하게 생성하고, 너그럽게 사용하기

함수 매개변수는 타입 범위가 넓어도 되는데, 반환할 땐 타입이 최대한 구체적이어야 함

```ts
type LngLat =
  | { lng: number; lat: number }
  | { lon: number; lat: number }
  | [number, number];

interface CameraOptions {
  center?: LngLat;
  zoom?: number;
  bearing?: number;
  pitch?: number;
}
```

매개변수 자리면 이 정도 유연함 괜찮은데, 이 타입이 반환값에도 그대로 쓰이면 문제 생김

```ts
function focusOnFeature(f: Feature) {
  const bounds = calculateBoundingBox(f);
  const camera = viewportForBounds(bounds);
  const {
    center: { lat, lng },
    zoom,
  } = camera;
  // ~~~ lat, lng 프로퍼티 없을 수도 있다는 에러
}
```

> 왜 이런 일이?
> center가 세 가지 형태 중 뭔지 모르니까, 이 값 쓰는 모든 곳에서 형태별로 분기 처리를 떠안게 됨
> 함수 하나 편하자고 넓힌 타입이 호출하는 모든 곳에 부담을 떠넘기는 셈

**해결 방법 - 받는 타입 / 돌려주는 타입 분리**

```ts
interface LngLat {
  lng: number;
  lat: number;
}
type LngLatLike = LngLat | { lon: number; lat: number } | [number, number];

interface Camera {
  center: LngLat; // 반환은 이 형태 하나로 고정
  zoom: number;
  bearing: number;
  pitch: number;
}
interface CameraOptions extends Omit<Partial<Camera>, "center"> {
  center?: LngLatLike; // 입력은 여전히 너그럽게
}
```

Partial + Omit 조합해서 엄격한 원본 타입(Camera) 기준으로 느슨한 입력용 타입(CameraOptions)을 파생시킴
-> 정답 형태는 하나, 받아주는 형태는 여러 개. 반환값 쓰는 쪽은 분기 처리 필요 없어짐

**배열 대신 이터러블 받기**

```ts
function sum(nums: Iterable<number>): number {
  let total = 0;
  for (const n of nums) total += n;
  return total;
}
```

number[]만 받으면 진짜 배열만 되는데, Iterable<number>로 받으면 Set, Map.values(), 제너레이터까지 다 받을 수 있음
-> 매개변수를 그냥 순회만 하는 게 목적이면 Iterable 쓰는 게 더 너그러움

> 정리: 매개변수는 쓰기 편하게 최대한 넓게, 반환값은 분기 처리 필요 없게 최대한 좁고 확정적으로. 둘을 같은 타입 하나로 겸용하면 그 타입 쓰는 모든 곳에 분기 부담이 퍼짐
