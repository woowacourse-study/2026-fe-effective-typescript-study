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
