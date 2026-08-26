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
