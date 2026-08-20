### 아이템 43. `any` 타입은 가능한 한 좁은 범위에서만 사용하기

`any`를 쓴다는 건 그 지점의 에러를 지우는 게 아니라, **그 지점부터 타입 정보를 흘려보내는 것**에 가까움.
→ 그래서 어디에 붙이느냐가 곧 얼마나 넓게 오염되느냐가 됨.

```tsx
declare function loadConfig(): Config;
declare function applyTheme(theme: Theme): void;

const config = loadConfig();
applyTheme(config); // Config는 Theme에 할당 불가
```

여기서 에러를 없애는 방법은 두 가지

**A. 변수 선언을 any로**

```jsx
const config: any = loadConfig();

applyTheme(config);   // 통과
config.verison;       // 오타인데 통과 <- 체커가 아예 꺼짐
```

**B. 전달하는 그 자리에서만 as any**

```tsx
const config = loadConfig(); // Config 유지

applyTheme(config as any); // 통과
config.verison; // 여기선 여전히 잡힘
```

**B**A는 `config`를 참조하는 모든 코드가 무방비가 되지만, B는 인자로 넘기는 그 표현식 하나에만 효력이 있음.
같은 타입 체크 끄기라도 A는 **변수의 주기 전체**, B는 **한 부분이** 해당사항임.

#### 반환 타입을 명시해서 밖으로 새는 걸 막기

함수 안에서 `any`를 썼는데 반환 타입을 추론에 맡기면, 그 `any`가 **호출자 쪽으로 그대로 흘러나감.**

```tsx
// 반환 타입이 any로 추론됨 - 호출하는 모든 곳이 오염
function getUser(id: string) {
  const raw = fetchFromLegacyApi(id) as any;
  return raw.data;
}

// 경계에서 막음
function getUser(id: string): User {
  const raw = fetchFromLegacyApi(id) as any;
  return raw.data;
}
```

TS가 알아서 추론해준다고 해도 **함수 시그니처는 명시하는 편이 좋음.** 스코프를 좁힌다는 관점의 연장선이다.
아

#### 객체는 문제되는 프로퍼티만 단언

```tsx
// 나머지 필드까지 전부 체크 해제
const settings: Settings = {
  retryCount: 3,
  logger: customLogger,
  timeout: 1000,
} as any;

// 문제가 있는 곳만
const settings: Settings = {
  retryCount: 3,
  logger: customLogger as any,
  timeout: 1000,
};
```

객체 전체를 `as any` 하면 나머지 프로퍼티의 오타랑 타입 불일치도 같이 통과함.

---

### 아이템 44. `any` 타입을 구체적으로 변형해서 사용하기

#### `any`는 너무 큼

`any`는 모든 값임. 그런데 실제로 모르는 건 대개 **일부임.**
→ 배열인 건 아는데 원소를 모른다, 함수인 건 아는데 인자를 모른다 같은 느낌

그렇다면 아는 만큼은 타입에 남겨야 함

```tsx
function getLength(array: any) {
  return array.length; // 오타든 뭐든 다 통과, 반환 타입도 any
}

function getLength(array: any[]) {
  return array.length;
}
```

`any[]`로만 바꿔도

- 함수 본문의 `array.length`가 `number`로 체크됨
- 반환 타입이 `number`로 추론됨
- 호출할 때 인자가 배열인지 검사됨

#### 원소 타입이 안 중요하면 `unknown[]`

```tsx
function hasItems(list: any[]) {
  const first = list[0]; // any -> 여기서부터 다시 전염
  return list.length > 0;
}

function hasItems(list: unknown[]) {
  const first = list[0]; // unknown -> 쓰려면 좁혀야 함
  return list.length > 0;
}
```

`any[]`는 원소를 꺼내는 순간 다시 `any`가 됨. 원소를 실제로 안 쓸 거라면 `unknown[]`이 더 나음.

---

### 아이템 45. 함수 안으로 타입 단언문 감추기

