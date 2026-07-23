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
