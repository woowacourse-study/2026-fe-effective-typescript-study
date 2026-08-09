제어 흐름 분석 -> 주변 코드이ㅡ 영향을 받아 타입이 변경되는 경우

명시적으로 타입을 작성해야 될 때와 필요없을 때를 구분해야한다.

## 아이템 18 추론 가능한 타입을 사용해 장황한 코드 방지하기

매번 타입을 명시적으로 지정해줄 필요는 없다.
-> 추론을 믿자.

오히려 명시적 타입 선언이 코드를 더 번잡하게 만들 때가 있다.

정보가 부족해 TS가 추론을 제대로 못할 때는 명시적 타입 구문을 작성해야한다.

객체 리터럴을 정의할 때 명시적 타입을 쓰면 잉여 속성 체크를 쓸 수 있어서 유용하다.

반환 타입 정의하는 것도 오류를 잡기 유용해서 명시하는게 좋다.

미리 반환타입을 명시하는 것은 TDD같은 느낌이다. 엉뚱한 코드가 되지 않는다 타입을 미리 명시하기 때문

## 아이템 19 다른 타입에는 다른 변수 사용하기

let을 쓰면 값은 바뀌지만, 타입은 최초 이후에 바뀌지 않는다.
바꾸려면 타입을 좁혀야 한다.

제목 그 자체다.

## 아이템 20 변수의 타입이 결정되는 원리 이해하기

let x = 'x'를 하면 기본형인 string으로 추론된다.
하지만 const 키워드를 써서
const x = 'x'로 하면, 'x' 리터럴 타입이 된다.

const로 변수를 선언하면 더 좁은 타입이 된다!
let이 저렇게 추론되는 이유는 재할당 가능성이 있다고 판단해서 그럼

const를 써도 객체에 대해서는 구체적인 타입 추론이 안됨. -> 각 요소를 let으로 할당한 것처럼 다룸

> TS의 타입 추론은 적당히 구체적이다

TS에 타입 추론에 직접 개입할 수 있는 방법은 3가지

1. 타입 직접 지정
2. 추가 문맥을 제공(매개변수 등으로)
3. as const 사용
4. Object.freez
5. satisfies

as const를 사용하면 **최대한 좁은 타입**으로 추론한다!!!
-> 이게 as const 용도

> 단언문은 쓰면 안 좋은데, as const는 타입 안정성을 해치지 않으므로 얼마든지 사용해도 괜찮다!

as const쓰면 배열을 튜플 타입으로 추론할 수도 있음 대신 [1,2,3] as const 하면 readonly [1, 2, 3] 이렇게 된다.

```ts
function tuple<T extends unknown[]>(...elements: T) {
  return elements;
}
const arr3 = tuple(1, 2, 3);
```

근데 tuple이라는 타입 추론용 함수를 만들면 위 arr3가 [number,number,number]로 추론이 잘 된다!

---

Object.freeze 쓰면 추론된 타입을 `readonly`로 만들어 줌!

- JS 런타입에 동작
- freeze와 readonly는 얕지만, const 단언문은 깊다.

---

satisfies도 TS가 너무 넓게 타입을 추론하지 않도록 해주는 역할임

객체를 사용하는 위치가 아닌, 객체를 정의하는 시점에서 오류를 발견할 수 있음 -> 이 관점에서 const 단언보다 satisfies가 낫다고 할 수 있다!

## 아이템 21 한꺼번에 객체 생성하기

원래 JS에서 빈 객체( {} <- 이거> )를 만들고 나중에 프로퍼티를 추가하는 것이 자연스럽다. 하지만 TS에서는 이걸 빈 객체로 추론해버린다.

그리고 TS 타입은 일반적으로 변경되지 않는다. 그래서 문제다.

이 아이템은 객체에 새로운 속성을 추가하는 것에 대한 이야기이다.

단언을 쓸 수 있지만, 좋지 않다.

그래서 spread연산자 적극적으로 사용하자. 그러면 추론이 잘 된다.

spread할때 어떤 값이 true일때만 추가하고 싶은 프로퍼티가 있을 수도 있는데 이것도 역시 TS가 optinal로 잘 추론을 해준다.

## 아이템 22 타입 좁히기

사용자 정의 타입가드가 타입 단언(as)보다 더 안전한 것은 아니다.

반환값이 true인 경우 해당 타입으로 좁히는 것인데, 반환하는 값이 반환 타입과 일치하는지는 체크하지 않기 때문이다.

TS가 map의 has와 get 메서드 사이의 관계를 제대로 이해하지 못한다. has했다고 get이 될거라고 추측하지 못한다는 것이다. -> 별도로 작성해야된다.