- 함수의 모든 부분을 안전한 타입으로 구현하는 것이 이상적이지만, 불필요한 예외 상황까지 고려해 가며 타입 정보를 힘들게 구성할 필요는 없음.
- 함수 내부에는 타입 단언을 쓰고, 함수 외부로 드러나는 타입 정의를 정확히 명시하는 정도가 적절함.
- 프로젝트 전반에 위험한 단언문이 드러나 있는 것보다, 제대로 타입이 정의된 함수 안으로 감추는 것이 더 좋은 설계임.

```ts
function shallowObjectEqual<T extends object>(a: T, b: T): boolean {
  for (const [k, aVal] of Object.entries(a)) {
    if (!(k in b) || aVal !== (b as any)[k]) {
      // 단언이 없으면: 'string'은 'T'의 인덱스로 쓸 수 없다는 오류
      return false;
    }
  }
  return Object.keys(a).length === Object.keys(b).length;
}
```

`k in b`로 확인했으므로 논리적으로 안전하다.
-> 객체 순회와 단언문이 코드에 직접 들어가는 것보다 **별도의 함수로 분리**해 내는 것이 훨씬 좋은 설계다.
(`cacheLast`처럼 반환 함수를 `as unknown as T`로 단언하는 경우도 같은 패턴이다.)

- 안전하지 않은 단언이 필요하다면 올바른 시그니처를 가진 함수 안에 숨김.
- 구현부의 타입 오류를 고치려고 함수 시그니처를 타협하지 않음.
- 단언이 왜 타당한지 설명하고, 철저히 테스트함.

---

### 아이템 46. 타입을 모르는 경우 any 대신 unknown 사용하기

`any`가 위험한 이유는 두 성질을 동시에 갖기 때문이다.

1. 어떠한 타입이든 `any`에 할당 가능함.
2. `any`는 어떠한 타입으로도 할당 가능함.

한 집합이 다른 모든 집합의 부분집합이면서 동시에 상위집합일 수는 없으므로 타입 시스템과 상충하고, 타입 체커가 무용지물이 된다.

#### 타입을 모르는 값의 반환 타입

```ts
function parseYAML(yaml: string): any {
  /* ... */
}

const book = parseYAML(`name: Wuthering Heights`);
alert(book.title); // 오류 없음 -> 런타임에 undefined
book("read"); // 오류 없음 -> 런타임에 예외
```

함수의 반환값에 타입 선언을 강제할 수 없어, 호출한 곳에서 선언을 생략하면 사용되는 곳마다 오류가 난다.
차라리 `unknown`을 반환하는 것이 안전하다.

```ts
function safeParseYAML(yaml: string): unknown {
  return parseYAML(yaml);
}
const book = safeParseYAML(`...`);
alert(book.title); // ~~~~ 개체가 'unknown' 형식입니다
```

> 제네릭(`safeParseYAML<T>(yaml): T`)은 대안이 아니다. 결국 타입 단언과 같은 효과인데 위험이 숨겨져 더 나쁘다.

#### `unknown` : 원하는 타입으로 변환

그대로 쓰는 타입이 아니라 변환 후 사용한다.

1. 타입 단언 `as Book`
2. 사용자 정의 타입 가드 `val is Book`
3. `instanceof`

```ts
function isBook(val: unknown): val is Book {
  return (
    typeof val === "object" && val !== null && "name" in val && "author" in val
  );
}
```

결론!

- `unknown`은 `any` 대신 쓸 수 있는 타입 안전한 타입. **값이 있지만 타입을 모를 때** 사용한다.
- 사용자가 타입 단언, 타입 체크를 하도록 **강제**하려면 `unknown`을 쓴다.
- `{}`, `object`, `unknown`의 차이를 이해한다.

---

### 아이템 47. 몽키 패치보다는 안전한 타입 사용하기

#### 몽키 패치

이미 만들어진 객체에 나중에 속성이나 메서드를 덧붙이는 행위

