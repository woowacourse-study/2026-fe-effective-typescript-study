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

---

### 아이템 31. 문서에 타입 정보 쓰지 않기

주석으로 타입 알려주는 것이 왜 나쁘냐?

코드 바뀌어도 주석은 안 따라감.
사람이 손으로 쓰는 거라 아무도 검증 안 해줘서, 결국 코드랑 다른 얘기하는 주석이 되기 쉬움ㅈ모

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

---

### 아이템 32. 타입 별칭에 null이나 undefined 포함하지 않기

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

---

### 아이템 33. 타입 주변에 null 값 배치하기

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

### 아이템 34. 인터페이스를 유니온으로 묶기 vs 필드를 유니온으로 만들기

이름이 헷갈리는데 둘을 구분하면

- **유니온의 인터페이스**: `interface` 하나 안에서 속성 값이 `A | B | C`인 경우
- **인터페이스의 유니온**: `interface1 | interface2` 처럼 인터페이스 자체를 유니온으로 묶는 경우

책 결론은 후자를 쓰라는 거임.
이유는 전자는 "말이 안 되는 조합"을 타입 시스템이 못 막음.

```ts
interface Layer {
  layout: FillLayout | LineLayout | PointLayout;
  paint: FillPaint | LinePaint | PointPaint;
}
```

이렇게 두면 `layout`이 `FillLayout`인데 `paint`는 `LinePaint`인 조합도 타입 에러 없이 통과됨. 근데 실제로는 layout이랑 paint가 항상 짝을 이뤄야 하는 관계임. 타입에 그 관계가 전혀 안 드러나 있음.

```ts
interface FillLayer {
  layout: FillLayout;
  paint: FillPaint;
}
interface LineLayer {
  layout: LineLayout;
  paint: LinePaint;
}
interface PointLayer {
  layout: PointLayout;
  paint: PointPaint;
}
type Layer = FillLayer | LineLayer | PointLayer;
```

이렇게 쪼개고 유니온으로 다시 합치면, "유효한 조합"만 타입으로 존재하게 됨. 잘못된 조합은 애초에 만들 수가 없어짐.

포인트는 "필드 안에 유니온이 있으면 무조건 나쁘다"가 아님. 서로 묶여서 움직이는 속성들을 각자 따로 유니온/옵셔널로 풀어놨을 때만 문제가 됨.

선택적 속성(`?`) 쓸 때도 똑같은 함정 있음.

```ts
interface Person {
  name: string;
  placeOfBirth?: string;
  dateOfBirth?: Date;
}
```

이거 두 필드가 항상 같이 있거나 같이 없어야 하는데, 타입만 보면 "둘 다 있음 / 하나만 있음 / 둘 다 없음" 네 가지 조합이 전부 유효해 보임. 실제로 유효한 건 두 가지뿐인데.

```ts
interface Name {
  name: string;
}
interface PersonWithBirth extends Name {
  placeOfBirth: string;
  dateOfBirth: Date;
}
type Person = Name | PersonWithBirth;
```

`extends`는 상속이라기보다 "얘도 Name의 속성은 포함하고 있다"는 표시 정도로 보면 됨. 이렇게 두면 "이름만 있음" / "이름+출생 정보 다 있음" 두 상태만 존재하고, "출생지만 있고 날짜는 없음" 같은 애매한 상태는 타입 레벨에서 아예 안 생김.

이 패턴 제일 흔한 형태가 태그된 유니온(discriminated union)임. 각 인터페이스에 `kind: 'fill'` 같은 리터럴 태그 하나씩 박아두면, `switch (layer.kind)`만으로 타입스크립트가 알아서 좁혀줌. 필드 여러 개가 세트로 움직인다 싶으면 옵셔널로 흩뿌리지 말고 객체로 묶어서 유니온 만드는 게 국룰.

### 아이템 35. string은 생각보다 넓은 타입임

타입 = 그 타입에 들어올 수 있는 값의 집합. 이 관점에서 보면 `string`은 사실상 "아무거나"에 가까움.

