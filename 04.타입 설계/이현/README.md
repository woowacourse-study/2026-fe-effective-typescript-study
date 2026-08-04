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
