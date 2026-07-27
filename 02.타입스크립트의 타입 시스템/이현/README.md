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

중요한건 속성의 교집합이 아님! 값들의 교집합이다.

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

## 아이템 8

심볼은 타입 공간 또는 값 공간에 존재한다.

이 아이템에서 말하는 심볼은 타입을 말하는 것이 아닌 것 같다..
"심볼"을 "단어"로 바꿔 이해하면 좋을듯 아직은

```ts
interface Cylinder {
  radius: number;
  height: number;
}

const Cylinder = (radius: number, height: number) => ({ radius, height });
```

JS에서는 같은 스코프 안에 이름이 똑같은 것을 두개 만들면 에러가 발생한다.
하지만 TS에서는 타입과 값이 각각 사는 공간이 다르기 때문에 이름이 같아도 충돌하지 않는다.

위쪽 Cylinder가 타입, 아래가 값이다.

가끔 `instanceof`를 TS 문법으로 생각하는 경우가 있는데 헷갈리지 말자 런타임에 체크하는 JS 문법이다.

---

**typeof 연산자의 2가지 용도**

typeof는 `값`을 읽어서 `타입`을 반환한다.

타입 공간의 typeof는 더 큰 타입의 일부분으로 사용할 수 있다, type 구문으로 이름을 붙이는 용도로도 사용 가능 -> 무슨말?

값에다가 typeof를 쓰면, js 런타임의 typeof연산자가 된다

속성 접근자인 []도 마찬가지

this도 값공간과 타입 공간에서 다르게 쓰인다.

여기서 하고싶은 말은 하나인 것 같다.
같은 연산자이더라도, TS와 JS가 받아들이는 동작은 다르다는 것! (typeof, [], this, &, | 등등..)

- 구조분해할당 안에서 타입을 정의하면 값의 관점에서 해석하므로 오류가 발생한다. 이때는 타입과 값을 구분해야한다.

---

## 아이템 9 타입 단언 대신 타입 선언 사용하기

TS에서 변수게 값을 할당하고 타입을 부여하는 방법 2가지

1. 타입 선언 -> :으로 타입 지정해주기
2. 타입 단언 -> as로 타입 지정해주기

타입 단언은 강제로 타입 지정 = 타입 체커에게 오류를 무시하라고 함
따라서 단언보다는 선언을 쓰는게 맞음

화살표함수 쓸 때 매개변수쪽에 반환타입 같이 붙여줘서 이 함수의 결괏값으로 뭐가 나오는지 잘 적어주자!

```ts
interface Person {
  name: string;
}

// ❌
const alice = { name: "Alice", age: 20 } as Person;

// ✅
const bob: Person = { name: "Bob", age: 20 };

// ✅
const people = ["alice", "bob", "jan"].map((name): Person => ({ name }));
```

타입 단언이 반드시 필요한 경우 -> 타입 체커가 추론한 타입ㅂ보다 우리가 판단하는 타입이 더 정확할 때 필요함
DOM 엘리먼트에 대해서는 TS보다 우리가 잘 알 것임

타입 단언을 사용할 때는 그 이유를 주석으로 같이 달아주는 것이 좋음

모든 상황에서 타입 단언을 쓸 수 있는 것이 아님! 대상 타입과 단언하려는 타입 사이에 인터섹션이 있어야지 단언할 수 있음 서로 포함관계가 아예없는데 단언하면 에러남

---

## 아이템 10 객체 래퍼 타입 피하기

JS에서는 string같은 기본형과 객체 타입을 서로 자유롭게 변환한다.
사실 string 자체적으로 charAt같은 메서드가 있는게 아니다. -> 아마 프로토타입 체이닝? 그거 같은데

String 객체는 자기 자신하고만 동일하다.

어떤 프로퍼티를 string같은 기본형에 할당하면 실행할때 그 속성은 사라지게 된다 -> 기본형에 속성을 넣는게 의미 없다는 뜻 (JS 내부 동작)

ts는 기본형과 객체 래퍼 타입을 별도로 모델링 한다
string - String
number - Number 등

**객체 래퍼 타입을 쓰지말고 기본형 타입을 쓰자.**

## 아이템 11 타입 체크와 잉여 속성 체크 구분해서 사용하기

타입이 명시된 변수에 객체 리터럴을 할당할 때 TS는 해당 타입의 속성이 있는지, 그리고 그 외의 속성은 없는지 확인함
-> 이게 잉여 속성 체크임