타입을 좁히고, 그 다음에 setTimeout같은 비동기 작업을 하면, 좁혀놨음에도 값이 어떻게 될지 모르기때문에 TS가 의도적으로 좁혔던 타입을 무효화시킴

## 아이템 23 일관성 있는 별칭 사용하기

구조분해할당 써라, optional인 경우에는 추가적인 null체크가 필요하다.

JS에서 객체,배열 아니면 다 깊은복사임

## 아이템 24 타입 추론에 문맥이 어떻게 사용되는지 이해하기

```ts
type Language = "JavaScript" | "TypeScript" | "Python";
function setLanguage(language: Language) {
  /* ... */
}

setLanguage("JavaScript"); // OK

let language = "JavaScript"; // -> string 타입으로 추론
setLanguage(language);
//          ~~~~~~~~ Argument of type 'string' is not assignable
```

해결 방법은 2가지이다.

첫 번째 방법은 `language : Language`로 할당할 때 타입을 써주는 것이다.
두 번째는 이전에 배웠던 것처럼 const를 이용하면, JavaScript 문자열 리터럴 타입이 되어서 할당 가능

> Q. const로 따로 상수화를 시키는 것 자체가 문맥을 잃는다고 표현하는 것 같은데 맞나?
> A. let이든 const든 따로 분리해서 선언하는 것 자체가 문맥을 잃게 만드는 행위이다.

튜플 타입
-> `const loc = [10, 20]`으로 정의하면, TS는 이걸 number 배열로 추론함.
우리가 원하는건 위도, 경도 이므로 길이 2를 가지는 정확한 튜플타입이어야 함!

`const loc : [number,number]= [10, 20]`도 되고, as const를 써도됨

- 중요한 것은 as const로 튜플타입을 만들면 readonly가 된다는 것임.

그래서 정의하는 곳에서도 readonly를 써줘야함

객체도 상수를 뽑아내면, 문맥을 잃어버림.
이런 공통적인 이유가 string으로 추론되거나 그래서 그럼

객체, 콜백함수를 상수로 뺄 일이 생긴다면, 문맥을 잃게되므로 타입을 잘 맞춰주어야 한다는 것이 이번 아이템의 핵심인듯.

## 아이템 25 타입의 진화 이해하기

```ts
function range(start: number, limit: number) {
  const nums = []; // any[]로 추론된다.
  for (let i = start; i < limit; i++) {
    nums.push(i);
  }
  return nums;
  //     ^? const nums: number[]
}
```

첨엔 any[]인데 나중에는 number[]로 바뀜

push할 때까지도 any[]임!

배열에 여러 타입의 값을 넣으면 유니온이 됨. `진화`하는 것임

이런걸 any의 진화, let의 진화, 배열의 진화라고 부름.

- 할당이 되는 그 line에서는 이 동작이 일어나지 않음
- 할당이 되고 나서 그 이후에 진화되는 것임

조건문에서도 똑같이 사용할 수 있음.

이거의 장점은 타입 구문을 줄일 수 있다는 것임

타입 추론을 개선하기 위해서 forEach 대신 for-of 루프를 사용하는 것이 좋다.

> const를 쓰고 null or undefined를 넣으면 각자 타입이 나오는데, let쓰면 any로 추론됨.

## 아이템 26 함수형 기법과 라이브러리로 타입 흐름 유지하기

서드파티 라이브러리를 써서 코드를 짧게 줄이는 것이 시간이 많이 들면, 안 쓰는 것이 낫다.
하지만 TS의 경우에는 라이브러리나 잘 되어있는 내장함수를 쓰는 편이 더 좋다.
타입 추론이 더 정확하게 되기 때문이다.

## 아이템 27 비동기 코드에는 콜백 대신 async 함수 사용하기

await 없이 async만 붙여도 Promise객체를 반환하는 함수가 되어 버린다. -> 항상 비동기 함수로 통일하도록 강제함

## 아이템 28 클래스와 커링 기법으로 새로운 추론 영역 만들기

```ts
export interface SeedAPI {
  "/seeds": Seed[];
  "/seed/apple": Seed;
  "/seed/strawberry": Seed;
  // ...
}
```

클라이언트 입장에서는 이 엔드포인트가 존재하고, 정의된 타입에 맞는 데이터를 반환하는지 체크해야됨.

> Q. declare가 무슨 키워드지?
> A. AI왈, declare는 타입스크립트 컴파일러에게 "이 변수나 함수는 실제로 어딘가에 존재하니까, 에러 내지 말고 내가 적어준 타입대로 믿고 통과시켜 줘!"라고 선언하는 문법이야.

