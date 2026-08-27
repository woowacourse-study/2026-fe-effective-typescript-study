## 아이템 59

```ts
type Shape =
  | { kind: 'circle'; radius: number }
  | { kind: 'square'; size: number };

function getArea(shape: Shape): number {
  switch (shape.kind) {
    case 'circle':
      return Math.PI * shape.radius ** 2;

    case 'square':
      return shape.size ** 2;
  }
}
```

모양에 따라 넓이를 구하는 함수이다.

```ts
type Shape =
  | { kind: 'circle'; radius: number }
  | { kind: 'square'; size: number }
  | { kind: 'triangle'; width: number; height: number };
```

`triangle`이 추가된 경우 기존 `getArea`를 수정해야 한다.

모든 switch를 지난 후 마지막에는 `never` 타입만 남는다 -> 이 점을 이용해 빠진 것을 검사하자

새로운 경우를 추가하면 기존 코드가 자동으로 깨지기 때문에 검사가 가능하다.

## 아이템 60

```ts
const obj = {
  one: 'uno',
  two: 'dos',
  three: 'tres',
};
for (const k in obj) {
  const v = obj[k];
}
```

k의 타입은 string이지만 obj 객체에는 'one', 'two', 'three'만 존재하기 때문에 오류가 발생한다.

이때 `as keyof typeof obj` 타입 단언을 사용하면 오류는 해결되지만, 타입 단언은 그저 타입 체크를 무효화하기 위한 수단이기 때문에 다른 방법을 사용해야 한다.

k의 타입이 `string`으로 추론되는 이유는 **구조적 타이핑**때문이다. 다른 속성이 존재할 수 있기 때문에 `string`으로 추론하는 것이다.

타입 문제 없이 객체의 키와 값을 순회하고 싶다면 `Object.entries`를 사용하면 된다.

```ts
function foo(abc: ABC) {
  for (const [k, v] of Object.entries(abc)) {
    //        ^? const k: string
    console.log(v);
    //          ^? const v: any
  }
}
```

키와 값의 타입을 다루려면 객체보다는 `Map`타입을 사용하면 더욱 간단하다.

## 아이템 61

객체와 해당 객체를 사용해 그리는 함수가 있을 때
1. 변경될 때마다 그리기
2. 새로운 값과 비교하기
둘 다 이상적인 방법이 아니다.

새로운 속성이 추가될 때 개발자가 직접 함수를 고치도록 유도하는 것이 좋다.

-> 타입 체커가 오류를 발생시키도록 하자

실패에 열림(실패해도 허용할건지)과 실패에 닫힘(실패하면 거부할거지) 중 어떤 것에 해당하는지 인지하는 것이 중요하다.