```ts
interface Room {
  numDoors: number;
  ceilingHeightFt: number;
}
const r: Room = {
  numDoors: 1,
  ceilingHeightFt: 10,
  elephant: "present",
  // ~~~~~~~ Object literal may only specify known properties,
  //         and 'elephant' does not exist in type 'Room'
};
```

```ts
const obj = {
  numDoors: 1,
  ceilingHeightFt: 10,
  elephant: "present",
};
const r: Room = obj; // OK
```

여기서 2번째 케이스에만 에러가 뜨지 않음. 첫 번째 코드에서만 잉여속성체크가 발생했음

```ts
interface Options {
  title: string;
  darkMode?: boolean;
}
function createWindow(options: Options) {
  if (options.darkMode) {
    setDarkMode();
  }
  // ...
}
createWindow({
  title: "Spider Solitaire",
  darkmode: true,
  // ~~~~~~~ Object literal may only specify known properties,
  //         but 'darkmode' does not exist in type 'Options'.
  //         Did you mean to write 'darkMode'?
});
```

이 코드는 런타임 에러가 없음. 하지만 개발자가 의도한대로 코드가 작동하지 않음. 오타를 내었기 때문임
구조적 타이핑에 의해서 에러가 발생하지 않기때문에 오히려 개발자가 더 헷갈리는 상황이 오게됨.

잉여속성체크가 적용되는 곳

1. 선언된 타입으로 변수를 직접 할당할 때
2. 함수의 매개변수 안에서
3. 함수의 반환값에서

객체 리터럴(중괄호를 이용해 객체를 직접 선언)이 아니면 잉여 속성 체크가 동작하지 않음

---

약한타입(모든 속성이 optional인 객체 타입)도 비슷하게 동작하는 구석이 있음

약한타입에 값을 할당할 때는 값이 가진 속성 중 최소 1개는 타입에 정의된 속성과 일치해야함.
이유는 약한타입은 구조적 타이핑 관점에서 그 어떤 타입이 들어와도 다 허용되는 타입임.
그래서 뭘 넣든 에러가 나지 않음 이건 개발자 관점에서 혼란을 야기할 수 있기 때문에
TS가 방어선을 구축해주는 느낌임

그럼 **잉여 속성 체크**와 다른점이 무엇이냐?
잉여 속성 체크는 객체 리터럴에만 적용이 되지만, 약한 타입에 적용되는 규칙은 할당방식에 제한이 없음. 리터럴인지 변수인지 상관없다는 뜻

---

잉여 속석 체크는 선택적 필드를 포함하는 Options같은 타입에 특히 유용하다
다만, 객체 리터럴에만 적용되고 적용 스코프도 매우 제한적이다.

일반적인 타입 체크 (구조적 타이핑): 필요한 조건만 최소한으로 있으면 나머지는 있어도 상관 없음
잉여 속성 체크 (객체 리터럴): 정의한 타입과 정확히 일치해야함

잉여 속성 체크는 개발자의 의도와 다르게 동작하는 것을 막아주기 위한 장치같은 것이다!

## 아이템 12 함수 표현식에 타입 적용하기

function 키워드를 쓰는 함수 문장과 const 변수에 함수를 담는 함수 표현식은 다르다.
그리고 TS에서는 함수 표현식을 사용하는게 좋다 -> 함수의 매개변수부터 반환값까지 전체를 함수 타입으로 선언해 표현식에 재사용할 수 있다는 장점이 있다.

공통 콜백 함수를 위한 타입 선언을 제공하는 것이 좋다 -> 라이브러리 만든다면

결론은 TS에서는 함수 표현식을 사용하자 다른 라이브러리에서 미리 정의된 함수 타입을 가져와서 쓸 때도 인자나 반환값들을 TS에게 자동추론 하도록 맡길 수 있다.

## 아이템 13 타입과 인터페이스의 차이점 이해하기

일반적으로는 interface 쓰자

함수 타입은 type 키워드 쓰는게 선호된다.

interface는 복잡한 상황에서 한계가 있다.

유니온 타입은 있지만, 유니온 인터페이스는 없다.

interface에는 "보강"이라는 개념이 있다.

한 파일 안에 동일한 이름의 interface가 정의된 경우에만 보강이 적용된다.
같은 이름으로 여러 interface를 선언하면, ts가 자체적으로 1개의 interface로 합쳐준다는 것이다 "보강"이다.

