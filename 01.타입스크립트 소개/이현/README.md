- ### 아이템 4 - 구조적 타이핑에 익숙해지기

  JS는 덕 타이핑이고, 타입스크립트도 이를 그대로 모델링한다.

  ```js
   interface Vector2D {
     x: number;
     y: number;
   }
  ```

  이러한 Vector2D 인터페이스가 존재하고,

  ```js
   function calculateLength(v: Vector2D) {
     return Math.sqrt(v.x ** 2 + v.y ** 2);
   }
  ```

  길이를 계산하는 함수가 있을 때,

  ```js
   interface NamedVector {
     name: string;
     x: number;
     y: number;
   }
  ```

  name이 들어간 벡터를 만들고, calculateLength 메서드의 파라미터로 NamedVector 인스턴스를 만들어 넣어주면 잘 작동한다.
  이것이 바로 덕 타이핑이다.

  x와 y가 있기 때문에 문제없이 작동하는 것이다. 하지만 우리는 Vector2D와 NamedVector와의 관계를 전혀 선언하지 않았다.
  JS의 덕타이핑을 모방한 것이 TS의 구조적 타이핑이다.

  차이점이라면 덕타이핑은 JS의 런타임에 작동한다는 것?

  `타입은 열려있다!!`
  -> ❗️ 함수의 매개변수 타입을 지정할 때 정확히 그 타입만 가지는 매개변수가 들어올 것이라고 생각하지 말자

  ````js
   interface Vector3D {
     x: number;
     y: number;
     z: number;
   }

   function calculateLengthL1(v: Vector3D) {
     let length = 0;
     for (const axis of Object.keys(v)) {
       const coord = v[axis];
       //            ~~~~~~~ Element implicitly has an 'any' type because ...
       //                    'string' can't be used to index type 'Vector3D'
       length += Math.abs(coord);
     }
     return length;
   }

   // 왜 여기서 오류가 날까? calculateLengthL1의 매개변수로 오는 v가 x,y,z만 가지고 있을 것이라고 함부로 판단할 수 없다.

   const vec3D = {x: 3, y: 4, z: 1, address: '123 Broadway'};
   calculateLengthL1(vec3D);  // OK, returns NaN

   // 이것도 작동할 것이기 때문에 calculateLengthL1의 v[axis] 구문에서 coord는 number라는 값을 보장받지 못하는 것이다.
   ```
  ````

  구조적 타이핑은 클래스 개념에도 적용된다. num이라는 인스턴스를 가지고, 생성자가 있는 클래스를 받는 자리에 num이라는 값을 가지는 객체가 들어갈 수 있는 이유도 구조적 타이핑 덕분이다.

  구조적 타이핑은 테스트에 도움이 된다. 인터페이스만 작성해놓으면, 나머지 세부구현은 구조적 타이핑으로 인해 어떻게 구현해도 상관 없다.

### 아이템 5 any 타입 지양하기

any 타입을 사용하면

- 런타임에 에러가 생길 수 있다
- 자동완성이 제공되지 않는다
- any 타입은 쓰지 말자

### 아이템 6

ts 설치하면 2가지 실행 가능

1. tsc
2. ts server -> 자동완성, 명세, 검사, 검색, 리팩터링 포함

편집기(vscode)에서 마우스 갖다대면 타입 추론 어떻게 되는지 확인할 수 있고, 추론이 다르면 직접 타입을 지정해주자 -> 타입 좁히기와 넓히기 할 때 유용함

fn + f2를 이용하면 해당하는 내가 원하는 변수명만 바꿀 수 있다 -> 동일한 맥락의 변수명만 골라서 바꿈

많은 타입을 탐색해보자. 그러면 어떻게 타입 오류가 나타나는건지 더 잘 이해할 수 있을 것이다.

### 아이템 7 타입을 값의 집합이라고 생각하기

타입 = 값의 집합

가장 작은 집합은 아무 값도 포함되지 않는 공집함 = never 타입
never에는 아무 값도 할당할 수 없음!

Q. never는 그럼 왜 존재하는거임?

never 다음으로 작은 집합은 리터럴 타입이다.
'A', 'B'같은거

TS 경고 문구를 보면 ~는 ~에 할당할 수 없습니다. 이런걸 많이 보게 되는데 이걸 집합으로 보면됨

---

**인터섹션(교집합)**

```ts
interface Person {
  name: string;
}

interface Lifespan {
  birth: Date;
  death?: Date;
}
```

중요한건 속성의 교집합이 아님! 값의 교집합이다.

Person에는 이런 값들의 집합이 올 수 있다.

```md
{ name: ... }
{ name: ..., age: ... }
...
```

LifeSpan에는 이런 값들의 집합이 올 수 있다.

```md
{ birth: ... }
{ birth: ..., death: ... }
...
```

구조적 타이핑에 의해서
{ name: ..., birth: ..., death:... }는 Person과 Lifespan 둘 다 될 수 있다. 그리고 이 집합이 바로 Person과 Lifespan 값의 교집합이다.

## **결론은 교집합은 겉으로 보기에는 해당 속성들을 모두 가지고 있는 것처럼 보이는 것이다!**

---

**keyof**

```ts
type K = keyof (Person | Lifespan);
// ^? type K = never
```

여기는 런타임에 Person이 올 수도 있고, Lifespan이 올 수도 있어서 지금 당장 어떤 key를 가져올 수 있는지 모름 그래서 keyof는 런타임 전에 당장 가져올 수 있는 것만 가져옴
![alt text](image.png)

---

**extends**

extends는 보통 필드를 추가할 때 사용한다.
하지만 이미 존재하는 필드의 부분집합을 만들 때에도 사용된다. (타입 범위를 좁히는 느낌)

K extends string을 K가 string을 상속받는다는 개념으로 접근하면 이해하기가 어렵다!

하지만 K가 string의 부분집합이라고 해석하면 그 의미를 의해하기가 더 쉬워진다.

```ts
function getKey<K extends string>(val: any, key: K) {
  // ...
}

getKey({}, "x"); // OK, 'x' extends string
getKey({}, Math.random() < 0.5 ? "a" : "b"); // OK, 'a'|'b' extends string
getKey({}, document.title); // OK, string extends string
getKey({}, 12);
//
```

12를 제외한 나머지는 string의 부분집합이라서 에러가 뜨지 않는다.

---

**튜플**

튜플 타입이 배열 타입의 하위집합이다.
그리고 튜플 타입은 length를 모델링하기 때문에 length가 다른 튜플을 서로 집어넣으려고 하면 에러가 난다.

타입스크립트 타입이 되지 못하는 값의 집합들이 있다.
정수에 대한 타입 등
