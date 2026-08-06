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

### 아이템 31. 문서에 타입 정보 쓰지 않기

주석으로 타입 알려주는 것이 왜 나쁘냐?

코드 바뀌어도 주석은 안 따라감.
사람이 손으로 쓰는 거라 아무도 검증 안 해줘서, 결국 코드랑 다른 얘기하는 주석이 되기 쉬움

```ts
// 나쁜 예: 주석으로 타입 설명
// nums는 숫자 배열, 반환값은 숫자
function sum(nums) { ... }

// 좋은 예: 타입 구문 자체가 설명
function sum(nums: number[]): number { ... }
```

타입 구문은 컴파일러가 매번 체크해주니까 코드랑 항상 일치함.
주석은 타입으로 못 담는 것만 -> 함수 목적, 사용 예시, 왜 이렇게 짰는지 담당하면 됨. 매개변수 하나 부연 설명하고 싶으면 `@param` 쓰면 됨.

> 값 안 바뀐다는 주석은 필요 없나?
> `readonly` 붙이면 됌. 굳이 주석으로 남길 필요 없음

변수명에 타입 우겨넣는 것도 지양. `ageNum` 대신 `age: number`로 나눠서 쓰면 됨 -> 다만 단위 헷갈리는 숫자는 예외, `temperatureC`, `timeMs`처럼 단위를 이름에 남기는 건 오히려 버그 줄여줌

## 아이템 32. 타입 별칭에 null이나 undefined 포함하지 않기

```ts
type User = { id: string; name: string } | null;
```

이렇게 짜면 안 됨.
-> 이유는 사람이 타입 이름을 읽는 방식 때문임.

> 왜 문제가 되나?
> `User`라는 이름만 보면 "id, name 있는 사용자"만 떠오르지, "사용자 아닐 수도"는 잘 안 떠오름
> 결국 이 타입 쓰는 곳에서 null 체크를 깜빡하기 쉬움

타입 체커 입장에서는 상관없음. 별칭을 그대로 펼쳐서 검사하니까. 문제는 사람이 읽을 때임

**해결 방법 - 쓰는 자리에서 null 드러내기**

```ts
function getUser(id: string): User | null { ... }
```

이러면 시그니처만 봐도 null 나올 수 있다는 게 바로 보임. 객체 속성 하나가 `null`이나 `undefined`인 건 상관없음
-> 문제 되는 건 별칭 전체를 감싸는 가장 바깥쪽의 null임

## 아이템 33. 타입 주변에 null 값 배치하기

null 여부를 안쪽 여기저기 흩어놓지 말고 바깥 경계 한 곳으로 몰아넣자.

```ts
function extent(nums: Iterable<number>) {
  let min, max;
  for (const num of nums) {
    if (!min) {
      min = num;
      max = num;
    } else {
      min = Math.min(min, num);
      max = Math.max(max, num);
      //             ~~~ 'number | undefined' 형식의 인수는
      //                 'number' 형식의 매개변수에 할당될 수 없습니다.
    }
  }
  return [min, max];
}
```

사람은 min이 있으면 max도 당연히 있다는 거 직관적으로 앎. 근데 타입스크립트는 둘을 독립된 변수로 봐서 그 관계를 모름

> 실제 가능한 조합은?
> 컴파일러 입장에선 네 가지(둘 다 undefined / min만 있음 / max만 있음 / 둘 다 있음) 다 가능해 보임
> 근데 진짜 나올 수 있는 건 "둘 다 undefined" 아니면 "둘 다 있음" 두 가지뿐임

하나의 튜플로 묶으면 잘못된 조합 자체가 안 만들어짐

```ts
function extent(nums: Iterable<number>) {
  let minMax: [number, number] | null = null;
  for (const num of nums) {
    if (!minMax) {
      minMax = [num, num];
    } else {
      const [min, max] = minMax;
      minMax = [Math.min(min, num), Math.max(max, num)];
    }
  }
  return minMax;
}
```

null 될 수 있는 지점을 `minMax` 하나로 모아놨으니, 쓰는 쪽에서는 딱 한 번만 체크(`!` 단언이든 if든) 하면 끝남

클래스도 같은 함정 있음. 프로퍼티 여러 개를 fetch로 채워야 한다고 일단 `null`로 초기화해두고 나중에 `init()`으로 채우는 방식은 피하기 -> 인스턴스가 "덜 채워진 상태"랑 "다 채워진 상태"를 둘 다 가질 수 있게 되고, 쓰는 쪽에서 매번 채워졌는지 신경 써야 함

> 정리
> fetch 먼저 다 끝내고, 값 다 준비된 다음에 인스턴스 생성하기
> null 처리를 클래스 내부가 아니라 바깥(생성 전 시점)으로 미는 것 -> 이게 이 아이템 제목이 말하는 "주변에 배치하기"임
