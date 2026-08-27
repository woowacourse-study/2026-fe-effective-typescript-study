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

## 아이템 60 객체를 순회하는 노하우 이해하기

객체를 순회할 때는 `for-of`와 `Object.entries`를 사용할 수 있다.
Object.entries는 타입 별로 신경 안쓰고 단순히 객체의 키와 값을 순회할 때 쓴다 -> 안정성이 조금 떨어지더라도 에러는 안 냄
대신 안정성을 조금 더 신경쓰려면 for-of를 쓰면 된다.

for-of 내부에서 ['a','b','c'] as const;처럼 key의 타입을 좁혀서 사용할 수 있다.
하지만 이건 개발자가 key를 잘 일치시켜줘야한다.

정리해보면 불변 객체를 순회할 때는 for of를 쓰고, 값이 변하는 객체를 순회할 때는 Object.entries를 쓰면 된다.
하지만 Object.entries도 키와 값의 타입을 다루기는 까다롭기 때문에 Map이 더 적절한 방안이 될 수 있다.

## 아이템 61 레코드 타입을 사용해 값 동기화하기

> 어떤 타입에 속성이 추가됐을 때, 그 타입과 관련된 다른 코드도 반드시 같이 수정되어야 한다면, Record<keyof T,...>으로 컴파일러가 그 수정을 강제하게 만들라는 의미

열린 실패: 새로운 상황이 추가되었을 때 더 넓게 허용하는 방향으로 동작함

실제로는 아래와 같이 써볼 수도 있음

```ts
const messages: Record<RequestStatus, string> = {
  idle: "대기 중",
  loading: "불러오는 중",
  success: "완료",
  error: "오류가 발생했습니다",
};
```

이렇게 하면, 4가지 상태를 빼먹지 않을 수 있음!
