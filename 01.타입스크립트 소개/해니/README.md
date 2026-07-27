### **아이템 1. 타입스크립트와 자바스크립트의 관계 이해하기**

**핵심: 타입스크립트는 자바스크립트의 상위 집합이다.**

- 모든 유효한 자바스크립트는 유효한 타입스크립트 (.js → .ts 확장자만 바꿔도 동작)
- 단, “타입 체크를 통과하는 자바스크립트”는 더 좁은 집합
- 문법 오류가 있는 코드는 둘 다에 속하지 않음

**타입 구문 없이도 오류를 잡는다**

- 존재하지 않는 프로퍼티 접근, 배열 메서드 오용 등을 추론으로 감지
- 타입을 명시하면 의도를 알려줄 수 있어 오류 원인을 더 정확히 짚어냄

**주의**

- 타입 체크를 통과해도 런타임 오류는 발생할 수 있다
- 타입 시스템은 런타임 동작을 모델링하지만 완벽하지 않다

### **아이템 2. 타입스크립트 설정 이해하기**

**설정은 tsconfig.json으로 관리한다**

- 커맨드 라인 플래그보다 파일로 관리 → 협업자와 도구가 컴파일 방식을 알 수 있음
- tsc --init으로 생성

**noImplicitAny**

- 타입을 명시하지 않은 변수가 암묵적으로 any가 되는 것을 방지
- any를 허용하면 타입 체커가 사실상 무력해짐
- 새 프로젝트라면 반드시 켤 것
- JS 마이그레이션 중일 때만 임시로 끄는 것 고려

**strictNullChecks**

- null, undefined가 모든 타입에 허용되는지 결정
- 켜면 “undefined는 객체가 아닙니다” 류의 런타임 오류를 컴파일 단계에서 포착
- 코드가 다소 번거로워짐

- 마이그레이션 순서: noImplicitAny → strictNullChecks
  - **왜 이 순서인가?**  
    순서를 뒤집으면 최악이다. 타입 표기가 없는 상태에서 strictNullChecks를 켜면, 아직 any인 값들이 null 검사를 그냥 통과해버려서 에러가 안 잡힌다. 나중에 noImplicitAny로 타입을 붙이는 순간 숨어있던 null 에러가 한꺼번에 터져나온다. 즉 일을 두 번 하게 된다. 반대로 이 순서면 각 단계가 이전 단계 위에 쌓인다. 타입이 먼저 다 붙어 있어야 null 분석이 의미 있는 결과를 낸다.

**strict**

- 위 설정들을 한 번에 적용

### **아이템 3. 코드 생성과 타입이 관계없음을 이해하기**

**컴파일러의 두 가지 독립적인 역할**

1. 최신 TS/JS → 구버전 JS로 트랜스파일
2. 타입 오류 체크

이 둘은 서로 완전히 독립적이다. 여기서 다음 결론이 따라온다.

런타임에 하는 게 아니다.

| **결론**                          | **설명**                                     |
| --------------------------------- | -------------------------------------------- |
| 타입 오류가 있어도 컴파일된다     | 오류는 경고에 가까움. 막으려면 noEmitOnError |
| 런타임에는 타입 체크가 불가능하다 | 타입 구문은 컴파일 시 전부 제거됨            |
| 타입 연산은 런타임에 영향이 없다  | as number는 값을 바꾸지 않음                 |
| 런타임 타입 ≠ 선언된 타입         | API 응답 등 외부 값은 선언과 어긋날 수 있음  |
| 타입으로 함수 오버로드 불가       | 시그니처는 여러 개, 구현체는 하나            |
| 타입은 런타임 성능에 영향 없음    | 빌드 타임 오버헤드만 존재                    |

**런타임에 타입 정보가 필요하다면?**

- 프로퍼티 존재 여부로 확인
- 태그 기법 사용 (kind: 'rectangle' 같은 필드)
- 타입과 값을 모두 만들어내는 클래스 사용

**타입 체크 통과 = 런타임 안전 아님**
타입 시스템은 런타임 동작의 모델일 뿐 보증서가 아니다. 특히 API 응답처럼 외부에서 들어오는 값은 선언된 타입과 얼마든지 달라질 수 있다.

**as는 변환이 아니라 주장이다.**
as number는 값을 숫자로 바꾸지 않고 “숫자라고 치자”라고 말할 뿐이다. 실제 변환은 Number() 같은 JS 연산이 필요하다.