> Q. Path extends keyof API 이 말이 집합 관점으로 표현하면, API의 key 유니온 타입의 부분집합이 Path라면 ok라는거야? 만족하지 않으면 어떻게 되는데?
> A. 에러가 발생하겠지 ㅇㅇ,,

```ts
const berry = fetchAPI<SeedAPI, "/seed/strawberry">("/seed/strawberry"); // ok
```

이건 너무 번거로움! api 경로를 직접 작성해야됨

#### 해결책1 - 클래스

ApiFetcher라는 클래스를 만들면, 1차로 어떤 API인지 확인하고 2차로 해당하는 API의 엔드포인트인지 검증할 수 있음!

#### 해결책2 - 커링

커링은 함수를 반환하는 함수이다. 클래스를 사용할 때와 문법만 다르고 똑같은 원리이다.

클래스나 커링 둘중 하나 원하는거 사용하면 된다.
하지만 커링 방식을 사용하는게 클래스보다 딱 1개 좋은 점이 있는데, 로컬 타입 별칭으로 타입 재사용이 가능하다는 점이다.

> Q. 로컬 타입 별칭이 뭐지?
> A. 특정 함수 스코프 안에서만 쓰이고 버려지는 타입 별칭

## 아이템 36 특수값에는 별도의 타입 사용하기

JS에서 -1은 특수한 값으로 쓰인다. 단순 숫자이지만, 그 의미를 지니기 때문에 특수한 타입으로 만들어 주어야한다.

양수만 들어올 수 있는 변수에 뭔가 특수한 값을 처리하기 위해 음수를 사용한다고 하면, 우리는 잘 받아들일 수 있지만 어디가에서 버그가 터질 수 있다.

따라서 이런 것들은 명확하게 null같은 값으로 처리를 해주어야한다.

네트워크 통신 같은 것도 지연됨, 오류같은 상태를 null이나 undefined로 처리하기보다는 태그된 유니온을 쓰는 것이 낫다. 아마도 모델링한다고 하면 이런식?

```ts
interface NetworkError {
  state: "error";
  // ...
}

interface NetworkDelayed {
  state: "delay";
  // ...
}
```

중요한건 null을 써라 쓰지말라가 아니다.
특수한 상황을 다뤄야 한다면 그 상황에 맞는 최선의 타입을 선택하라는 것이다.

없으면 없다고 null, 에러는 에러라고 표시하는 "error"처럼

## 아이템 37 선택적 속성 지양하기

Optional을 사용하면 문제가 있다.
바로 그 값이 없을 때의 처리를 값이 사용되는 곳마다 매번 처리를 해주어야 한다는 것이다.
특히 선택적 속성이 없을 때 기본값을 설정해놨을 때 실수로 선택적 속성을 적어주지 않은 곳에서는 의도하지 않은 동작이어도 그걸 찾기가 힘들다.

그래서 이 책에서 제시하는 방법은 어차피 기본값이 있다면, 최초에 정규화를 하라는 것이다.

```ts
interface InputAppConfig {
  darkMode: boolean;
  // ... other settings ...
  /** default is imperial */
  unitSystem?: UnitSystem;
}
interface AppConfig extends InputAppConfig {
  unitSystem: UnitSystem; // required
}
```

이렇게 처음부터 정규화를 해놓고 쓰면 사용하는 곳에서 기본값 처리를 해줄 필요가 없다.
이렇게 하는 것 대신에
`type AppConfig = Required<InputAppConfig>`로 할 수도 있다.

> Required<T>는 그 타입의 모든 속성을 required로 만들어줌. 기존에 optional인 것들도!

Optional을 그럼 대체 언제 써야하냐?

- 이미 존재하는 API를 모델링하거나 API를 수정할 때
- 실제로 선택적인 경우 (middleName 같은 경우)

## 아이템 38 같은 타입의 매개변수를 반복하지 않기

drawRect(25,50,75,100,1); 이 코드만 보고 얘가 뭘 하는지 알 수 있음? 절대 ㄴ

다같은 number타입이라 순서를 바꾸어 입력해도 타입 체크는 작동하지 않는다.

```ts
interface Point {
  x: number;
  y: number;
}
interface Dimension {
  width: number;
  height: number;
}
function drawRect(topLeft: Point, size: Dimension, opacity: number) {
  // ...
}
```

이렇게 완전히 분리시켜 받는다면 잘못된 값을 받기가 힘들어진다.

```ts
interface DrawRectParams extends Point, Dimension {
  opacity: number;
}
function drawRect(params: DrawRectParams) {
  /* ... */
}

drawRect({ x: 25, y: 50, width: 75, height: 100, opacity: 1.0 });
```

이렇게 1개의 객체로 만드는 것도 방법이다.

결론은 직관적이고 사용하기 쉬운 매개변수 타입을 설계하라는 것이다.
