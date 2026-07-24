## 아이템 6. 편집기를 사용해 타입 시스템 탐색하기

타입스크립트를 사용하면 IDE를 통해 언어 서비스를 제공받을 수 있다.

예를 들어 아래처럼 변수를 선언하면 `num`이 `number` 타입이라는 것을 편집기가 자동으로 추론해 보여준다.

```ts
let num = 10; // number로 추론
```

언어 서비스를 통해 얻을 수 있는 것:

- 에러가 발생했을 때 무엇이 잘못되었는지 파악하기 쉽다.
- 값이 어떤 타입인지 시각적으로 확인 가능하다.
- 자동완성 기능을 제공해준다.

---

## 아이템 7. 타입을 값의 집합이라고 생각하기

타입스크립트의 타입은 **값들의 집합**이라고 생각하면 이해하기 쉽다.

예를 들어 `number` 타입은 `0`, `1`, `2` 등 숫자들의 집합이다. 기본 제공 타입뿐 아니라 유니온을 활용해 새로운 값의 집합을 만들 수도 있다.

### 유니온(`|`) - 합집합

`AB` 타입은 `'A'`와 `'B'` 값의 모음이므로 둘 중 하나만 할당 가능하다.

```ts
type AB = "A" | "B";
```

### 인터섹션(`&`) - 교집합

`&`는 두 타입의 교집합되는 값의 모음이다. `'A'`이면서 동시에 `'B'`인 값은 존재하지 않으므로 `never` 타입이 된다.

```ts
type AB = "A" & "B"; // never
```

`&`는 주로 **객체 타입을 연결**할 때 사용한다. 객체를 `&`로 연결하면 두 객체의 **모든 속성**을 가진 타입이 된다.

```ts
interface Person {
  name: string;
  address: string;
}

interface Tree {
  name: string;
  type: string;
}

type PersonAndTree = Person & Tree;

const a: PersonAndTree = {
  name: "이름",
  type: "나무",
  address: "서울',
};
```

### extends - 부분 집합

`extends`는 '~에 할당할 수 있는' 또는 '~의 부분 집합'이라는 의미로 받아들이면 쉽다.

아래에서 `Vector2D`는 `Vector1D`의 부분집합이다. `Vector1D`는 `x` 속성만 가지면 되지만, `Vector2D`는 `x`에 더해 `y`까지 가져야 하므로 더 좁은 집합이기 때문이다.

```ts
interface Vector1D {
  x: number;
}

interface Vector2D extends Vector1D {
  y: number;
}

interface Vector3D extends Vector2D {
  z: number;
}
```

---

### 아이템 8. 타입 공간과 값 공간의 심벌 구분하기

타입스크립트의 심벌은 **타입 공간**과 **값 공간** 중 한쪽(혹은 양쪽)에 속함.
두 공간은 서로 독립적이기 때문에, 같은 이름을 타입으로도 값으로도 선언할 수 있음. 다만 헷갈리기 쉬워 권장되지는 않음.

```ts
type Visualcamp = "SeeSo" | "RnD" | "Marketing" | ""; // 타입 공간
const Visualcamp = "Eye tracking"; // 값 공간
```

=> 이름은 같지만 서로 다른 곳에 존재하는 것임

- `interface`, `type` 으로 만든 것은 **타입 공간에만** 존재함. 컴파일 후 JS로 트랜스파일되면 흔적 없이 사라짐.
- 반대로 `const`, `let`, 함수 등은 **값 공간**에 존재하며 런타임까지 살아남음.

- `typeof`는 위치에 따라 의미가 다름
  같은 `typeof`라도 **타입 문맥**에서 쓰였는지 **값 문맥**에서 쓰였는지에 따라 완전히 다르게 동작함

```ts
interface Person {
  name: string;
  age: number;
  hobby: string;
}

const james: Person = {
  name: "James",
  age: 29,
  hobby: "Skateboard",
};

// 타입 문맥의 typeof -> 타입스크립트 타입을 돌려줌
type James = typeof james; // Person

// 값 문맥의 typeof -> 런타임에 문자열을 돌려주는 JS 연산자
const jamesType = typeof james; // "object"
```

