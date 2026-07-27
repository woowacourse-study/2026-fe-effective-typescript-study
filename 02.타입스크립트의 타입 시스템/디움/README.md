## 아이템 6

### 타입스크립트 서버는 자동으로 켜지는 것일까?
-> VS Code 자체에 TypeScript가 내장되어 있어서 자동으로 켜진다.

VS Code 설치
→ 내장 TypeScript 언어 서비스가 있음
→ .ts/.tsx 파일을 열면 tsserver 자동 실행

프로젝트에 typescript 설치
→ 프로젝트에 맞는 TypeScript 버전 사용 가능
→ tsc 명령어와 빌드·타입 검사에도 사용

## 아이템 7

**할당 가능한 값의 집합 = 타입**

### & 연산자
& 연산자는 두 타입의 교집합을 계산한다.
이때 타입의 교집합은 **속성 이름의 교집합**이 아니라, 그 타입에 할당 가능한 **값의 교집합**이다.

Q. 구조적 타이핑의 원리가 들어간걸까?

A. `&` 자체가 구조적 타이핑의 원리를 따르는 연산자라기보다, “두 타입을 동시에 만족”시키는 연산자이고, 객체 타입에 적용될 때 TypeScript의 구조적 타이핑 때문에 속성이 합쳐져 보이는 것이다.

객체 타입에서는 타입 이름보다 **값이 어떤 구조를 가졌는지**를 기준으로 판단한다.
즉,  `A & B`는 구조적 타이핑에 따라 **A의 구조와 B의 구조를 모두 만족하는 타입**이다.

``` ts
interface Person {
	name: string;
}

interface Lifespan{
	birth: Date;
	death?: Date;
}

type PersonSpan = Person & Lifespan;
```

당연히 `name, birth, death`  세 가지보다 더 많은 속성을 가지는 값도 `PersonSpan` 타입에 속한다. 

-> TS는 구조적 타이핑을 따르기 때문

### 템플릿 리터럴 타입
문자열을 조합해서 새로운 문자열 타입을 만드는 기능

-> 자바스크립트의 템플릿 리터럴이랑 모양이 비슷하다

``` ts
type EventName = `on${string}`;

const a: EventName = 'onClick'; // 가능
const b: EventName = 'onChange'; // 가능
const c: EventName = 'click'; // 오류
```

### 핵심
- 타입은 값의 집합이다.
- & 연산자는 구조적 타이핑의 원리를 생각해보면 결과를 이해할 수 있다.

### 타입 계층도
<img width="780" height="270" alt="image" src="https://github.com/user-attachments/assets/c0aa6478-fba0-42c0-b262-ff2b824c95d8" />


## 아이템 8

### `symbol` 타입

- **절대로 다른 값과 우연히 같아지지 않는 고유한 값**을 만들기 위한 자바스크립트의 원시 타입
- 주로 객체의 안전한 프로퍼티 키를 만들 때 사용

``` ts
const a = Symbol();
const b = Symbol();

console.log(a === b); // false
```

### `unique symbol` 타입

- 특정 심벌 하나만 나타내는 타입

``` ts
const ID: unique symbol = Symbol('id');
```

타입스크립트의 심벌은 **타입 공간** 또는 **값 공간**에 존재한다.
- 타입 선언(:) 또는 단언문(as) 다음에 나오는 심벌은 타입
- (=) 다음에 나오는 것은 값

class와 enum은 타입과 값 두 가지 모두 가능하다.

`enum`
- TypeScript 문법
- 관련 있는 여러 상수 값을 하나의 이름 아래 묶어 관리하는 기능

타입의 속성을 얻을 때에는 반드시 `obj['field']`를 사용해야 한다.

``` ts
interface Person{
	name: string;
	age: number;
}

const pName: Person.name = '1'; // 오류
```

### 다형성 this