**any는 편의가 아니라 타입 체커의 스위치를 끄는 행위.**
noImplicitAny를 켜야 하는 이유가 여기 있다.

### **새로운 것**

- **noEmitOnError** : 타입 오류가 있으면 출력 자체를 막는 설정. 컴파일이 그냥 되어버린다는 게 낯설다면 이걸 켜두면 된다.

- **태그 기법(tagged union)** : 런타임 타입 판별이 필요할 때 kind: 'rectangle' 같은 필드를 두는 패턴. 인터페이스는 instanceof가 안 되지만 클래스는 타입과 값을 동시에 만들어내므로 가능하다.

- **마이그레이션 순서가 있다** : noImplicitAny 먼저, strictNullChecks 나중. 둘을 동시에 켜면 오류 폭탄을 맞는다.

- 컴파일러와 타입체크는 독립적으로 이루어진다.

---

### 아이템 4. 구조적 타이핑에 익숙해지기

- **덕 타이핑(duck typing)**: 객체가 어떤 타입에 부합하는 변수, 메서드를 가지면 그 타입에 속하는 것으로 간주하는 방식. "오리처럼 걷고, 헤엄치고, 꽥꽥거리면 오리다"에서 유래.

- **구조적 타이핑(structural typing)**: 타입 구조가 유사하면 별도 선언 없이 두 타입이 서로 호환되는 것.

- JS가 덕 타이핑 기반이고 TS는 JS의 런타임 동작을 모델링하므로, TS도 구조적 타이핑을 따른다.

```ts
interface Vector2D {
  x: number;
  y: number;
}

function calculateLength(v: Vector2D) {
  return Math.sqrt(v.x * v.x + v.y * v.y);
}

interface NamedVector {
  name: string;
  x: number;
  y: number;
}

const v: NamedVector = { x: 3, y: 4, name: "Zee" };

console.log(calculateLength(v)); // 5
```

`Vector2D`와 `NamedVector` 사이에 아무 관계도 선언하지 않았지만 호환된다.

- `NamedVector` 타입의 값도 문제없이 사용 가능
- `NamedVector`용 별도 함수를 정의할 필요가 없음 (재사용성 ↑)

#### 타입은 "열려(open)" 있음

타입에 선언된 속성 외에 임의의 속성이 추가로 있어도 오류가 발생하지 않는다. 이것이 구조적 타이핑의 장점이자, 아래 문제의 원인이다.

#### 문제: 객체 순회

```ts
interface Vector3D {
  x: number;
  y: number;
  z: number;
}

function calculateLength1(v: Vector3D) {
  let length = 0;
  for (const axis of Object.keys(v)) {
    const coord = v[axis];
    // ~~~~~~~~ 'string'은 'Vector3D'의 인덱스로 사용할 수 없기에
    //          엘리먼트는 암시적으로 'any' 타입입니다.
    length += Math.abs(coord);
  }
  return length;
}
```

`axis`는 `"x" | "y" | "z"` 중 하나이고 `coord`는 `number`일 것으로 기대하지만, **타입스크립트가 오류를 정확히 찾아낸 것이 맞다.**

```ts
const example = {
  x: 3,
  y: 4,
  z: 5,
  text: "텍스트입니다",
};

calculateLength1(example); // 타입 에러 발생하지 않음
```

타입이 열려 있어 `text` 같은 추가 속성을 가진 객체도 인자가 될 수 있다. 따라서

- `v`는 어떤 속성이든 가질 수 있으므로 `axis`의 타입은 `string`이 될 수 있고
- `coord`는 `any`가 된다.
- 즉, 정확한 타입으로 객체를 순회하는 것은 까다롭다.

#### 해결

속성 수가 많지 않다면 루프 대신 각 속성에 직접 접근한다.

```ts
function calculateLength1(v: Vector3D) {
  return Math.abs(v.x) + Math.abs(v.y) + Math.abs(v.z);
}
```

---

### 아이템 5. any 타입 지양하기

타입 선언에 시간을 쓰기 싫어 `any`나 타입 단언문(`as any`)을 쓰고 싶어지지만, 일부 특별한 경우를 제외하면 `any`를 쓰는 순간 타입스크립트의 장점 대부분을 잃는다. 부득이하게 쓰더라도 그 위험성을 알고 있어야 한다.