또 다른 차이점으로는 특정 스코프 내에서 발생한다.
type 별칭은 함수 내에서 타입이 정의되고, 그 타입의 값을 내보내도 문제가 발생하지 않는다. 말 그대로 별칭이기 때문에 크게 상관하지 않는다.

중요한 것은 interface이다. interface로 정의된 타입은 TS가 interface 이름으로 접근하려 한다.

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

이 코드에서 에러가 발생하는 이유는 bee는 Hummingbird라는 인터페이스이다.
d.ts (ts 빌드할 때 만들어지는 파일) 파일을 생성하도록 설정했을 때 문제가 발생하는데, Hummingbird interface는 getHummer 함수 스코프 안에서 정의되어 있는데 외부에서 접근하려고 하니까 오류가 발생하는 것이다.

AI의 말을 빌리자면,

```
타입스크립트 컴파일러가 .d.ts 파일을 작성하려고 보니, 외부에서 절대 접근할 수 없는(스코프에 갇힌) 이름을 공개 API(export된 함수)의 설명서에 적어야 하는 모순적인 상황을 발견하고 미리 차단(에러)하는 것"
```

**결론**

중요한 것은 일반적으로 interface 키워드를 작성하고, 복잡한 타입을 표현해야 될 때만 type을 쓰자는 것이다.

## 아이템 14 변경 관련 오류 방지를 위해 readonly 사용하기

자바스크립트의 기본형은 불변이다.
하지만 배열과 객체는 가변이다.
이 부분에서 TS의 readonly 접근자가 사용될 수 있다.

객체 전체를 readonly하고싶으면 Readonly<T>를 쓰면 된다.

주의사항 1. 기능이 얕게 동작한다.

내부 값이 변경될 수 있다는 말이다.

```ts
interface Outer {
  inner: {
    x: number;
  };
}
const obj: Readonly<Outer> = { inner: { x: 0 } };
obj.inner = { x: 1 };
obj.inner.x = 1; // OK
```

Outer 자체는 readonly이지만, 안에 존재하는 inner 객체는 적용되지 않아서 inner 객체의 값은 바꿀 수 있는 것이다.

만약 깊은 readonly를 사용하고 싶다면 `ts-essentials`에 있는 DeepReadonly 제네릭을 사용하도록 하자.

주의사항 2. 속성을 변경하는 메서드가 있으면, Readonly로 막을 수 없다.
date.setFullYear같이 직접 값을 변경하는 메서드가 있다면 readonly로 되어있어도 값이 바뀔 수 있다는 점이다.

---

number[](Array<T>)와 readonly number[](ReadonlyArray<T>)

ReadonlyArray<T>에는 pop, push, shift같이 값을 변경할 수 있는 메서드가 없음

Array<T>는 ReadonlyArray<T>의 서브타입임

readonly가 재할당을 방지하는 것은 아니다.
const는 한 번 선언하면 재할당을 할 수 없다.
하지만 readonly는 새로운 값으로의 재할당 자체는 할 수 있다.
단지 값의 변경이 안되는 것 뿐이다.

매개변수를 readonly로 바꿀 때의 문제점

- readonly가 매개변수인 함수 안에서 같은 매개변수로 또 다른 함수를 호출하면 그때도 readonly로 바꾸어야함

```ts
// 1. 일반 함수 (원본을 훼손할 가능성이 있음)
function mutateArray(arr: number[]) {
  arr.push(100);
}

// 2. 안전한 함수 (readonly 약속)
function safeFunction(arr: readonly number[]) {
  // ❌ 에러 발생!
  // "나는 원본 안 건드린다고 약속(readonly)했는데,
  // 그걸 훼손할지도 모르는 일반 함수(mutateArray)한테 넘겨줄 순 없어!"
  mutateArray(arr);
}
```

이를 해결하려면 어쩔 수 없이 타입 단언을 써야할 수도 있음

결론은 변하지 않는 값이라면 readonly를 써서 컴파일러와 개발자 모두에게 readonly라는 것을 알려주자!
= readonly 적극적으로 쓰자

## 아이템 15 타입 연산과 제네릭 사용으로 반복 줄이기

클린 코드를 위해 중복을 제거하는 것처럼 타입도 반복되면 공통으로 추출하자

**매핑된 타입**을 이용해 중복을 줄이는 방법

```ts
type TopNavState = {
  [K in "userId" | "pageTitle" | "recentFiles"]: State[K];
};
```

사실이건 Pick 유틸리티 타입과 똑같은 것임 Pick 내부적으로 이렇게 작동한다고 생각하면 된다.