- 클래스나 인터페이스 안에서 `this`를 타입으로 쓰면, **현재 클래스 자체로 고정되지 않고 실제로 호출한 하위 클래스의 타입을 가리키는 기능**

``` ts
class Builder {
  setName(name: string): this {
    return this;
  }
}

class UserBuilder extends Builder {
  setAge(age: number): this {
    return this;
  }
}

const builder = new UserBuilder();

builder
  .setName('이름')
  .setAge(20);
```

- `Builder의` 하위 클래스인 `UserBuilder` 인스턴스에서 호출하면 `setName`의 반환 타입인 `this`는 `UserBuilder`로 해석된다.
- 따라서 `setName을` 호출한 뒤에도 `setAge`를 이어서 호출할 수 있다.

### 구조 분해 할당에서 `:`는 이름 바꾸기로 해석된다.

``` ts
function email({ to: Person, subject, body }) {
  // ...
}
```

`to`의 타입을 `Person`으로 지정한다는 의도가 담겨있지만, 실제로 그렇지 동작하지 않는다.

`{ to: Person}`은 객체의 `to`프로퍼티 값을 `Person`이라는 지역 변수에 저장한다.

``` js
function email(arg) {
  const Person = arg.to;
}
```

이 코드와 같은 의미이다.

## 아이템 9

### 타입 단언 대신 타입 선언을 사용하자

- 타입 선언은 그 값이 선언된 타입임을 명시한다.
- 타입 단언은 TS가 추론한 타입이 있더라도 해당 타입으로 간주한다.
- 타입 단언은 사실상 타입 체커에게 오류를 무시하라고 하는 것이다.
	- 타입 단언은 타입 오류를 없앨 뿐, 런타임 오류가 발생할 수 있다.

타입 스크립트는 DOM에 접근할 수 없기 때문에 `HTMLElement`와 같은 경우에는 타입 단언이 필요하다.

## 아이템 10

### 기본형을 객체로 래핑한다.

- `string` 기본형에는 메서드가 없지만, 자바스크립트는 메서드를 가지는 `String` 객체가 존재한다.
- 내장 메서드를 사용할 때, 기본형을 객체로 래핑하고, 메서드를 호출하고, 래핑한 객체를 버린다.

``` js
x = "hello"
x.language = 'English'
x.language // 오류
```

객체로 변환된다는 말만 보면 해당 코드는 정상 동작할 것 같지만, 실제로는 오류가 발생한다.

-> 메서드를 호출하고 객체를 버리기 때문이다.


## 아이템 11

### 잉여 속성 체크
- 객체 리터럴을 직접 사용할 때, 대상 타입에 선언되지 않은 속성을 실수로 작성했는지 검사하는 추가 안전장치이다.
- 선언된 타입으로 변수를 할당하거나, 함수의 매개변수, 함수의 반환값에서 동작한다.

```ts
interface Room {
	numDoors: number;
}

// 오류
const r: Room = {
	numDoors: 1,
	ceilingHeightFt: 10,
};

// 정상
const obj = {
	numDoors: 2,
	ceilingHeightFt: 20,
};
const r: Room = obj;
```

두 번째 방식이 정상 동작하는 이유는 `const r: Room = obj;` 에서 `obj`는 **객체 리터럴**이 아니기 때문에 잉여 속성 체크가 적용되지 않는다.

> 할당의 개념을 정확히 알아야 잉여 속성 체크와 일반적인 구조적 할당 가능성 체크를 구분할 수 있다.

TS에서의 할당 : 어떤 값의 타입을, 어떤 대상 타입으로 사용해도 되는지 검사하는 것

타입스크립트의 타입은 닫혀있지 않다. 즉 그 타입에 적힌 속성만 가져야 하는 게 아니라, 그 속성들을 최소한 가지고 있으면 된다.

## 아이템 12

### 타입스크립트에서는 함수 표현식을 사용하는 것이 좋다.
- 함수의 매개변수부터 반환값까지 함수 타입으로 선언해 재사용할 수 있다.

