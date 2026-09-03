### 아이템 65. devDependencies에 타입스크립트와 @types 추가하기

#### 왜 devDependencies인가

타입스크립트는 컴파일이 끝나는 순간 사라진다. 빌드 결과물인 자바스크립트 어디에도 타입은 남지 않는다.
즉 **런타임에는 필요 없고 개발 시점에만 필요한 도구**이므로 `devDependencies`에 두는 게 맞다.

#### 챙겨야 할 두 가지 의존성

**1. 타입스크립트 컴파일러 자체**

전역 설치(`npm i -g typescript`)는 편해 보이지만 함정이 있다.

- 팀원 A는 5.4, 팀원 B는 5.7을 쓰고 있어도 아무도 모른다
- 내 로컬에선 통과한 빌드가 CI에서 깨진다
- "제 컴퓨터에선 되는데요"의 전형적인 시나리오

프로젝트의 `devDependencies`에 명시하면 `package.json` 하나로 모두가 같은 버전을 쓰게 되고, CI 환경까지 자동으로 동일해진다.

**2. 타입 의존성 (@types)**

`@types` 패키지는 DefinitelyTyped에서 관리하는 타입 선언 모음이다.
`@types/react`처럼 라이브러리의 타입 정보만 담고 있고, 실행되는 코드는 한 줄도 들어있지 않다.

여기서 헷갈리기 쉬운 지점 -> **원본 라이브러리가 `dependencies`여도 `@types`는 `devDependencies`다.**

```json
{
  "dependencies": {
    "react": "^18.3.1"
  },
  "devDependencies": {
    "typescript": "^5.7.2",
    "@types/react": "^18.3.12"
  }
}
```

`react`는 브라우저에서 실제로 돌아가지만, `@types/react`는 컴파일이 끝나면 할 일이 없다.

-> 타입스크립트와 `@types`는 둘 다 런타임에 존재하지 않는다. 전역이 아니라 프로젝트의 `devDependencies`에 넣자.

---

### 아이템 66. 타입 선언과 관련된 세 가지 버전 이해하기

#### 타입스크립트를 쓰면 버전 하나로는 안 끝난다

보통 자바스크립트에서 의존성 관리는 "라이브러리 버전 하나"만 신경 쓰면 된다.
그런데 타입스크립트를 도입하는 순간 관리해야 할 버전이 셋으로 늘어난다.

1. **라이브러리 버전** : 실제 동작하는 코드
2. **타입 선언(@types) 버전** : 그 코드를 설명하는 타입
3. **타입스크립트 버전** : 그 타입을 해석하는 컴파일러

세 축이 각각 따로 움직이기 때문에, 하나만 어긋나도 원인을 짐작하기 어려운 오류가 튀어나온다.
"라이브러리는 잘 돌아가는데 빌드만 안 되는" 상황이 대표적이다.

#### 어긋났을 때 생기는 대표적인 상황

| 상황                            | 증상                                                      |
| ------------------------------- | --------------------------------------------------------- |
| 라이브러리 > 타입 선언          | 새로 추가된 API가 타입에 없어서 `Property does not exist` |
| 타입 선언 > 라이브러리          | 타입상 존재하는 메서드가 런타임엔 `undefined`             |
| 타입 선언이 요구하는 TS > 내 TS | 최신 문법을 못 읽어서 선언 파일 자체에서 오류             |

세 번째가 특히 성가시다. 내 코드는 건드린 적도 없는데 `node_modules` 안쪽에서 에러가 나기 때문이다.

#### 패치 버전을 얕보면 안 되는 이유

세 버전이 독립적이라 보통 `x.x.0` → `x.x.1` 같은 패치 변경에서는 공개 API 사양이 바뀌지 않는다.
그래서 메이저·마이너가 같다면 굳이 올릴 필요가 없다고 생각하기 쉽다.

하지만 **타입 선언의 패치 버전은 성격이 다르다.**
`@types` 패키지에서 패치가 올라가는 이유는 대부분 이런 것들이다.

- 잘못 선언된 타입 수정
- 빠져 있던 API 추가
- 너무 좁게/넓게 잡힌 시그니처 교정

즉 라이브러리 쪽 패치는 "버그 수정", 타입 선언 쪽 패치는 "타입 버그 수정"이다.
내가 겪고 있는 이상한 타입 오류가 이미 패치되어 있을 가능성이 꽤 있다는 뜻이다.

#### 호환되지 않을 때의 해결 순서