열두 달 중 하나를 받고 싶어서 필드 타입을 `string`으로 뒀다 치면, `"Octobar"` 같은 오타도 컴파일 통과함. `string`은 `any`랑 비슷한 급으로 취급해도 될 정도로 헐렁한 타입임.

리터럴 유니온으로 좁히면 얻는 게 두 가지임.

- 값이 다른 함수로 넘어가도 타입 정보가 안 죽고 따라감
- 타입 자체에 주석 달듯 의미를 붙일 수 있음 (`type RecordingType = 'studio' | 'live'`)

### pluck 함수로 보는 좁히기 단계

객체 배열에서 특정 키 값만 뽑는 함수 만든다고 하면.

```ts
function pluck(records: any[], key: string): any[] {
  return records.map((r) => r[key]);
}
```

`any`, `string` 둘 다 문제. `key`에 존재하지도 않는 문자열을 넣어도 안 걸림.

```ts
function pluck<T>(records: T[], key: keyof T) {
  return records.map((r) => r[key]);
}
```

`keyof T`로 바꾸면 최소한 "존재하는 키만" 넣을 수 있게 됨. 근데 반환 타입을 보면 여전히 헐렁함. `keyof T`가 유니온이니까 반환값도 그 유니온에 걸리는 모든 값 타입의 합집합이 되어버림. `Album`이 `artist: string`, `title: string`, `releaseDate: Date`, `recordingType: 'studio' | 'live'`로 이루어져 있다면

```
T[keyof T] = string | string | Date | ('studio' | 'live')
```

여기서 `'studio' | 'live'`는 `string`의 부분집합이라 그냥 `string`에 흡수돼버림. 결국 `(string | Date)[]`. `key`로 뭘 넣었는지랑 상관없이 반환 타입이 뭉뚱그려짐.

```ts
function pluck<T, K extends keyof T>(records: T[], key: K): T[K][] {
  return records.map((r) => r[key]);
}
```

제네릭을 하나 더 씀. 여기서 `K`는 "이번 호출에서 실제로 넘긴 그 키 하나"로 고정됨 (유니온 전체가 아니라). 그래서

```ts
pluck(albums, "releaseDate"); // Date[]
pluck(albums, "artist"); // string[]
pluck(albums, "recordingType"); // ('studio' | 'live')[]
pluck(albums, "recordingDate"); // 컴파일 에러, 애초에 없는 키
```

호출 시점에 넘긴 키 값에 따라 반환 타입이 그때그때 정확하게 따라옴. 여기서 제네릭 `K`의 역할은 "함수를 여러 타입에 재사용 가능하게"가 아니라, "입력한 키와 반환 타입 사이의 의존 관계를 타입 레벨에 그대로 새기는 것"에 더 가까움.

---

### 아이템 36. 특수값에는 별도의 타입 사용하기

`find` 함수가 못 찾으면 -1을 리턴하는 관례, 어디서나 본 패턴이다

```ts
function find(arr: number[], target: number): number {
  // 못 찾으면 -1 리턴
}
```

문제는 반환 타입이 그냥 `number`라는 점이다. -1이 "없음"을 뜻한다는 정보는 코드 어디에도 남지 않는다. 호출부에서 `arr[idx + 1]`처럼 바로 써버리면 인덱스 -1 + 1 = 0에서 엉뚱한 값을 꺼내온다. 타입 체커는 이걸 막을 방법이 없다, 애초에 number는 다 number이기 때문이다.

해결방법은 "없음"이라는 상태를 값이 아니라 타입으로 표현하는 것이다.

```ts
type NetworkState =
  | { state: "loading" }
  | { state: "error"; error: Error }
  | { state: "success"; data: string };
```

세 상태를 유니온으로 쪼개면 `data`는 success일 때만 존재한다는 게 타입 레벨에서 강제된다. null 대신 태그된 유니온을 쓰라는 얘기가 아니라, 특수한 케이스가 하나 생기면 그 케이스를 표현할 전용 타입을 만들라는 게 요지다.

---

