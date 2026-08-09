## 아이템 29 유효한 상태만 타입으로 표현하기

타입 설계만 잘 해도 로직의 코드 퀄리티가 많이 오른다. (읽기 쉽고, 버그가 없어진다.)
그럼 타입 설계를 잘 한다는 것이 뭐냐?

동시에 존재할 수 없는 타입을 분리하는 것이다.

```ts
interface State {
  pageText: string;
  isLoading: boolean;
  error?: string;
}
```

여기엔 논리적인 오류가 존재한다.
isLoading이 true이면서 error가 존재할 수 있는 상황이다.
따라서 예상하지 못한 다양한 상황에서 버그가 발생할 수 있다.

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

interface State {
  currentPage: string;
  requests: { [page: string]: RequestState };
}
```

이전거를 개선하면 이렇게 된다. 식별가능한 유니온 타입이라고 부르고 태그된 유니온 타입으로도 불린다.
특정 상태일 때 특정 부가정보만 받도록 한 것이다.

물론 코드는 길어졌지만, 유효한 상태만 나올 수 있도록 타입을 정의해서 코드 안정성이 올라갔다. (로직은 없지만 로직에서 안정성이 확보됨)

#### 앞으로 타입 정의할 때 유효한 상태만 나올 수 있는지 따져보자! 경우의 수 돌려가면서 체크 고고

## 아이템 30 엄격하게 생성하고, 너그럽게 사용하기

여기서 하고싶은 말은 명확하다. 함수를 정의할 때 매개변수로는 더 넓은 타입이 올 수 있어도
반환할 때는 넓은 상태 그대로이면 안 되고, 최대한 타입을 좁혀서 return해야 한다는 것이다.

만약 넓은 타입을 가진채로 반환된다면, 사용하는 곳에서 분기처리를 싹다 해줘야한다.
이렇게 되면 사용하기 어려운 함수가 되어버린다. 우리는 사용하기 쉬운 함수를 만들어야 한다.

> 사용하기 쉬운 API일수록 그 내부는 타입 시스템이 엄격할 것이다.

그럼 어떻게 매개변수는 넓은 타입을 가지면서 좁은 타입을 반환하는 함수를 만들어야 되냐?
2가지 방법이 있음

첫 번째 방법은 유틸리티 타입 안쓰고 그냥 이렇게 하는거임

```ts
interface CameraOptions {
  center?: LngLatLike; // 👈 LngLatLike로 너그럽게!
  zoom?: number;
  bearing?: number;
  pitch?: number;
}
```

두 번째는 유틸리티 타입을 이용한 방법

```ts
interface Camera {
  center: LngLat;
  zoom: number;
  bearing: number;
  pitch: number;
}
interface CameraOptions extends Omit<Partial<Camera>, "center"> {
  center?: LngLatLike;
}
```

어쨌든 결국 중요한 것은 실제 사용하는 타입과 매개변수로 받을 타입 이 2개로 분리하는 것이 중요함! 이것만 기억해둬도 나중에 떠올리기 쉬울듯

이런 패턴을 쉽게 적용할 수 있는게 숫자 배열을 받아 함계를 계산하는 함수임
매개변수로 number[]만 받게하면 진짜 배열밖에 못받는데, Iterable<number>를 사용하면 Map, Set같은 순회가능한 것들을 다 받을 수 있음
-> 매개변수를 순회하는 것이 목적이면 Iterable을 쓰자!

## 아이템 31 문서에 타입 정보 쓰지 않기

주석으로 타입 정보를 알려주지말고, ts 문법으로 타입 명시해라 (어차피 ts 쓴다면 보통 이렇게 하겠지만)
ts 문법을 이용해야지만 타입을 잘못 작성할 일이 줄어들고, 주석보다 오히려 더 보기 편하다.

변수나 속성의 단위가 명확하지 않다면, 변수며이나 속성명에 단위를 넣는 것도 괜찮다! (`temperatureC`,`temperatureF`)

## 아이템 32 타입 별칭에 null 또는 undefined 포함하지 않기

`type User = {~~} | null` 이런식으로 코드 짜면 안된다.

사람은 보통 User라는 별칭을 보고 User안에 있는 name, age같은 속성들에 대한 것만 떠올린다.
만약 User라는 타입 별칭에 null까지 있으면 이걸 사용하는 곳에서는 null도 받을 수 있다는 것을 알아채기 힘들다.

따라서 별칭에는 포함시키지 말고 이 타입을 사용하는 곳에서 `User | null` 이런식으로 쓰자

> 내가 가장 많이하는 실수 중 하나이기도 하다.

## 아이템 33 타입 주변에 null 값 배치하기

일단 주변에 null 값을 배치한다는게 뭔 말인가?
-> 바깥쪽 가장자리로 빼라는 말이다

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
      //             ~~~ Argument of type 'number | undefined' is not
      //                 assignable to parameter of type 'number'
    }
  }
  return [min, max];
}
```