**1. 버전을 서로 맞춘다 (기본)**

라이브러리와 타입 선언의 메이저·마이너를 일치시키고, 필요하면 타입스크립트 버전도 올린다.
`@types`가 요구하는 최소 TS 버전은 해당 패키지 문서에 명시되어 있다.

**2. 그래도 안 되면 보강한다**

타입 선언이 아직 안 고쳐졌거나, 라이브러리 버전을 올릴 수 없는 상황이라면
직접 선언 보강(declaration merging)으로 구멍을 메운다.

```ts
// types/some-lib.d.ts
import "some-lib";

declare module "some-lib" {
  interface Options {
    // 실제로는 지원하지만 타입 선언에 빠져 있는 옵션
    retryCount?: number;
  }
}
```

`any`로 덮어버리는 것보다 훨씬 안전하고, 나중에 공식 타입이 고쳐졌을 때 이 파일만 지우면 된다.

-> 라이브러리·타입 선언·타입스크립트, 이 세 버전이 어긋나면 이해하기 힘든 오류가 생긴다. 먼저 버전을 서로 맞추고, 그래도 안 되면 보강 기법으로 채워 넣자.

---

### 아이템 67. 공개 API에 등장하는 모든 타입 export하기

라이브러리를 만들다 보면 "이 타입은 내부 구현용이니까 굳이 공개할 필요 없겠지" 싶어서 `export`를 빼놓게 된다.

```ts
interface SecretName {
  first: string;
  last: string;
}

interface SecretSanta {
  name: SecretName;
  gift: string;
}

export function getGift(name: SecretName, gift: string): SecretSanta {
  // ...
}
```

함수만 내보내고 타입 두 개는 감췄다. 그런데 이 두 타입은 이미 **공개 API의 시그니처에 등장하고 있다.** 사용자는 `getGift`를 쓰려면 어떤 모양의 인자를 넘겨야 하는지, 무엇이 돌아오는지 이미 알고 있어야 한다.

그리고 알고 있는 것을 넘어서, 꺼내 쓸 수도 있다.

#### Parameters와 ReturnType으로 추출하기

타입스크립트는 함수 타입에서 매개변수와 반환 타입을 뽑아내는 유틸리티 타입을 제공한다.

```ts
type MySanta = ReturnType<typeof getGift>; // SecretSanta
type MyName = Parameters<typeof getGift>[0]; // SecretName
```

`typeof getGift`로 함수의 타입을 얻고, `ReturnType`으로 반환 타입을, `Parameters`로 매개변수 타입의 튜플을 얻은 뒤 인덱스로 꺼낸다.

이름만 다를 뿐 결과물은 원본 그대로다. 결국 **감춘 쪽은 아무것도 지키지 못했고, 사용자만 번거로워졌다.**

#### 그래서 그냥 익스포트하자

의도적으로 숨긴 타입이라도 공개 API에 등장하는 이상 사용자는 어떻게든 손에 넣는다. 그렇다면 처음부터 내보내는 편이 낫다.

```ts
export interface SecretName {
  first: string;
  last: string;
}

export interface SecretSanta {
  name: SecretName;
  gift: string;
}

export function getGift(name: SecretName, gift: string): SecretSanta {
  // ...
}
```

익스포트해두면 사용자가 이런 일들을 편하게 할 수 있다.

- 함수를 감싸는 래퍼를 만들 때 인자 타입을 그대로 재사용
- 반환값을 담을 변수나 상태에 타입을 명시
- 그 타입을 확장하거나 `Pick`, `Omit`으로 가공

반대로 감춰두면 사용자는 위의 `ReturnType` 곡예를 부리거나, 아예 타입을 손으로 다시 적는다. 어느 쪽이든 원본이 바뀌었을 때 조용히 깨지는 코드가 된다.

-> 공개 API의 시그니처에 등장하는 타입은 이미 공개된 것이나 다름없다. 사용자가 추출해 쓰게 만들지 말고 처음부터 `export`하자.

---

### 아이템 68. API 주석에 TSDoc 사용하기

같은 설명이라도 어떻게 쓰느냐에 따라 사용자에게 보이기도 하고 안 보이기도 한다.

```ts
// 인사말을 생성합니다. 결과는 보기 좋게 꾸며집니다.
function greetInline(name: string, title: string) {
  return `Hello ${title} ${name}`;
}

/** 인사말을 생성합니다. 결과는 보기 좋게 꾸며집니다. */
function greetTSDoc(name: string, title: string) {
  return `Hello ${title} ${name}`;
}
```