#### any의 문제점

| 문제                                  | 설명                                                                                             |
| ------------------------------------- | ------------------------------------------------------------------------------------------------ |
| **타입 안전성이 없다**                | 타입 체커에게 "체크하지 말라"고 지시하는 것과 같아 안전성을 보장받지 못한다.                     |
| **함수 시그니처를 무시한다**          | 호출부는 약속된 타입을 넘기고 함수는 약속된 타입을 반환해야 하는데, `any`는 이 약속을 무시한다.  |
| **언어 서비스가 제공되지 않는다**     | 자동완성, 도움말, 이름 변경(rename) 등 TS 경험의 핵심 요소를 누리지 못해 생산성이 떨어진다.      |
| **버그를 감춘다**                     | 없는 속성에 접근해도 타입 체커는 통과시키고, 런타임에서야 에러가 난다.                           |
| **타입 설계를 감춘다**                | 특히 객체 타입에서 설계 의도가 불분명해진다. 설계가 드러나도록 타입을 일일이 작성하는 편이 좋다. |
| **타입 시스템의 신뢰도를 떨어뜨린다** | 런타임에 타입 오류가 발견되면 타입 체커를 신뢰할 수 없게 된다.                                   |

#### 요약

`any` 타입은 타입 체커와 언어 서비스를 무력화시키고, 진짜 문제점을 감추며, 개발 경험과 타입 시스템의 신뢰도를 떨어뜨린다. 최대한 사용을 피하자.

## 아이템 13. 타입과 인터페이스의 차이점 이해하기

타입스크립트에서 명명된 타입을 정의하는 방법은 `interface`와 `type` 두 가지가 있다.
대부분의 경우 둘 다 사용할 수 있으므로, **같은 상황에서는 동일한 방법으로 정의해 일관성을 유지**하는 것이 중요하다.

> 접두사로 `I` 또는 `T`를 붙이는 것은 지양하자.

### 공통점

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

### 차이점

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

### 결론

**일반적으로는 `interface`를 사용하고, 복잡한 타입(유니온, 튜플, 원시 타입 같은 것들)을 표현해야 할 때만 `type`을 쓰자.**

---

## 아이템 14. 변경 관련 오류 방지를 위해 readonly 사용하기

자바스크립트의 기본형은 불변이지만, **배열과 객체는 가변**이다. 여기서 TS의 `readonly` 접근자가 유용하다.

- `readonly` 속성으로 값 변경을 방지할 수 있다.
- 객체 전체를 막고 싶으면 `Readonly<T>` 제네릭 유틸리티 타입을 쓴다.

### 주의사항

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

### 배열 : `number[]` vs `readonly number[]`

- `readonly number[]`(`ReadonlyArray<T>`)에는 `pop`, `push`, `shift` 같은 변경 메서드가 없다.
- `Array<T>`는 `ReadonlyArray<T>`의 서브타입이다.

**readonly는 재할당을 막지 않는다.**
-> `const`는 변수 자체의 재할당을 막지만, `readonly`는 값의 변경을 막을 뿐 새로운 값으로의 재할당은 가능하다.

### 함수 매개변수에 readonly 사용

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

### 결론

**변하지 않는 값이라면 `readonly`를 적극적으로 써서, 컴파일러와 개발자 모두에게 불변임을 알리자.**

---

## 아이템 15. 타입 연산과 제네릭 사용으로 반복 줄이기

클린 코드에서 중복을 제거하듯, **타입에서도 DRY 원칙**을 지켜야 한다.
타입 중복은 코드 중복만큼 많은 문제를 낳는다.

### 이름 붙이기

가장 간단한 방법은 반복되는 타입에 이름을 붙이는 것이다.

```ts
interface Point2D {
  x: number;
  y: number;
}

function distance(a: Point2D, b: Point2D) {}
```

### 확장으로 중복 제거

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

### 인덱스 접근 타입

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

### 매핑된 타입과 유틸리티 타입

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

### 함수/제네릭 관련

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

### 결론

**반복되는 타입은 이름 붙이기, 확장, 매핑된 타입, 유틸리티 타입(`Pick`, `Partial`, `ReturnType` 등)으로 줄이자. 단, 성급한(잘못된) 추상화보다는 중복이 낫다.**
