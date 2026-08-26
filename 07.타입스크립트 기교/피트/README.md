## TS스터디 27일차: 아이템 59 (7장)

### 아이템 59: 완전성 체크를 위해 never 타입 사용하기

정적 타입 분석은 발생하면 안 되는 일을 찾아낼 때 사용할 수 있는 훌륭한 방법이다.

반대로 어떤 행위를 반드시 해야 하지만, 하지 않아서 발생하는 오류도 존재한다.

TS 자체만으로는 위 경우를 모두 잡기 힘듦

완전성 체크 기법으로 해결!

`switch`문의 `default`에서 값이 `never` 타입인지 검사하고, 이후 코드가 변경됐을 때 여전히 `never`인지 확인하는 방법이 완전성 체크 기법이다.

```ts
function assertNever(value: never): never {
  throw new Error(`처리하지 않은 값: ${value}`);
}

function getArea(shape: Shape): number {
  switch (shape.kind) {
    case 'circle':
      return Math.PI * shape.radius ** 2;

    case 'square':
      return shape.size ** 2;

    default:
      return assertNever(shape);
  }
}
```

`Shape`에 새로운 종류가 추가됐는데 `case`문을 작성하지 않으면, `default`에 도달한 `shape`가 더 이상 `never`가 아니게 되므로 타입 오류가 발생한다.

안전성을 높이기 위해서 반환 타입 명시와 `never` 검사, `satisfies` 연산자 등을 사용하는 것이 좋다.

> 한 유니온의 모든 경우를 검사하려면 `never`를 활용한다.  
> 함수의 반환 타입도 명시해 누락된 반환 경로를 검사한다.