### 아이템 37. 선택적 속성 지양하기

```ts
interface Config {
  darkMode: boolean;
  unitSystem?: UnitSystem;
}
```

`unitSystem`이 없을 때 "imperial"을 기본값으로 쓴다고 치면, 이 필드를 참조하는 모든 코드가 `?? "imperial"`을 반복해야 한다. 한 곳이라도 빠뜨리면 그 자리만 다른 동작을 하는데, 타입 체커는 아무 말도 안 한다.

정답은 생성 시점에 정규화. optional은 입력 단계에만 두고, 내부에서 쓰는 타입은 required로 못박는다

```ts
interface InputConfig {
  darkMode: boolean;
  unitSystem?: UnitSystem;
}
type Config = Required<InputConfig>;
```

optional을 남겨둬도 되는 경우는 딱 두 가지. 이미 존재하는 API 형태를 그대로 따라야 할 때, 그리고 값이 진짜 있어도 그만 없어도 그만일 때(중간 이름 같은 것?) 뿐이다. optional 필드가 하나 늘 때마다 상태 조합은 배로 불어난다는 걸 기억할 것!

---

### 아이템 38. 같은 타입의 매개변수를 반복하지 않기

```ts
drawRect(25, 50, 75, 100, 1);
```

이 호출만 보고 무엇을 그리는지 맞출 수 있는 사람은 없다. 다섯 인자 전부 number라서, 좌표와 크기 순서를 통째로 바꿔 넣어도 컴파일은 통과한다. 버그는 런타임에 화면을 보고서야 발견된다.

의미가 다른 값은 타입으로 분리해야 순서 실수가 컴파일 타임 에러가 된다

```ts
interface DrawRectParams {
  x: number;
  y: number;
  width: number;
  height: number;
  opacity: number;
}
function drawRect(params: DrawRectParams) {
  /* ... */
}

drawRect({ x: 25, y: 50, width: 75, height: 100, opacity: 1.0 });
```

객체 하나로 묶는 순간 인자 순서라는 개념 자체가 사라진다. 매개변수가 서너 개를 넘어가면 리팩터링할 타이밍이라고 보면 된다.

---

### 아이템 39: 타입 간 차이점을 모델링하기보다 같은 타입으로 통합하기

#### 문제 상황을 먼저 분류해보기

"같은 데이터인데 타입이 두 개"인 상황은 대충 세 가지 원인으로 나뉜다.
원인이 다르면 해결책도 달라서 구분해두는 게 낫다.

**1. 표기 규칙 차이**

```ts
// 서버가 주는 것
interface PostResponse {
  post_id: number;
  created_at: string;
  thumbnail_url: string;
}

// 프론트에서 쓰고 싶은 것
interface Post {
  postId: number;
  createdAt: string;
  thumbnailUrl: string;
}
```

의미는 100% 같은데 껍데기만 다르다. 가장 흔하고, 가장 없애기 쉬운 케이스이다.

**2. 완성도 차이**

```ts
interface UploadInput {
  file: File;
  visibility?: "public" | "private"; // 안 주면 기본값
}

interface Upload {
  file: File;
  visibility: "public" | "private"; // 정규화 후엔 항상 있음
}
```

같은 개념의 "채워지기 전 / 채워진 후"다.

**3. 직렬화 경계 차이**

```ts
// JSON에서 온 것
{ createdAt: '2026-08-11T09:00:00Z' }  // string

// 앱 안에서 쓰는 것
{ createdAt: new Date(...) }           // Date
```

이건 물리적으로 통합이 불가능하다. JSON에 Date 타입이 없기 때문이다.

#### 차이를 그냥 두면 뭐가 비싼가?

타입이 2개면 신경 쓸 게 2개가 아니라 대략 이만큼 늘어난다.

- 변환 함수 (`toPost`, `toPostResponse`)
- 변환을 **빠뜨렸는지** 확인할 방법 : 이게 제일 위험함. `post_id`를 안 바꿔줬는데 `undefined`가 조용히 흘러가는 버그가 생김.
- 필드 하나 추가할 때 고쳐야 할 파일 개수
- 새로 온 팀원이 "그래서 어떤 걸 써야 하나요" 물어보는 시간..?