### Parameters 유틸리티 타입
- `Parameters<T>`는 **함수 타입 `T`의 매개변수 타입들을 튜플로 뽑아내는 내장 유틸리티 타입**
- 기존 함수와 같은 매개변수를 받는 새 함수를 만들 때 사용할 수 있다.
- 반환 타입은 추출하지 않으므로 새 함수에서 다르게 지정할 수 있다.

## 아이템 13

대부분의 경우에는 type을 사용해도 되고 interface를 사용해도 된다.
- 같은 상황에서는 동일한 방법으로 타입을 정의해 일관성을 유지해야 한다.
- 특정 목적이나 의도가 분명해지는 경우에는 type을 사용하고, 일반적으로는 interface를 사용한다.

> 접두사로 I 또는 T를 사용하는 것은 지양하자

### 공통점

JS에선 함수 또한 객체이기 때문에 함수 타입도 interface나 type으로 정의할 수 있다.
```ts
type TFn = (x: number) => string;
interface IFn {
	(x: number): string;
}
type TFnAlt = {
	(x: number): string;
};
```

함수 타입 정의는 타입 별칭을 사용하는 것이 더 자연스럽고 간결하다.

interface와 type은 서로 확장 가능하다.
- `interface`는 `extends`
- `type`은 교차 타입 `&`
- 오류를 더 찾아낼 수 있는 `interface`와 `extends` 조합을 사용하는 것이 좋다.

type과 interface의 재귀
``` ts
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

interface에는 유니온의 개념이 없다.

interface에는 **타입 보강**이 있다.
``` ts
interface IState {
  name: string;
  capital: string;
}

interface IState {
  population: number;
}

const wyoming: IState = {
  name: 'Wyoming',
  capital: 'Cheyenne',
  population: 578_000,
}; // 정상
```

핵심 : 반드시 사용해야 하거나 사용해야 할 이유가 있으면 type, 그렇지 않으면 일반적으론 interface를 사용한다.

## 아이템 14

`readonly` 속성을 사용해서 값 변경을 방지할 수 있다.

`Readonly<T>` 제네릭 유틸리티 타입을 사용하면 객체의 모든 속성에 할당을 방지할 수 있다.

`readonly` 주의사항
- 얕게 동작한다.
- `Readonly<T>` 가 속성에만 적용된다.

### 함수의 매개변수에 readonly 사용

```ts
function printNames(names: string[]) {
  names.push('Kim');
}
```
배열을 일반 배열로 받으면 함수 내부에서 직접 수정할 수 있다.
하지만 `readonly`를 사용하면 함수가 배열을 **읽기만 하고 변경하지 않는 함수**라는 사실을 타입에 표현할 수 있다.

- 함수 내에서 해당 매개변수가 변경이 발생하는지 체크한다.
- 함수의 호출부에 매개변수가 변경되지 않는다는 사실을 명시한다.
- 호출부에서는 readonly 배열이나 readonly 객체만 전달하게 된다.

함수가 매개변수를 변경하지 않는다면 readonly를 사용하는 것이 좋다.

## 아이템 15

같은 코드는 반복하지 마라는 **DRY 원칙**을 타입에서도 지켜야 한다.

타입 중복은 코드 중복만큼 많은 문제를 발생시킨다.
-> 가장 간단한 방법은 타입에 이름을 붙이는 것
```ts
interface Point2D{
	x: number;
	y: number;
}

function distance(a: Point2D, b: Point2D) {}
```

한 인터페이스가 다른 인터페이스를 확장하게 하는 방법을 통해 중복을 제거할 수 있다.
```ts
interface Person{
	firstName: string;
	lastName: string;
}

interface PersonWithBirth extends Person{
	birth: number;
}
```

### 타입 중복 제거해보기

```ts
interface State {
  userId: string;
  pageTitle: string;
  recentFiles: string[];
  pageContents: string;
}