JS에서는 그냥 되지만, TS는 **타입이 선언 시점에 확정된다**고 보기 때문에 나중에 붙인 속성을 전혀 모른다. 구조적 타이핑이라 해도 "없는 속성"은 읽을 수도 쓸 수도 없다.

```ts
document.monkey = "Tamarin";
//       ~~~~~~ Document 형식에 'monkey' 속성이 없습니다
```

#### 왜 문제인가?

타입 에러 자체보다도, 전역 객체나 DOM 같은 **공유 자원의 인터페이스를 마음대로 바꾼다**는게 문제다. 어디서 붙였는지 추적이 안 되고, 붙기 전에 접근하면 `undefined`인데 타입은 항상 있다고 말함.

### 해결 순서

- 애초에 안붙인다

상태는 상태를 담는 모듈 등에 두고, DOM/전역은 건드리지 않는다.

```ts
// state.ts
export const appState = { user: null as User | null };
```

- 타입 보강

서드파티 스크립트가 전역에 뭔가를 꽂는 상황은 실제로 있다. 이때는 `as any`가 아니라 보강으로 선언한다.

```ts
// global.d.ts
export {}; // 모듈로 만들어야 declare global이 동작

declare global {
  interface Window {
    dataLayer?: unknown[]; // 반드시 선택적으로
  }
}
```

보강의 단점은 **범위가 프로젝트 전체**라는 것. 특정 시점 이후에만 존재하는 값인데 타입상으로는 어디서나 존재하는 것처럼 보인다. `?`나 `| undefined`를 붙여서 "없을 수도 있음"을 타입에 새겨두는 게 최소한의 방어다.

- 단언을 함수 하나에 가둔다

단언을 없앨 수 없다면 최소한 **한 군데로 모아서** 나머지 코드는 깨끗하게 유지한다.

---

### 아이템 48. 무결성 함정 피하기

#### 무결성이란

모든 표현식의 **정적 타입이 런타임 값을 항상 포함**하면 그 타입 시스템은 무결하다. TS는 무결하지 않고, 그건 버그가 아니라 **의도된 설계**다.

완전한 무결성을 추구하면 기존 JS 코드를 거의 받아줄 수 없고, 매 줄마다 증명을 요구하게 된다. TS는 실용적으로 대부분 맞는 쪽을 택했다. 대신 어디가 뚫려 있는지는 알고 써야 한다.

#### 함정이 될 수 있는 것

- `any`

`any`는 자기 자리에만 머무르지 않고 반환값, 인자, 구조 분해를 타고 번진다.

-> 모르겠는 값은 `unknown`으로 받고 좁혀서 쓴다. `any`가 불가피하면 함수 하나 안으로 스코프를 제한한다.

- 타입 단언

`as`는 책임진다는 선언이지 검증이 아니다.

조건문으로 좁히거나, 값 자체를 검사하는 사용자 정의 타입 가드를 쓴다. 리터럴의 타입을 확인만 하고 싶은 거라면 단언 대신 `satisfies`를 쓴다.

- 인덱스 접근

배열과 인덱스 시그니처는 존재 여부를 검사하지 않는다.

```ts
const xs = [1, 2, 3];
const x = xs[10];
d;

const dict: Record<string, string> = { "007": "Bond" };
dict["008"].toUpperCase();
```

→ `noUncheckedIndexedAccess`를 켠다.

> **`strict`에 포함되지 않는다**
> TS 6.0부터 `strict`가 기본값이 됐지만, `noUncheckedIndexedAccess`는 여전히 `strict` 묶음 밖이다. `exactOptionalPropertyTypes`도 마찬가지. 둘 다 **명시적으로 켜야** 한다.
>
> ```jsonc
> {
>   "compilerOptions": {
>     "noUncheckedIndexedAccess": true,
>     "exactOptionalPropertyTypes": true,
>   },
> }
> ```

