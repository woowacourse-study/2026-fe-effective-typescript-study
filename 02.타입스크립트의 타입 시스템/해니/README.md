### 아이템 6. 편집기를 사용해 타입 시스템 탐색하기

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

### 아이템 7. 타입을 값의 집합이라고 생각하기

타입스크립트의 타입은 **값들의 집합**이라고 생각하면 이해하기 쉽다.

예를 들어 `number` 타입은 `0`, `1`, `2` 등 숫자들의 집합이다. 기본 제공 타입뿐 아니라 유니온을 활용해 새로운 값의 집합을 만들 수도 있다.

#### 유니온(`|`) - 합집합

`AB` 타입은 `'A'`와 `'B'` 값의 모음이므로 둘 중 하나만 할당 가능하다.

```ts
type AB = "A" | "B";
```

#### 인터섹션(`&`) - 교집합

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

#### extends - 부분 집합

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

### 아이템 13. 타입과 인터페이스의 차이점 이해하기

타입스크립트에서 명명된 타입을 정의하는 방법은 `interface`와 `type` 두 가지가 있다.
대부분의 경우 둘 다 사용할 수 있으므로, **같은 상황에서는 동일한 방법으로 정의해 일관성을 유지**하는 것이 중요하다.

> 접두사로 `I` 또는 `T`를 붙이는 것은 지양하자.

#### 공통점

1. **함수 타입 정의** : JS에서 함수도 객체이므로 함수 타입은 `interface`와 `type` 둘 다로 정의할 수 있다.

```ts
type TFn = (x: number) => string;

interface IFn {
  (x: number): string;
}

type TFnAlt = {
  (x: number): string;
};
```

함수 타입은 타입 별칭(`type`)을 쓰는 것이 더 자연스럽고 간결하다.

2. **확장** : 서로 확장이 가능하다.

- `interface`는 `extends`
- `type`은 교차 타입 `&`
- 오류를 더 잘 찾아내는 `interface` + `extends` 조합이 권장된다.

3. **재귀** — 둘 다 재귀 타입을 표현할 수 있다.

```ts
interface TreeNode {
  value: string;
  children: TreeNode[];
}

type TreeNode = {
  value: string;
  children: TreeNode[];
};
```

#### 차이점

**interface는 불가능, type만 가능한 것**

- 원시 타입 정의
- 유니온 타입 (유니온 인터페이스는 없다)
- 튜플 정의

```ts
interface A {
  name: "A";
}
interface B {
  name: "B";
}

type AB = A | B;
type Name = AB["name"]; // 'A' | 'B'
```

동일한 키에 값이 다른 interface를 유니온으로 결합하면, 해당 키도 유니온으로 결합된다.

**type은 불가능, interface만 가능한 것**

같은 이름의 interface를 여러 번 선언하면 TS가 하나로 합쳐준다. 이를 **보강(declaration merging)** 이라 한다.

```ts
interface IState {
  name: string;
  capital: string;
}

interface IState {
  population: number;
}

const wyoming: IState = {
  name: "Wyoming",
  capital: "Cheyenne",
  population: 578_000,
};
```

- 하나의 프로젝트 안에서 남발하면 타입이 여러 곳에 흩어져 혼란을 준다.
- 주로 외부 라이브러리의 타입을 보강할 때 사용한다.

**스코프 차이 — `.d.ts` 생성 시 주의**

`type` 별칭은 함수 내부에서 정의한 뒤 그 값을 내보내도 문제가 없다.
반면 interface로 정의한 타입은 TS가 그 **이름으로 접근**하려 하기 때문에, 함수 스코프에 갇힌 interface를 외부로 노출하면 오류가 발생할 수 있다.

```ts
export function getHummer() {
  interface Hummingbird {
    name: string;
    weightGrams: number;
  }
  const bee: Hummingbird = { name: "Bee Hummingbird", weightGrams: 2.3 };
  return bee;
}
```

`.d.ts` 파일을 생성하도록 설정하면,
컴파일러가 외부에서 접근할 수 없는(스코프에 갇힌) 이름을 공개 API의 설명서에 적어야 하는 모순을 발견하고 미리 에러로 차단한다.

#### 결론

**일반적으로는 `interface`를 사용하고, 복잡한 타입(유니온, 튜플, 원시 타입 같은 것들)을 표현해야 할 때만 `type`을 쓰자.**

---

### 아이템 14. 변경 관련 오류 방지를 위해 readonly 사용하기

자바스크립트의 기본형은 불변이지만, **배열과 객체는 가변**이다. 여기서 TS의 `readonly` 접근자가 유용하다.

- `readonly` 속성으로 값 변경을 방지할 수 있다.
- 객체 전체를 막고 싶으면 `Readonly<T>` 제네릭 유틸리티 타입을 쓴다.

#### 주의사항

**1. 얕게(shallow) 동작한다.**

`Readonly<T>`는 속성에만 적용되고 내부 중첩 객체는 여전히 변경 가능하다.

```ts
interface Outer {
  inner: { x: number };
}

const obj: Readonly<Outer> = { inner: { x: 0 } };
obj.inner = { x: 1 }; // 에러 (inner 재할당 불가)
obj.inner.x = 1; // OK (내부 값은 변경 가능)
```

깊은 readonly가 필요하면 `ts-essentials`의 `DeepReadonly` 제네릭을 사용한다.

**2. 속성을 변경하는 메서드는 막지 못한다**

`date.setFullYear()`처럼 내부 값을 직접 바꾸는 메서드가 있으면 `readonly`여도 값이 바뀐다.

#### 배열 : `number[]` vs `readonly number[]`

