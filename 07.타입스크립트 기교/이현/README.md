## 아이템 59 완전성 체크를 위해 never 타입 사용하기

switch-case를 사용할때 혹시라도 누락된 케이스가 있는지 확인하기 위해서 never타입을 사용할 수 있다.

never타입은 공집합이기 때문에 switch-case 최하단으로 내려왔을 때 그 값이 never가 아니라면 case가 누락된 것이다.

책에서는

```ts
function assertUnreachable(value: never): never {
  throw new Error(`Missed a case! ${value}`);
}
```

이런 함수를 switch-case의 default:에 배치하여서 런타임에도 체크 가능하게끔 해놨다.

이런 패턴은 반환값이 있는 경우 조금 더 빛을 발한다 따라서 함수의 반환값에 원하는 값을 선언해놓자
그래야 never 패턴이 의마기 생긴다.

```ts
default:
const exhaustiveCheck: never = shape;
// ~~~~~~~~~~~~~~~ Type 'Line' is not assignable to type 'never'.
throw new Error(`Missed a case: ${exhaustiveCheck}`);
```

```ts
    default:
      shape satisfies never
      //    ~~~~~~~~~ Type 'Line' does not satisfy the expected type 'never'.
      throw new Error(`Missed a case: ${shape}`);
```

assertUnreachable 외에 이렇게 2가지 방식으로도 체크가 가능하다

> 가위바위보 같은 모든 경우의 수를 따져야할 때도 never가 빠진 경우의 수를 체크해주는 역할을 해서 아주 유용하다
