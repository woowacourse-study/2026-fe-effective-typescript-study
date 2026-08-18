## 아이템 43 any 타입은 가능한 한 좁은 스코프에서만 사용하기

`const pizza: any = getPizza();` 이 방식보다는
`eatSalad(pizza as any);` 이 방식이 조금 더 낫다

매개변수 안 스코프에서만 any가 되기 때문이다. 전자는 any로 선언한 것이 퍼지고 퍼져 큰 문제가 될 가능성이 높다.

객체 내부에 값 하나의 타입이 불분명하다면, 객체 전체를 any로 단언하지말고, 문제가 되는 타입만 any로 단언하자.

결국 any를 최대한 좁은 스코프에서만 쓰라는 제목이 핵심이다.

## 아이템 44 any를 구체적으로 변형해서 사용하기

같은 any를 사용해도 조금 더 구체적인 any가 존재한다.
무슨 타입이 오는지는 모르겠지만, 배열이라면 any[] 이런식으로,,

## 아이템 45 함수 안으로 타입 단언문 감추기

일단 여기서 설명하는건 any나 unknown을 함수에서 써야한다면, 함수를 사용하는 곳에서 계속 as 단언하지말고, 구현체에서 return할 때 1번만 쓰라는 이야기이다.

또한 타입 시스템이 논리적인 것을 체크할 수 없을 때 최대한 좁은 스코프에서 단언을 사용하는 것도 좋은 방법이다.
그리고 단언한 곳에는 꼭 주석을 작성해주도록 하자.
주석 뿐 아니라 단위 테스트도 꼼꼼히 작성하자

## 아이템 46 타입을 모르는 경우 any 대신 unknown 사용하기

```ts
function parseYAML(yaml: string): any {
  // ...
}

interface Book {
  name: string;
  author: string;
}
const book: Book = parseYAML(`
  name: Wuthering Heights
  author: Emily Brontë
`);
```

이렇게 타입 선언을 해주는 것보다 parseYAML 자체가 unknown을 반환하게 만드는 것이 더 안전하다.

unknown은 any 대신 쓸 수 있는 유용한 타입이다.
차이점이라면, unknown은 어떻게든 타입을 좁히지 않으면 사용할 수 없다는 것이다.

unknown의 친구중에는 `{}`타입이 있다. 이 타입은 unknown보다 좁다. null과 undefined를 제외한 모든 값이다.

결론은 any보다 unknown 쓰자