```ts
type OptionsUpdate = { [k in keyof Options]?: Options[k] };
```

이건 Partial 유틸리티 타입이다.

(+ keyof는 타입을 받아서 속성 타입의 유니온을 반환한다.)

매핑된 타입과 as를 함께 이용하면 키와 값을 반전시키는 타입을 만들 수도 있다!

`K in keyof T`와 같은 형태로 매핑된 타입을 사용한다면 TS는 이를 `동형 매핑 타입`으로 다룬다.
동형 매핑 타입이란 readonly나 ?같은 접근자를 그대로 복사해준다.
하지만, keyof 없이 [K in 'name']처럼 직접 매핑을 하면 동형이 아니기 때문에 readonly같은 키워드들이 다 날아간다.
`Pick` 유틸리티 타입도 동형매핑 타입이다!

함수나 메서드의 반환값에 대한 타입을 만들도 싶으면 ReturnType<T>를 쓰면 된다.
이 제네릭에는 반환값을 얻고싶은 함수 그 자체를 쓰면 안되고, `typeof 함수명`을 써야한다!

덤으로 성급한 추상화 하지말자!!

```ts

```

## 아이템 16 인덱스 시그니처보다 정확한 타입 사용하기

```ts
[property: string]: string
```

이게 인덱스 시그니처

단점은 어떤 키를 정확히 가져야 하는지가 나타나지 않음 -> 자동완성 기능도 제공 x
그리고 모든 키가 string을 가져야함

따라서 정적이라면 interface로 정확한 타입을 지정해주는 것이 낫다.

그럼 동적 데이터라면 어떻게 할까?
key이름을 미리 알 수 있는 방법은 없기 때문에 이때 인덱스 시그니처를 사용한다.

열 이름을 알 수 있는 특정한 상황에서는

```ts
const products = parseCSV(csvData) as unknown[] as ProductRow[];
```

이런 방법을 쓰기도 한다. unknown으로 한번 거쳐가는 이유는 as를 쓰기 위해서임 (as도 intersection이 있어야 쓸수있기때문이라고 전에 공부햇음)

하지만 제일 좋은 방법은 `Map 타입`을 사용하는 것이다.
Map에서 get의 반환값은 항상 undefined를 가질 수 있다.

그리고 Map을 객체 타입으로 변환하는 과정을 거쳐서 객체를 만들면 된다.

이는 번거롭지만 넓게 정의된 타입에서 원하는 타입으로 구체적으로 만드는 일반적인 패턴이다.

필드의 개수가 제한되어 있는 경우에도 인덱스 시그니처를 쓰면 안된다!!!
-> optional or 유니온으로 하자

이것들이 번거롭다면 Record 제네릭을 사용해보자

---

인덱스 시그니처를 이용하면 잉여 속성 체크를 우회할 수 있다.

```ts
interface ButtonProps {
  title: string;
  onClick: () => void;
  [otherProps: string]: unknown;
}

renderAButton({
  title: "Roll the dice",
  onClick: () => alert(1 + Math.floor(20 * Math.random())),
  theme: "Solarized", // ok
});
```

이런 느낌

## 아이템 17 숫자 인덱스 시그니처 지양하기

JS에서 객체의 키는 보통 문자열이다.

배열이나 숫자같은 것을 key로 넣어도 결국 `toString` 메서드가 호출되어 객체를 문자열로 변환한다.
TS에서도 숫자 키를 허용하지만, 문자열 키와는 다른 것으로 인식함

`x['1']`같은 문법을 사용해도 값에 접근 가능하다.
어차피 런타임에서 JS가 자체적으로 숫자 1로 바꿔줘서 TS가 허용해주는 것이다.

하지만 아래와 같이 변수를 넣으면 안된다. 저 변수에 어떤 값이 올 수 있을지 모르기 때문이다.

```ts
const xN = xs[inputEl.value];
```

브라우저의 NodeList나 함수의 arguments처럼 "진짜 배열은 아닌데, 숫자 인덱스랑 length 속성만 가지고 있는 객체(유사 배열 객체)"를 다뤄야 할 때가 있다.

이때 함수 매개변수 타입으로 일반 배열([]) 대신 ArrayLike를 적어주면, 진짜 배열이든 유사 배열이든 다 받아줄 수 있게 된다.

배열 메서드(push, pop 등)는 필요 없고 순회만 할 건데, 배열처럼 생긴 애들은 다 들어와라고 할 때 쓰는 유연한 타입이다.