둘 다 사람이 읽으면 똑같지만, 편집기에서 함수 위에 마우스를 올렸을 때 툴팁에 뜨는 쪽은 아래뿐이다.

| 형태                | 편집기 툴팁 | 자동완성 목록 |
| ------------------- | ----------- | ------------- |
| `// 인라인 주석`    | 안 보임     | 안 보임       |
| `/** TSDoc 주석 */` | 보임        | 보임          |

즉 인라인 주석은 **코드를 열어본 사람만** 읽지만, TSDoc 주석은 **API를 쓰는 사람에게 그 자리에서** 전달된다.

#### @param과 @returns

각 매개변수와 반환값도 따로 설명할 수 있다.

```ts
/**
 * 인사말을 생성한다.
 * @param name 인사할 대상의 이름
 * @param title 대상의 칭호
 * @returns 사람이 읽기 좋은 형태의 인사말
 */
function greet(name: string, title: string) {
  return `Hello ${title} ${name}`;
}
```

이렇게 써두면 인자를 입력하는 도중에도 편집기가 지금 채우고 있는 매개변수의 설명을 띄워준다.

#### 타입 정의에도 붙일 수 있다

함수뿐 아니라 인터페이스와 그 필드 하나하나에도 붙는다.

```ts
/** 특정 시간과 장소에서 수행된 측정 */
interface Measurement {
  /** 어디에서 측정되었나? */
  position: Vector3D;
  /** 언제 측정되었나? (epoch 기준 초 단위) */
  time: number;
  /** 측정된 운동량 */
  momentum: Vector3D;
}
```

객체 리터럴을 작성할 때 각 속성마다 이 설명이 그대로 따라 나온다. 타입 이름만으로는 전달되지 않는 **단위, 기준, 뉘앙스**를 적어두기 좋은 자리다. 위의 `time: number`에서 "초 단위인가 밀리초 단위인가"는 타입이 절대 알려줄 수 없는 정보다.

#### 마크다운은 되지만, 길게 쓰지는 말자

TSDoc 주석은 마크다운 형식을 지원한다. 굵은 글씨, 목록, 코드 블록이 툴팁에 그대로 렌더링된다.

```ts
/**
 * 이 **인터페이스**는 다음 세 가지를 가진다:
 * - 위치
 * - 시간
 * - 운동량
 */
```

다만 쓸 수 있다는 것과 써야 한다는 것은 다르다. 툴팁은 잠깐 스쳐 지나가는 공간이고, 사용자가 원하는 건 긴 설명이 아니라 지금 필요한 한 줄이다. **장황하게 늘어놓지 말고 요점만** 적는 게 좋다.

#### 주석에 타입을 적지 말 것

JSDoc에는 타입 정보를 적는 문법이 있다.

```ts
/**
 * @param {string} name 인사할 대상의 이름
 * @returns {string} 인사말
 */
```

타입스크립트에서는 쓰지 않는다. 타입은 이미 코드에 있고, 컴파일러가 검사해준다. 주석에 한 번 더 적으면 **검사되지 않는 사본**이 하나 늘어나는 셈이라, 시그니처를 고치는 순간 주석만 조용히 틀린 말을 하게 된다.

주석은 타입이 말하지 못하는 것만 담당하면 된다. 무엇을 하는 함수인지, 어떤 전제가 있는지, 값의 단위와 범위는 무엇인지 같은 것들이다.

-> 편집기가 보여주는 주석은 `/** */` 형태뿐이다. 공개 API에는 TSDoc으로 요점만 적고, 타입은 주석이 아니라 코드에 맡기자.

### 아이템 69. 콜백 함수에서 this에 대한 타입 제공하기

#### this는 호출 방식이 결정한다

`let`, `const`는 렉시컬 스코프라 작성된 위치만 보면 값을 알 수 있다.
`this`는 다이내믹 스코프라 **호출하는 쪽**을 봐야 안다.

```ts
class C {
  vals = [1, 2, 3];
  logSquares() {
    for (const val of this.vals) console.log(val ** 2);
  }
}

const c = new C();
c.logSquares(); // 1 4 9

const method = c.logSquares;
method(); // TypeError: Cannot read properties of undefined
```

