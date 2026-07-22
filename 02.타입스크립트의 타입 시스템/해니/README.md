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