우리는 직관적으로 min이 있으면, max도 무조건 있다는 것을 안다. 하지만 ts는 그걸 알 수 없다.

현재 min, max는 둘중하나만 undefined일수도, 하나만 undefined일수도, 모두 값이 존재할 수도 있다.

하지만 될 수 있는 것은 둘다 undefined거나 아니거나이다.

그래서 이걸 `let minMax: [number,number] | null = null` 이런식으로 표현하면 유효한 상태만 받을 수 있게 되고, 로직에서의 오류도 잡을 수 있게 된다.

또 하나 많이 하는 실수가 있는데, class를 사용할 때다.

값을 fetch한 뒤에 생성하지 않고, 클래스 생성할 때는 null로 초기화 하고, init 메서드로 따로 값을 채우는 방식은 좋지 안핟.

프로퍼티가 여러개 있고, 다 fetch로 값을 가져와야한다면, 특정 시점에따라 unll인 것과 채워진 것이 다 다르기 때문이다. 그래서 이를 사용하는 곳에서 null처리가 까다로워진다.

따라서 먼저 fetch로 값을 가져온 후에 안전한 값으로 인스턴스를 생성하는게 낫다.

> 이부분이 제목과 조금 더 직접적인 연관이 있다. 클래스 내부에서 null처리를 하지말고 외부로 null을 배치하라는 말이다.

## 아이템 34 유니온의 인터페이스보다는 인터페이스의 유니온 사용하기

유니온의 인터페이스 -> interface 안에 있는 속성이 유니온 타입인 경우

인터페이스의 유니온 -> interface | interface

사실 이 아이템에서도 이야기하는 것은 4장의 이전 아이템들과 크게 다르지 않다.

유효하지 않은 상태를 만들지 말라는 것!

중요한 건 인터페이스 안에 유니온 속성이 하나라도 있으면 나쁘다는 뜻이 아니다.

서로 관계가 있는 여러 속성을 각각 독립적인 유니온이나 선택적 속성으로 표현하는 경우에 문제가 있는 것이다.

> Q. 인터페이스의 유니온 사용하기의 대표적인 예시가 태그된 유니온인건가?
> A. 그렇다

태그된 유니온을 사용할 수 있다면 보통 사용하는 것이 좋다.

---

```ts
interface Person {
  name: string;
  // These will either both be present or not be present
  placeOfBirth?: string;
  dateOfBirth?: Date;
}
```

아이템 33에서 했던 것처럼

```ts
interface Person {
  name: string;
  birth?: {
    place: string;
    date: Date;
  };
}
```

이렇게 바꿀 수도 있지만, 만약 이게 API 응답이라서 구조를 바꾸지 못한다면

> 이 상황 실제로 오늘 백엔드 팀원이랑 API 설계할 때 있었음

`인터페이스의 유니온`을 사용해서 타입을 지정할 수 있음!

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

이런식으로 바꾸면 됨

> Q. 근데 이렇게 하면 타입의 안정성을 올리기 위해서 수많은 타입을 정의해야 될 수도 있을 것 같은데, 그럼 가독성이 너무 떨어지지 않나? 그리고 단순히 Name이라고 지을 일은 보통 없을 것 같은데 예제의 네이밍이 조금 애매하지는 않나?

## 아이템 35 string 타입보다 더 구체적인 타입 사용하기

> 이거 아주 나한테 유용한 아이템이다. 내가 보통 이런식으로 타입을 정의함

일단 무작정 string 하지말고, 문자 리터럴 타입의 유니온으로 할 수 있으면 하고, Date도 Date 객체로 하는 것이 좋다는 이야기다.

```ts
function pluck(records: any[], key: string): any[] {
  return records.map((r) => r[key]);
}
```

이 코드를 제네릭을 이용해서 개선하면

```ts
function pluck<T>(records: T[], key: keyof T) {
  return records.map((r) => r[key]);
}
```

이렇게 해볼 수 있다.

> 솔직히 내가 보기엔 이정도도 훌륭하다고 생각한다. 하지만 저자는 더 나은 방식이 있다고 언급한다.

```ts
function pluck<T, K extends keyof T>(records: T[], key: K): T[K][] {
  return records.map((r) => r[key]);
}
```

이렇게 하면 문제가 뭘까?

```ts
type RecordingType = "studio" | "live";

interface Album {
  artist: string;
  title: string;
  releaseDate: Date;
  recordingType: RecordingType;
}

function pluck<T>(records: T[], key: keyof T) {
  return records.map((r) => r[key]);
}

const albums: Album[] = [];

const dates = pluck(albums, "releaseDate");
const artists = pluck(albums, "artist");
```

여기서 dates와 artists는 타입 추론이 이상하게 된다.
(string | Date)[] 타입이 된다.

왜 추론이 넓게 될까? step by step으로 가보자

```ts
type AlbumKey = keyof Album;
// "artist" | "title" | "releaseDate" | "recordingType"
```