`c.logSquares()`는 사실 두 가지 일을 한다. 메서드를 호출하고, `c`를 `this`로 바인딩한다. 참조만 떼어내면 두 번째가 사라진다.
메서드는 객체에 소속된 게 아니라 그냥 프로퍼티에 담긴 함수이기 때문이다. `call`/`apply`/`bind`는 이 두 번째 일을 수동으로 시키는 장치다.

#### 콜백으로 넘길 때 터진다

```ts
class ResetButton {
  render() {
    return makeButton({ text: "Reset", onClick: this.onClick }); // this가 사라짐
  }
  onClick() {
    alert(`Reset ${this}`);
  }
}
```

`this.`는 함수를 꺼내오는 경로일 뿐이고, 넘어가는 값은 함수 그 자체다. 해결책은 두 가지이다.

```ts
// 1. 생성자에서 bind -> this가 고정된 새 함수로 덮어쓴다
constructor() { this.onClick = this.onClick.bind(this); }

// 2. 화살표 함수 프로퍼티 -> 자기 this가 없어 바깥(인스턴스) this를 쓴다
onClick = () => { alert(`Reset ${this}`); };
```

둘 다 함수가 프로토타입이 아닌 인스턴스마다 생긴다는 대가는 있다.

TS의 this 바인딩은 JS와 동일하다. TS가 추가로 주는 건 **this에 무엇이 와야 하는지를 타입으로 못 박는 기능**이다. 첫 번째 매개변수 자리의 `this`는 실제 인자가 아니라 컴파일 타임 전용 표식이다.

```ts
function addKeyListener(
  el: HTMLElement,
  listener: (this: HTMLElement, e: KeyboardEvent) => void,
) {
  el.addEventListener("keydown", (e) => listener.call(el, e));
}
```

이득이 양방향이다.
구현 쪽에서는 `listener(e)`처럼 this 없이 호출하면 에러가 나고, 사용하는 쪽에서는 잘못된 this가 잡힌다.

```ts
class Foo {
  registerHandler(el: HTMLElement) {
    addKeyListener(el, (e) => {
      this.innerHTML; // ~~~~~~~~~ 'Foo'에 'innerHTML' 속성이 없습니다
    });
  }
}
```

화살표 함수 안의 `this`는 `Foo`인데 콜백은 `HTMLElement`를 기대하므로, 런타임에 터질 코드가 컴파일 단계에서 걸린다. (`noImplicitThis`가 전제)

-> 메서드 참조를 떼어내면 바인딩이 사라진다. 콜백으로 넘길 때는 `bind`나 화살표 함수로 고정하고, 콜백 타입에 `this`가 쓰인다면 `this` 매개변수로 명시한다. 애초에 콜백이 `this`를 쓰지 않게 설계하는 것이 가장 좋다.

---

### 아이템 70. 의존성 분리를 위해 미러 타입 사용하기

#### 타입 하나 때문에 딸려오는 의존성

```ts
function parseCSV(contents: string | Buffer) {
  /* ... */
}
```

`Buffer` 때문에 `@types/node`가 필요해지고, 라이브러리로 공개하면 이게 **사용자에게 전파되는 의존성**이 된다. 근데 단점이 있다.

- JS로 쓰는 사람: `@types` 자체가 필요 없다
- 브라우저 TS 개발자: Node 타입이 필요 없다
- Node TS 개발자: 이미 갖고 있다

셋 중 둘에게는 낭비다. `Buffer` 타입 정의는 방대한데 실제로 쓰는 건 `toString` 하나뿐이다.

TS는 구조적 타이핑을 쓰므로, 쓰는 메서드만 가진 인터페이스를 직접 선언하면 진짜 `Buffer`도 그대로 통과한다.

```ts
interface CsvBuffer {
  toString(encoding?: string): string;
}

function parseCSV(contents: string | CsvBuffer) {
  /* ... */
}
```

이렇게 외부 타입에서 실제로 쓰는 부분만 흉내 낸 타입이 **미러 타입**이다.

#### 트레이드오프

미러 타입은 원본과 연결이 끊긴 복사본이라, 원본이 바뀌어도 컴파일러가 알려주지 않는다. 실제 `Buffer`를 넣어보는 단위 테스트로 확인해야 한다. 그리고 복제량이 많아지면 그건 신호이다. 그때는 정식 의존성을 추가하는 편이 낫다. **라이브러리를 공개할 때**는 필수가 아닌 타입 의존성을 끊을 가치가 크고, **애플리케이션 코드**에서는 의존성이 이미 있으니 그냥 원본 타입을 쓰면 된다.