특히 프론트에서는 여기에 **폼 상태 타입**까지 끼어들면 같은 데이터에 대해 타입이 3개가 된다.
-> API 응답 타입 / 도메인 타입 / 폼 타입

#### 해결 순서

**애초에 안 생기게 만들기**

변환 함수를 잘 짜는 게 아니라 **변환할 일 자체를 없애는 것**이다.
-> 백엔드랑 협의해서 응답 키를 카멜로 맞추거나, ORM 레벨에서 매핑을 걸거나
"내 쪽 코드만 고칠 수 있다"고 생각하면 이 선택지가 안 보인다.

**변환 지점을 딱 한 곳으로 몰기**

못 바꾸면, 변환이 코드 곳곳에 흩어지는 걸 막는 게 두 번째 해결방법일 것 같다.
API 레이어 밖으로는 `PostResponse`가 절대 새어나가지 않게 한다.

```ts
export async function fetchPost(id: number): Promise<Post> {
  const res = await http.get<PostResponse>(`/posts/${id}`);
  return toPost(res);
}
```

컴포넌트 어디서도 `post_id`를 못 보게 하는 게 목표이다.

**변환을 타입 레벨로 자동화**

키 이름 규칙이 기계적이면 손으로 인터페이스를 두 번 쓸 이유가 없다.

```ts
type SnakeToCamel<S extends string> = S extends `${infer Head}_${infer Tail}`
  ? `${Head}${Capitalize<SnakeToCamel<Tail>>}`
  : S;

type ObjectToCamel<T> = {
  [K in keyof T as K extends string ? SnakeToCamel<K> : K]: T[K];
};

type Post = ObjectToCamel<PostResponse>;
// { postId: number; createdAt: string; thumbnailUrl: string }
```

`PostResponse`에 필드가 추가되면 `Post`는 자동으로 따라온다. 동기화 실수가 구조적으로 불가능해진다.

다만 이건 **런타임 변환까지 해주진 않는다.** 타입만 맞고 실제 객체는 그대로일 수 있으니
런타임 변환 함수와 짝을 맞춰야 한다. 여기서 어긋나면 오히려 더 위험한 상태가 된다.

---

### 아이템 40: 부정확한 타입보다는 미완성 타입 사용하기

#### 정밀도와 정확성은 다른 느낌이다

이 아이템이 헷갈리는 이유는 "정밀 = 좋음"이라고 뭉뚱그려 생각하기 때문이다. 축이 두 개다.

- **정밀도(precision)**: 허용하는 값의 범위가 얼마나 좁은가
- **정확성(correctness)**: 그 범위가 현실과 일치하는가

|               | 현실과 일치                        | 현실과 불일치                      |
| ------------- | ---------------------------------- | ---------------------------------- |
| **넓은 타입** | 미완성 -> 못 잡지만 거짓말은 안 함 | .                                  |
| **좁은 타입** | 이상적                             | **최악** -> 잡아주는 척하면서 틀림 |

#### 정밀도를 올리다가 꺾이는 순간

예를 들어 필터 표현식을 타입으로 표현한다고 해보자. 한 칸씩 조여본다.

```ts
// 0단계
type Filter = any;

// 1단계 : 최소한의 형태
type Filter = (string | number | Filter)[];

// 2단계 : 첫 요소는 연산자
type Op = "and" | "or" | "not" | "==" | ">" | "in";
type Filter = [Op, ...unknown[]];

// 3단계 : 연산자별 인자 개수까지
type Filter =
  | ["not", Filter]
  | ["==", string, string | number]
  | ["and", Filter, Filter];
```

3단계가 제일 좋아 보이는데, 여기서 사고가 난다.
`['and', a, b, c]`처럼 인자가 3개 오는 게 실제로 유효하다면? `['in', field, ...values]`는?
-> 규칙을 잘못 알고 조인 순간, 멀쩡한 코드가 에러가 된다.

