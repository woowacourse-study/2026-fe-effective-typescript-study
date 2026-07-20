## 아이템4

### 구조적 타이핑 vs 덕 타이핑

**덕 타이핑**

- 필요한 속성과 동작을 가지고 있는지를 기준으로 사용할 수 있다고 판단하는 방식
- 대표적인 예시 -> 자바스크립트
- 런타임에 타입을 검사

**구조적 타이핑**

- 함수의 매개변수 타입이 요구하는 속성을 객체가 모두 가지고 있다면, 해당 인자로 전달할 수 있다고 판단하는 방식
- 대표적인 예시 -> 타입스크립트
- 컴파일 시 타입을 검사

### 클래스 할당

```ts
class SmallNumContainer {
  num: number;

  constructor(num: number) {
    if (num < 0 || num >= 10) {
      throw new Error();
    }

    this.num = num;
  }
}

const b: SmallNumContainer = { num: 2024 }; // 정상
```

해당 코드가 정상 동작하는 이유

- 클래스 타입도 **구조적으로 비교**하기 때문
- 생성자 함수는 인스턴스를 생성하는 클래스 자체의 기능이므로 생성자는 보지 않는다.

#### private

`private num: number` 처럼 `private` 를 사용하면 오류가 발생한다.
-> TypeScript에서 `private`가 포함된 클래스는 단순히 구조만 같다고 호환되지 않는다.

다른 클래스에 똑같은 `private` 멤버를 만들어도 안 된다.

```ts
class Other {
  private num: number;

  constructor(num: number) {
    this.num = num;
  }
}

const other = new Other(5);
const value: SmallNumContainer = other; // 오류
```

두 `private num`이 서로 다른 클래스에서 선언됐기 때문이다.

### "덕 타이핑을 구조적 타이핑으로 모델링한다"의 의미

- JavaScript의 유연한 사용 방식을 TypeScript가 객체의 구조를 비교하는 타입 검사 규칙으로 표현했다.
- JavaScript에서는 현재 필요한 동작을 할 수 있는지가 중요하다. TypeScript는 이러한 사용 방식을 객체의 구조를 비교하는 구조적 타입 시스템으로 정적으로 검사한다.

즉, 필요한 동작을 할 수 있는지 검사하는 덕 타이핑을 구조를 비교하는 타입 검사로 표현한 것이다.

## 아이템 5

### any 타입에는 타입 안정성이 없습니다

```ts
let age: number;
age = '12' as any;
age += 1;
```

위 코드에서 `age`는 121이 된다.

number 타입으로 선언하였지만 `as any` 를 사용하여 string 타입의 값을 할당했다.
-> any 타입을 사용하면 타입 안정성을 보장할 수 없다.

### any vs unknown

`any`와 `unknown`은 둘 다 **어떤 값이든 받을 수 있는 타입**이지만, 값을 사용할 때의 안전성이 다르다.

#### any

타입 검사를 사실상 무시

```ts
let value: any = 'hello';

value.toUpperCase(); // 가능
value.foo.bar(); // 가능
value(); // 가능
```

단, 실제 값이 예상과 다르면 런타임 에러가 발생한다.

#### unknown

어떤 값이든 받을 수 있지만, **타입을 확인하기 전에는 사용할 수 없다.**

-> 타입 좁히기를 먼저 해야 한다.

```ts
let value: unknown = 'hello';

value.toUpperCase();
// 오류: value의 타입을 알 수 없음

if (typeof value === 'string') {
  value.toUpperCase(); // 가능
}
```