켜면 `xs[10]`이 `number | undefined`가 되어 매번 확인해야 한다.

- 이중변동

`strictFunctionTypes`는 함수 매개변수를 반공변으로 검사하지만, **메서드 문법으로 선언된 것에는 적용되지 않는다.**
배열의 `sort`, 이벤트 핸들러 등 기존 API 호환 때문에 남겨둔 허점?이다.

- 배열/객체의 공변성

`Hen[]`을 `Animal[]`로 넘기는 건 읽기만 할 때는 안전하지만 쓰는 순간 깨진다.

```ts
function addFox(animals: Animal[]) {
  animals.push(new Fox());
}
const hens: Hen[] = [new Hen()];
addFox(hens); // Hen[] 안에 Fox
```

수정하지 않는 매개변수는 `readonly`로 받는다.
그러면 `push` 자체가 막히고, 호출자도 안심할 수 있다. 값을 바꿔야 한다면 인자를 변형하지 말고 새 배열을 반환한다.

— 선택적 속성과 초과 속성 검사

초과 속성 검사는 **객체 리터럴을 직접 대입할 때만** 동작한다. 변수를 한 번 거치면 검사가 사라진다.

-> 선택적 속성을 습관적으로 늘리지 않는다. `type`, `value`, `age`처럼 충돌하기 쉬운 이름 대신 `ageInYears`, `formattedAge`처럼 의미를 담아 구분한다. `exactOptionalPropertyTypes`를 켜면 `age?: number`에 `undefined`를 명시적으로 넣는 것도 막힌다.

- 외부에서 들어오는 값

`JSON.parse`, `fetch().json()`, `localStorage`, 폼 입력, URL 쿼리는 전부 `any`이거나 거짓말이다.

```ts
const user = await res.json(); // any
const user: User = await res.json(); // 검증 0, 희망사항
```

-> 시스템 **경계에서는 타입이 아니라 런타임 검증**을 쓴다. Zod 등 Standard Schema를 따르는 라이브러리로 파싱하면 검증 결과에서 타입을 추론할 수 있다.

```ts
const User = z.object({ id: z.string(), age: z.number() });
const user = User.parse(await res.json()); // 여기서부터는 타입을 믿어도 된다
```

---

### 아이템 49. 타입 커버리지를 추적해 타입 안전성 유지하기

`noImplicitAny`를 켜도 `any`는 사라지지 않는다.

- 내가 명시적으로 쓴 `any`
- 서드파티 `.d.ts`에서 흘러 들어오는 `any`
- 제네릭 기본값이나 추론 실패로 생긴 `any`

#### type-coverage

```bash
npx type-coverage --detail --strict
```

- `--detail` : `any`인 심벌의 위치를 전부 출력
- `--strict` : 안전하지 않은 단언과 암묵적 `any`까지 포함해서 계산
- `// type-coverage:ignore-next-line` : 정당한 예외는 주석으로 표시

CI에 하한선을 걸어두는 것이 핵심이다.

```jsonc
// package.json
{ "typeCoverage": { "atLeast": 98, "strict": true, "ignoreCatch": true } }
```

#### typescript-eslint

커버리지 숫자는 "얼마나 뚫렸는가"를 보여주고, 린트 규칙은 "어디로 새어 들어오는가"를 짚어준다.

```
@typescript-eslint/no-unsafe-assignment
@typescript-eslint/no-unsafe-call
@typescript-eslint/no-unsafe-member-access
@typescript-eslint/no-unsafe-return
@typescript-eslint/no-unsafe-argument
```

처음부터 `any` 없이 시작한 프로젝트라면 얻는 게 적다. 다만 **직접 쓰지 않아도 유입된다**는 게 이 아이템의 요점이므로, 의존성이 많은 프로젝트라면 유입 감지기로 걸어둘 가치는 충분하다. JS에서 마이그레이션 중인 코드베이스라면 진행률 지표로 그대로 쓸 수 있다.