Album의 키는 이렇게 유니온 타입이 된다.
그렇기 때문에 value도 전체 유니온 타입이 되는 것이다.

value가 될 수 있는 타입은 아래와 같다.
`string | string | Date | RecordingType`

근데 RecordingType은 문자열 리터럴 타입이어서 string의 부분집합이므로 사라져서 최종적으로는 (string | Date)가 되는 것이다.

그래서 타입을 더 좁히기 위해서 제네릭을 하나 더 쓰는 방식을 사용한다.

```ts
function pluck<T, K extends keyof T>(records: T[], key: K): T[K][] {
  return records.map((r) => r[key]);
}
```

> 이부분 솔직히 아직 잘 모르겠음

```ts
const dates = pluck(albums, "releaseDate");
//    ^? const dates: Date[]
const artists = pluck(albums, "artist");
//    ^? const artists: string[]
const types = pluck(albums, "recordingType");
//    ^? const types: RecordingType[]
const mix = pluck(albums, Math.random() < 0.5 ? "releaseDate" : "artist");
//    ^? const mix: (string | Date)[]
const badDates = pluck(albums, "recordingDate");
//                             ~~~~~~~~~~~~~~~
// Argument of type '"recordingDate"' is not assignable to parameter of type ...
```

이렇게 하면 놀랍게도 모든 타입이 정확히 추론된다.

## 아이템 36 특수값에는 별도의 타입 사용하기

JS에서 -1은 특수한 값으로 쓰인다. 단순 숫자이지만, 그 의미를 지니기 때문에 특수한 타입으로 만들어 주어야한다.

양수만 들어올 수 있는 변수에 뭔가 특수한 값을 처리하기 위해 음수를 사용한다고 하면, 우리는 잘 받아들일 수 있지만 어디가에서 버그가 터질 수 있다.

따라서 이런 것들은 명확하게 null같은 값으로 처리를 해주어야한다.

네트워크 통신 같은 것도 지연됨, 오류같은 상태를 null이나 undefined로 처리하기보다는 태그된 유니온을 쓰는 것이 낫다. 아마도 모델링한다고 하면 이런식?

```ts
interface NetworkError {
  state: "error";
  // ...
}

interface NetworkDelayed {
  state: "delay";
  // ...
}
```

중요한건 null을 써라 쓰지말라가 아니다.
특수한 상황을 다뤄야 한다면 그 상황에 맞는 최선의 타입을 선택하라는 것이다.

없으면 없다고 null, 에러는 에러라고 표시하는 "error"처럼

## 아이템 37 선택적 속성 지양하기

Optional을 사용하면 문제가 있다.
바로 그 값이 없을 때의 처리를 값이 사용되는 곳마다 매번 처리를 해주어야 한다는 것이다.
특히 선택적 속성이 없을 때 기본값을 설정해놨을 때 실수로 선택적 속성을 적어주지 않은 곳에서는 의도하지 않은 동작이어도 그걸 찾기가 힘들다.

그래서 이 책에서 제시하는 방법은 어차피 기본값이 있다면, 최초에 정규화를 하라는 것이다.

```ts
interface InputAppConfig {
  darkMode: boolean;
  // ... other settings ...
  /** default is imperial */
  unitSystem?: UnitSystem;
}
interface AppConfig extends InputAppConfig {
  unitSystem: UnitSystem; // required
}
```

이렇게 처음부터 정규화를 해놓고 쓰면 사용하는 곳에서 기본값 처리를 해줄 필요가 없다.
이렇게 하는 것 대신에
`type AppConfig = Required<InputAppConfig>`로 할 수도 있다.

> Required<T>는 그 타입의 모든 속성을 required로 만들어줌. 기존에 optional인 것들도!

Optional을 그럼 대체 언제 써야하냐?

- 이미 존재하는 API를 모델링하거나 API를 수정할 때
- 실제로 선택적인 경우 (middleName 같은 경우)

## 아이템 38 같은 타입의 매개변수를 반복하지 않기

drawRect(25,50,75,100,1); 이 코드만 보고 얘가 뭘 하는지 알 수 있음? 절대 ㄴ

다같은 number타입이라 순서를 바꾸어 입력해도 타입 체크는 작동하지 않는다.

```ts
interface Point {
  x: number;
  y: number;
}
interface Dimension {
  width: number;
  height: number;
}
function drawRect(topLeft: Point, size: Dimension, opacity: number) {
  // ...
}
```

이렇게 완전히 분리시켜 받는다면 잘못된 값을 받기가 힘들어진다.

```ts
interface DrawRectParams extends Point, Dimension {
  opacity: number;
}
function drawRect(params: DrawRectParams) {
  /* ... */
}

drawRect({ x: 25, y: 50, width: 75, height: 100, opacity: 1.0 });
```

이렇게 1개의 객체로 만드는 것도 방법이다.

결론은 직관적이고 사용하기 쉬운 매개변수 타입을 설계하라는 것이다.