즉,

- 타입 문맥의 `typeof`는 값의 타입을 얻는 도구이고
- 값 문맥의 `typeof`는 런타임에 `"object"`, `"string"` 같은 문자열을 반환하는 JS 연산자임

- `class`는 특이하게 **두 공간에 모두** 존재하는 예약어
  클래스는 내부적으로 생성자 함수이기 때문에 값으로도 쓸 수 있고, 동시에 인스턴스의 타입 역할도 함

- 타입 문맥에서 **클래스 이름** -> 인스턴스의 타입
- 타입 문맥에서 **`typeof 클래스`** -> 생성자(클래스 자체)의 타입
- 값 문맥에서 인스턴스에 `typeof`를 쓰면 -> 런타임엔 `object`

---

### 아이템 9. 타입 단언보다는 타입 선언을 사용하기

값에 타입을 부여하는 방법은 두 가지

```ts
interface Person {
  name: string;
}

const person1: Person = {}; // 타입 선언
const person2 = {} as Person; // 타입 단언
```

- **타입 선언(`: Person`)**: 값이 그 타입을 만족하는지 컴파일러가 검사함
- **타입 단언(`as Person`)**: 컴파일러가 추론한 타입을 무시하고 "이건 그냥 이 타입이야"라고 강제함

위 예시에서 `{}`는 `name`이 없으니 `Person`이 아님

그래서 **선언한 `person1`은 에러가 나지만, 단언한 `person2`는 에러가 나지 않는다.**
단언은 검사를 건너뛰기 때문에 잠재적 버그를 그대로 통과시킨다.
-> 이것만 봐도 기본값은 선언이어야 함.

|           | 속성이 안 맞아도 할당 가능 | 잉여 속성 체크 |
| --------- | :------------------------: | :------------: |
| 타입 단언 |             O              |       X        |
| 타입 선언 |             X              |       O        |

**단언도 아무 때나 되는 건 아이다!**
단언은 **포함 관계**(더 넓은 타입 ↔ 더 좁은 타입)일 때만 허용됨.
`{}`를 `Person`으로 단언할 수 있었던 건, `Person`이 `{}`보다 더 구체적인(좁은) 타입이기 때문임

서로 서브타입 관계가 아니면 단언할 수 없다.

```ts
interface Person {
  address: string;
  age: number;
}
interface Car {
  wheel: string;
  address: string;
}

const car: Car = { wheel: "바퀴", address: "주소" };
const person = car as Person; // 서로 서브타입이 아니라 단언 불가
```

-> `address`라는 공통 속성은 있지만 어느 쪽도 다른 쪽을 포함하지 않아 단언이 막힌다.

- 화살표 함수의 반환 타입은 단언 대신 선언으로
  `map` 콜백의 반환값에 타입을 주고 싶을 때도 단언보다 반환 타입 지정이 나음

```ts
["hello"].map((name): Person => ({ name }));
```

- 그래도 단언이 필요한 순간
  타입스크립트보다 내가 타입을 더 확실히 아는 경우엔 단언이 정당함

대표적으로:

- DOM 요소에 타입을 줄 때 (`document.querySelector(...) as HTMLInputElement`)
- 서버 응답처럼 컴파일러가 타입을 알 수 없는 값

---

### 아이템 10. 객체 래퍼 타입 피하기

원시 타입(`string`, `number`, ...)에는 각각 대응하는 **객체 래퍼 타입**(`String`, `Number`, ...)이 있음. 할당 방향은 한쪽으로만 열려 있음.

- 원시 타입 → 객체 래퍼 타입: **가능**
- 객체 래퍼 타입 → 원시 타입: **불가능**

```ts
const str1: String = new String("str1"); // 객체 래퍼
const str2: string = str1; // String을 string에 할당 불가

const str3 = "str3";
const str4: String = str3; // string은 String에 할당 가능
```

JS 자체에서도 `new String(...)` 같은 래퍼 생성은 지양함.
타입 표기 역시 항상 소문자 원시 타입(`string`, `number`, `boolean`)을 쓰는 것이 맞음.

---