꺾였는지 알려주는 신호 세 가지:

1. **정상 값이 에러 난다** : 거짓 양성. 제일 명확한 신호
2. **에러 메시지가 해석 불가능해진다** : 유니온이 커지면 TS는 "어느 멤버에도 대입 불가"만 뱉는다. 실제 원인이 세 번째 인자여도 첫 줄을 가리킴
3. **`as any`가 늘어난다** : 타입이 현실을 못 따라가면 사람들은 타입을 고치는 대신 우회한다. 이 순간 타입은 안전장치가 아니라 통행세가 됨

책은 이 구간을 "불쾌한 골짜기"라고 부른다. 정밀도를 올릴수록 좋아지다가, 어느 지점에서 오히려 떨어진다.

### 좌표 예시 : 3번째 원소

```ts
type Coordinate = [number, number]; // [경도, 위도]
```

깔끔해 보이지만 틀렸다. GeoJSON 좌표에는 **고도가 세 번째로 올 수 있다.**
저렇게 선언하면 유효한 데이터가 타입 에러가 난다.

```ts
type Coordinate = number[]; // 미완성이지만 거짓말은 아님
```

길이 검증을 못 하니 아쉽지만, 틀린 것보단 낫다.

---

### 아이템 41. 해당 분야의 용어로 타입명 짓기

구조가 정확해도 이름이 흐리면 결국 구현을 열어봐야 함.

```ts
interface Animal {
  name: string;
  endangered: boolean;
  habitat: string;
}
```

필드가 부족한 게 아니라, 이 분야에 이미 있는 표준 용어를 안 썼다는 게 문제다.

```ts
interface Animal {
  commonName: string;
  genus: string;
  species: string;
  status: ConservationStatus; // IUCN 적색목록 등급
  climates: KoppenClimate[]; // 쾨펜 기후 구분
}
```

`status`가 IUCN 등급이 되는 순간 "멸종 위기의 기준이 뭐냐"는 논쟁이 사라짐. 기준은 우리가 정하는 게 아니기 때문이다.
전문용어를 쓰라는 건 어렵게 쓰라는 게 아님 -> **합의된 어휘를 재발명하지 말라는 뜻**이다.

**세 가지 기준**

1. 같은 개념엔 같은 단어 : `fetchUser`/`getMember`/`loadAccount`가 다 같은 걸 가져오면 읽는 사람은 차이가 있다고 믿는다.
2. `data`, `info`, `item`, `object`는 이름이 아니다 : 장바구니도 `item`, 주문 확인도 `item`이면 구별이 안 된다. 이름이 안 떠오르면 타입의 책임이 아직 안 정리된 것.
3. 계산 방식이 아니라 데이터의 실제 구현체로 : `INodeList`보다 `Directory`. 구현은 바뀌어도 정체는 잘 안 바뀌니 이름 수명도 길어진다.

---

### 아이템 42. 데이터 형태를 추측해서 타입 만들지 않기

API, DB, 설정 파일, 서드파티
-> 우리가 다루는 타입의 상당수는 바깥에서 온다. 이런 타입은 **작성하는 게 아니라 가져오는 것**이다.

값 몇 개 찍어보고 만들면 내가 본 샘플에만 맞는 타입이 나온다.

```ts
type Geometry = Point | LineString | Polygon; // GeometryCollection 누락
```

명세엔 있는데 테스트 데이터에 없었을 뿐이다.
이런 누락은 개발 중이 아니라 운영 중에 드러나고, 뒤늦게 케이스를 추가하면 그 타입을 쓰던 코드가 전부 깨진다.

요청 바디도 서버가 정한 계약이다.
명세에서 나온 타입이면 필드를 빠뜨렸을 때 컴파일 단계에서 잡히고, 짐작해 만든 거면 500 응답으로 돌아온다.

```ts
export interface CreateCommentRequest {
  body: string;
  postId: string;
  title: string;
}
```

-> 결국 **타입은 내 머릿속이 아니라 도메인에서 온다.**