interface TopNavState {
  userId: string;
  pageTitle: string;
  recentFiles: string[];
}
```
`userId`, `pageTitle`, `recentFiles`가 중복된다.

```ts
interface TopNavState {
  userId: State['userId'];
  pageTitle: State['pageTitle'];
  recentFiles: State['recentFiles'];
}
```
인덱스 접근 타입으로 중복을 줄일 수 있다.

```ts
type TopNavState = {
  [K in 'userId' | 'pageTitle' | 'recentFiles']: State[K];
};
```
매핑된 타입을 사용해 좀 더 개선할 수 있다.

- 이 패턴을 사용한 대표적인 예시가 `Pick`이다.

우연히 같은 이름과 타입을 가지고 있고, 용도가 다르면 중복을 제거하지 않아도 된다.
-> 잘못된 추상화보다 중복이 낫다는 말을 기억하자.

## 아이템 16

### 인덱스 시그니처

```ts
type Rocket = {
	[property: string]: string;
};
```

- 키의 이름(property) : 키의 위치만 표시하는 용도. 무시할 수 있는 정보이다.
- 키의 타입(string) : string | number | symbol의 서브타입이어야 한다.
- 값의 타입(string) : 어떠한 타입이 와도 가능하다.

단점
- 잘못된 키를 포함한 모든 키를 허용한다(ex. Name)
- 특정 키가 필요하지 않다.
- 키마다 다른 타입을 가질 수 없다.
- 자동완성 기능이 동작하지 않는다.

인덱스 시그니처는 동적 데이터를 표현할 때 사용하지만 동적인 데이터를 모델링하기 위해서는 `Map` 타입을 사용하는 것이 더 낫다. 

-> 데이터를 사용하는 시점에 오류를 발견하기보다는 오류를 미리 발견할 수 있다.

가능한 필드가 제한되어 있는 타입에는 인덱스 시그니처를 사용하면 안된다.

-> 가능하다면 인덱스 시그니처보다 정확한 타입을 사용하는 것이 좋다.

## 아이템 17

암시적 타입 강제란 JavaScript가 연산을 수행하기 위해 **개발자가 변환하지 않았는데도 값의 타입을 자동으로 바꾸는 것**을 의미한다.

### 자바스크립트 객체의 키는 문자열

배열도 객체이므로 숫자로 접근해도 런타임에서는 문자열 키로 변환된다.

``` ts
const xs = [10, 20, 30];

xs[1];       // 20
xs["1"];     // 20
Object.keys(xs); // ["0", "1", "2"]
```

따라서 다음 객체의 `1`도 실질적으로 `"1"`이라는 키이다.

``` ts
const x = { 1: "one" };
```

### TypeScript의 숫자 인덱스는 런타임 동작이 아니다

``` ts
interface NumberDictionary {
  [index: number]: string;
}
```

이는 **타입 검사 단계에서만 존재하는 편의 기능**. 자바스크립트 런타임에 진짜 숫자 키가 생기는 것은 아니다.

``` ts
const values: NumberDictionary = {
  0: "zero",
  1: "one",
};

Object.keys(values); // 타입과 달리 실제 결과는 string[]
```

### 배열 메서드가 필요 없다면 `ArrayLike<T>`

함수가 숫자 인덱싱과 `length`만 필요로 한다면 `ArrayLike<T>`를 사용한다.

``` ts
function checkedAccess<T>(
  values: ArrayLike<T>,
  index: number
): T {
  if (index < values.length) {
    return values[index];
  }

  throw new Error();
}
```

일반 배열뿐 아니라 다음처럼 배열과 비슷한 객체도 받을 수 있다.

``` ts
const tupleLike: ArrayLike<string> = {
  0: "A",
  1: "B",
  length: 2,
};
```

`ArrayLike` 객체의 키도 런타임에서 확인하면 문자열이다.