### 아이템 11. 타입 체크와 잉여 속성 체크 구분해서 사용하기

객체 리터럴을 변수에 할당하거나 함수에 매개변수로 전달할 때 잉여 속성 체크가 수행된다.
잉여 속성 체크는 원래 구조적 타입 시스템을 보완하기 위한 추가검사이다. 그래서

```
interface User {
name: string;
}

const obj = {
name: "해니",
hobby: "축구",
};

const user: User = obj;
```

의 경우, User가 요구하는 name이 있네? 그럼 User처럼 사용할 수 있겠네라고 생각한다.

```
interface User {
name: string;
}

const user: User = {
nmae: "해니", // 오타
};
```

근데 이렇게 오타를 하면 오류를 못잡는다.
구조적 타입만 사용했다면 "nmae"가 그냥 추가 속성으로 취급될 수도 있어서 실수를 발견하기 어렵다.
그래서 ts가 객체 리터럴을 직접 넣는 경우에는 추가 속성이 있는지 한 번 더 검사하자! 라는 규칙을 만들었고, 그게 잉여 속성 체크이다.

잉여 속성 체크는 오류를 찾는 효과적인 방법이다. -> 그치만 타입스크립트 타입 체커가 수행하는 일반적인 구조적 할당 가능성 체크와 역할이 다르다.

```
const obj = {
name: "해니",
hobby: "축구",
};

const user: User = obj;
```

근데 이미 여기서 obj는 하나의 독립적인 객체 타입으로 추론 되었는데, ts는 이 객체를 다른 타입으로 사용할 수 있는지만 확인한다.

그러니까 user가 필요한 속성은 가지고 있는지 확인하지만, 그 외의 타입을 검사하지 않는다. 왜냐하면 다른 곳에서도 사용할 수 있는 일반 객체일 수 있기 때문이다.

잉여 속성 체크에는 한계가 있다. 임시 변수를 도입하면 잉여 속성 체크를 건너뛸 수 있다.

**핵심** : TypeScript는 구조적 타입 시스템을 사용하기 때문에 원래는 추가 속성이 있어도 허용함. 하지만 객체 리터럴을 직접 할당하는 경우에는 오타나 실수를 방지하기 위해 잉여 속성 체크를 추가로 수행함. 변수에 저장된 객체는 일반적인 구조적 타입 검사만 수행하므로 잉여 속성 체크는 적용되지 않음.

### 아이템 12. 함수 표현식에 타입 적용하기

타입 스크립트에서는 함수 표현식을 사용하는 것이 좋다.

```ts
function rollDice1(sides: number): number {} // 문장
const rollDice2 = function (sides: number): number {}; // 표현식
const rollDice3 = (sides: number): number => {}; // 표현식
```

함수의 매개변수부터 반환값까지 전체를 함수 타입으로 선언하여 함수 표현식에 재사용할 수 있는 장점이 있다.

```ts
type DiceRollFn = (sides: number) => number;
const rollDice: DiceRollFn = (sides) => {
  return 0;
};
```

반복되는 함수를 하나의 타입으로 통합할 수 있다.

```ts
type BinaryFn = (a: number, b: number) => number;
const add: BinaryFn = (a, b) => a + b;
const sub: BinaryFn = (a, b) => a - b;
const mul: BinaryFn = (a, b) => a * b;
const div: BinaryFn = (a, b) => a / b;
```

함수의 매개변수에 타입을 선언하는 것보다 함수 표현식 전체 타입을 정의하는 것이 코드도 간결하고 안전하다.

```ts
// Bad
async function checkedFetch(input: RequestInfo, init?: RequestInit) {
  const response = await fetch(input, init);
  if (!response.ok) {
    throw new Error("Request failed: " + response.status); //return으로 바뀌어도 오류를 잡아내지 못한다.
  }
  return response;
}

// Good
const checkedFetch: typeof fetch = async (input, init) => {
  const response = await fetch(input, init);
  if (!response.ok) {
    throw new Error("Request failed: " + response.status); // return으로 바뀌면 오류를 잡아낸다.
  }
  return response;
};
```

다른 함수의 시그니처를 사용하기 위해 typeof fn을 사용한다.
