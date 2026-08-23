### 아이템 50. 제네릭이 타입 간 함수라고 생각하기

값 영역에서 반복 줄이는 게 함수, 타입 영역에서 그 역할 하는 게 제네릭
-> 그냥 타입계 함수라고 보면 됨.

```ts
type MyPartial<T> = { [K in keyof T]?: T[K] };
//        ^함수명  ^매개변수    ^본문
type Result = MyPartial<Person>; // 호출
```

`T`는 매개변수, `MyPartial<Person>`은 호출, 나온 타입이 반환값

중요한 건 순수 함수라는 점이다.
같은 입력엔 항상 같은 출력, 부수 효과 없음 -> 하나씩 짚어보는 디버깅이 통함

#### extends는 상속이 아니라 매개변수 타입 애노테이션

```ts
type MyPick<T extends object, K extends keyof T> = { [P in K]: T[P] };
```

키워드 때문에 상속처럼 읽히는데 하는 일은 `function f(x: number)`의 `: number`랑 똑같음
-> 받을 수 있는 범위를 좁히는 것! 여기선 T가 객체, K가 그 객체의 키

제약 안 걸면 `T[P]`가 성립 안 해서 에러가 제네릭 정의부 안쪽에서 터진다.
제약 걸면 호출 지점에서 터짐. 후자가 나음.
-> 잘못 쓴 사람이 잘못 쓴 자리에서 빨간 줄 보는 게 맞으니까

#### 유니온 넣으면 쪼개짐

일반 함수랑 갈라지는 지점이 여기인 것 같다.
유니온 넣으면 인자 하나가 아니라 여러 번 호출된 것처럼 동작함.

```ts
type Elem<T> = T extends (infer U)[] ? U : never;

type A = Elem<string[] | number[]>; // string | number
// 사실상 Elem<string[]> | Elem<number[]> 로 분배됨
```

`{ [K in keyof T] }` 같은 동형 매핑 타입도 마찬가지 -> `Partial<A | B>`는 `Partial<A> | Partial<B>`
분배 막고 싶으면 `[T] extends [any[]]`처럼 대괄호로 감싸면 됨. (자세한 건 아이템 53)

제네릭 새로 만들 땐 유니온, `never`, `any` 세 개 넣어보고 결과 확인하는 습관 들이면 좋을 듯 하다.
특히 `never`는 유니온의 빈 집합이라 분배 결과가 통째로 증발하는 경우 많음

#### 타입 세계의 as any

값 영역에서 `as any`로 에러 뭉개듯, `K & keyof T` 같은 인터섹션을 "에러 사라지니까" 붙이는 순간 그건 골치 아프게 된다.
타입 검사 통과했다는 게 정확성을 보증 못 하게 된다.
원인은 대부분 제약이 없어서니까 -> 인터섹션 말고 `K extends keyof T`로 시그니처를 고치는 게 맞는 수순이다.