- `readonly number[]`(`ReadonlyArray<T>`)에는 `pop`, `push`, `shift` 같은 변경 메서드가 없다.
- `Array<T>`는 `ReadonlyArray<T>`의 서브타입이다.

**readonly는 재할당을 막지 않는다.**
-> `const`는 변수 자체의 재할당을 막지만, `readonly`는 값의 변경을 막을 뿐 새로운 값으로의 재할당은 가능하다.

#### 함수 매개변수에 readonly 사용

```ts
function printNames(names: string[]) {
  names.push("Kim"); // 원본을 훼손할 수 있음
}
```

매개변수를 `readonly`로 받으면 **"이 함수는 배열을 읽기만 하고 변경하지 않는다"** 는 사실을 타입으로 표현할 수 있다.

- 함수 내부에서 매개변수 변경이 일어나는지 컴파일러가 체크한다.
- 호출부에 변경이 없음을 명시한다.
- 호출부에서는 readonly 배열/객체를 전달할 수 있게 된다.

**전파 문제** : readonly 매개변수를 받는 함수 안에서 그 값을 다른 함수에 넘기면, 그 함수도 readonly를 받아야 한다.

```ts
function mutateArray(arr: number[]) {
  arr.push(100);
}

function safeFunction(arr: readonly number[]) {
  mutateArray(arr); // 에러
  // readonly로 약속했는데, 훼손할지 모르는 일반 함수에 넘길 수 없다
}
```

해결하려면 어쩔 수 없이 타입 단언을 써야 할 수도 있다.

#### 결론

**변하지 않는 값이라면 `readonly`를 적극적으로 써서, 컴파일러와 개발자 모두에게 불변임을 알리자.**

---

### 아이템 15. 타입 연산과 제네릭 사용으로 반복 줄이기

클린 코드에서 중복을 제거하듯, **타입에서도 DRY 원칙**을 지켜야 한다.
타입 중복은 코드 중복만큼 많은 문제를 낳는다.

#### 이름 붙이기

가장 간단한 방법은 반복되는 타입에 이름을 붙이는 것이다.

```ts
interface Point2D {
  x: number;
  y: number;
}

function distance(a: Point2D, b: Point2D) {}
```

#### 확장으로 중복 제거

`interface`는 `extends`, `type`은 `&`로 공통 속성을 확장한다. 타입은 닫혀 있지 않아 더 유연하게 확장할 수 있다.

```ts
interface Person {
  firstName: string;
  lastName: string;
}

interface PersonWithBirth extends Person {
  birth: number;
}
```

#### 인덱스 접근 타입

```ts
interface State {
  userId: string;
  pageTitle: string;
  recentFiles: string[];
  pageContents: string;
}

interface TopNavState {
  userId: State["userId"];
  pageTitle: State["pageTitle"];
  recentFiles: State["recentFiles"];
}
```

#### 매핑된 타입과 유틸리티 타입

매핑된 타입으로 더 개선할 수 있고, 이것이 바로 `Pick`의 내부 동작이다.

```ts
type TopNavState = {
  [K in "userId" | "pageTitle" | "recentFiles"]: State[K];
};
```

- `Pick<T, K>` : 위 매핑된 타입과 동일
- `Partial` : 모든 속성을 선택적으로

```ts
type OptionsUpdate = { [k in keyof Options]?: Options[k] };
```

- `keyof`는 타입을 받아 속성 키들의 유니온을 반환한다.
- 매핑된 타입과 `as`를 함께 쓰면 키와 값을 반전시킬 수도 있다.

**동형 매핑 타입(homomorphic)**
`K in keyof T` 형태로 매핑하면 TS는 이를 동형 매핑 타입으로 다뤄 `readonly`, `?` 같은 접근자를 그대로 복사한다.
`keyof` 없이 `[K in 'name']`처럼 직접 매핑하면 동형이 아니라서 이런 키워드가 날아간다. `Pick`도 동형 매핑 타입이다.

#### 함수/제네릭 관련

**부분만 확장** : 부모의 모든 속성을 가져오지 않을 땐 `Pick`으로 일부만 확장한다.

```ts
interface Action {
  name: string;
  type: string;
  prop: string;
}

type SaveAction = Pick<Action, "name" | "type"> & {
  data: string;
};
```

**제네릭 매개변수 제한** : `extends`로 제네릭이 받을 수 있는 타입을 제한한다.

```ts
interface A {
  name: string;
}

type B<T extends A> = [T];

const b: B<{ name: string }> = [{ name: "B" }];
```

**제네릭 위치에 따른 차이** : 타입에 제네릭을 걸지, 함수 값에 걸지에 따라 다르다.

```ts
type A<T> = (param: T) => T; // 타입 선언 시 타입 결정
type B = <T>(param: T) => T; // 함수 호출 시 타입 결정

const a: A<string> = (param) => param;
const b: B = (param) => param;

const bReturn = b(1); // 호출 시점에 결정
const bReturn2 = b<string>("");
```

- `A`: 타입 자체에 제네릭 => 변수 선언 시 타입을 고정한다.
- `B`: 함수 값에 제네릭 => 호출할 때 타입을 결정해 더 유연하다.

**반환 타입 추출** : 함수의 반환 타입은 `ReturnType<T>`로 만든다. 이때 함수 자체가 아니라 `typeof 함수명`을 넣어야 한다.

#### 결론

**반복되는 타입은 이름 붙이기, 확장, 매핑된 타입, 유틸리티 타입(`Pick`, `Partial`, `ReturnType` 등)으로 줄이자. 단, 성급한(잘못된) 추상화보다는 중복이 낫다.**
