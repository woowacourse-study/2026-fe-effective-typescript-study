## 아이템 65 devDependencies에 typescript와 @types 추가하기

peerDependencies: 런타임에 필요하긴 하지만, 의존성을 직접 관리하지 않는다.
-> npm에서 내 프로젝트를 다운받아 사용하는 사람들한테 버전을 맡기는 느낌임.

이걸 쓰는 이유는 react를 여러 라이브러리가 필요로 해서 여러 라이브러리들이 react를 각각 가지고 있으면 여러 react를 서비스에서 사용하게 되기 때문!!

따라서 peer로 하나의 react 인스턴스만 만들어서 공유하는 것이 중요하다.

일반적으로 ts는 런타임에는 필요없기 때문에 devDependencies에 추가!
ts를 일반 의존성으로 추가해도 되지만 그렇게 했을 때의 단점이 존재한다.

1. 팀원 모두가 항상 동일한 ts를 설치한다는 보장이 없다.
2. 프로젝트를 셋업할 때 별도의 단계가 추가된다.

따라서 devDependencies에 넣는게 좋다.
@types/lodash, @types/react 등의 타입 라이브러리는 devDependencies에 있어야된다!
그 원본이 일반 Dependencies에 있더라도 타입 정의는 dev에 있어야한다.

@types를 devDependencies에 넣으면 추가 장점도 얻을 수 있음!

1. 앱에 서버 컴포넌트가 존재하는 경우 npm install --production을 실행할 수 있다. 그리고 더 경량화된 이미지가 되고, 이미지를 가동하는 시간도 더욱 빨라질 것이다.

-> devDependencies이므로 프로덕션에서는 다 빠지는 것임! 그래서 성능 향상!

2.자동 의존성 업데이트 도구를 사용하는 경우 의존성에 대한 우선순위를 지정할 수 있다. -> 이거 신경써야함!

```md
react dependencies
axios dependencies
zod dependencies ← 런타임 validation 한다면

typescript devDependencies
eslint devDependencies
prettier devDependencies
vitest devDependencies
@types/node devDependencies
@types/react devDependencies
```

이렇게 해놓으면 devDependencies 애들은 후순위로 밀리는 것임!

## 아이템 66 타입 선언과 관련된 세 가지 버전 이해하기

TS는 안 그래도 복잡한 의존성 관리를 더 어렵게 한다. 왜냐하면 3가지의 버전을 고려해야하기 때문이다.

1. 라이브러리 버전
2. @types 버전
3. 타입스크립트 버전

3개중 한개라도 맞지 않으면 이상한 오류가 발생한다.

라이브러리 버전과 @types버전이 똑같더라도 그 2개가 잘 맞지 않을 수 있다.

내 프로젝트에서 사용하는 타입스크립트 버전이 나머지 2개와 맞지 않을 수도 있으니 유의해야한다.

요즘에는 타입 선언 자체도 번들링에 포함시키는 경우가 있다. 하지만 이는 4가지 문제를 발생시킨다.

1. 번들된 타입 선언에 버그가 있어도 타입만 따로 교체하기 어렵다. @types/foo처럼 타입이 별도 패키지면 라이브러리 본체는 그대로 두고 타입 선언 버전만 바꿀 수 있는데, 번들 타입은 라이브러리 패키지에 묶여 있으니까 그 선택권이 없다.

2. 라이브러리가 다른 라이브러리의 타입 선언에 의존할 때 문제가 생긴다. 예를 들어 foo가 자기 .d.ts 안에서 bar의 타입을 참조하면, foo를 설치한 사용자도 결국 bar의 타입 선언을 필요로 한다. 그런데 그 의존성이 devDependencies에만 있으면 라이브러리를 배포한 뒤 소비자에게는 설치되지 않아서 타입 에러가 날 수 있다. 반대로 dependencies에 넣자니 JavaScript 사용자에게는 필요 없는 타입 패키지까지 설치하게 되는 문제가 생긴다.

3. 과거 버전용 타입 선언을 유지하기 어렵다. 라이브러리 본체가 여러 버전으로 배포되어 있고, 각각 타입 정의가 조금씩 달라야 할 수 있다. DefinitelyTyped는 같은 라이브러리의 여러 버전에 대한 타입 선언을 별도로 관리할 수 있는데, 번들 방식은 기본적으로 그 특정 라이브러리 버전에 타입 하나가 딱 붙어 있는 구조라서 과거 버전 호환 타입을 관리하기가 더 불편하다.

4. 타입 선언 수정 주기가 라이브러리 릴리스 주기에 묶인다. 타입 오타 하나, 새 TypeScript 버전에서 생긴 호환성 문제 하나를 고치려고 해도 라이브러리 새 버전을 다시 배포해야 한다. 반면 @types/\*는 타입 선언만 따로 빠르게 패치할 수 있고, 커뮤니티가 독립적으로 유지보수할 수도 있다.

공식 권장사항은 라이브러리가 타입스크립트로만 작성된 경우에만 타입 선언을 라이브러리에 포함한다.
JS로 작성되어 있다면, DefinitelyTyped에 올리는 것을 추천한다.

결론은 3가지 버전을 잘 호환시키자는 것이다.

## 아이템 67 공개 API에 등장하는 모든 타입 export하기

라이브러리 사용할때 해당 라이브러리가 타입 export를 제대로 안해놨다면 ReturnType이나 Parameters 제네릭 써서 추출해서 사용해라.

## 아이템 68 API 주석에 TSDoc 사용하기

API에 주석을 달아야한다면 //(인라인)으로 달지말고 /\*\* \*/ 으로 달아야한다. 그래야지 이 주석정보를 띄워준다.
다만 타입 정보를 주석에 달지는 말아라.

폐기되었다면 @deprecated를 달아라

## 아이템 69 콜백 함수에서 this에 대한 타입 제공하기

js에서 this가 원래 어떻게 동작하는지부터 알아보자

함수가 어디에 있냐보다, 어떻게 호출했냐가 더 중요하다. -> this를 사용하는 메서드 자체가 어디서 호출되었는지가 더 중요한거임

그래서 this를 사용하는 메서드를 호출했는데, 인스턴스가 없다? (다른 변수에 담겨서 사용되는 경우 등)
그러면 this가 undefined가 되는 것이다.

그래서 JS는 .call()이라는 프로토타입 레벨의 메서드를 지원한다.
call의 인자로 들어간 애는 해당 함수의 this가 된다. `method.call(c)` -> 이런식으로

```ts
class ResetButton {
  render() {
    return makeButton({
      text: "Reset",
      onClick: this.onClick,
    });
  }

  onClick() {
    alert(`Reset ${this}`);
  }
}
```

이 예제를 한번 보자.
onClick에 this.onClick이 있다. 그래서 나중에 ResetButton이 makeButton하고 onClick하면 잘 될 것 같지만, 실제로는 그렇지 않다.
this.onClick을 전달한게 아니라, onClick함수 자체를 전달한 것이기 때문이다.

여기서는 해결책을 2가지 제시한다.

방법1. bind 메서드를 사용하는 것

`onClick: this.onClick.bind(this)` 이렇게 하면, 해당 함수가 나중에 어디서 호출되더라도 this를 지금의 this로 고정한다.
따라서 문제가 해결된다.

방법2. 화살표 함수를 사용하는 것

```ts
class ResetButton {
  onClick = () => {
    alert(`Reset ${this}`);
  };
}
```

화살표 함수에는 자기 자신의 동적인 this가 존재하지 않는다. 대신에 자신이 `만들어진` 바깥쪽 this를 그대로 사용한다.

따라서 자연스레 ResetButton이 this가 되는 것이다.

이걸 TS에서는 아래와 같이 사용할 수 있다.

```ts
function addKeyListener(
  el: HTMLElement,
  listener: (this: HTMLElement, e: KeyboardEvent) => void,
) {
  el.addEventListener("keydown", (e) => listener.call(el, e));
}
```

이렇게 listener의 인자로 this의 타입을 정해놓는 것이다.
그리고 그걸로 this바인딩을 해주는 패턴이다.

이게 바로 this에 어떤 것이 와야하는지 정해주는 TS에서만 가능한 패턴이다.

> 레벨1 미션에서 이 문제를 실제로 겪어봐서 더 체감된다. 그때도 화살표 함수로 전부 바꿨던 경험이 있다.

## 아이템 70 의존성 분리를 위해 미러 타입 사용하기

내 함수가 외부 라이브러리 타입의 아주 작은 일부만 필요하다면,
그 외부 타입을 공개 API에 그대로 노출하지 말고 필요한 구조만 직접 타입으로 만들어서 의존성을 끊어라.

Node.js의 Buffer라는 친구를 사용하고 싶음. 근데 내가 실제로 사용하는 것은 Buffer의 수만은 메서드중 하나의 메서드뿐이다.
그런데 어떤 메서드의 인자로 Buffer type이 와야된다고 해놓으면, 문제가 발생한다.

바로 @types/node를 설치하고 Buffer type을 가져와야 하는 것이다.

근데 이 의존성은 겁나 길고, Buffer 내용도 너무 많아서 낭비가 훨씬 심한 의존성이 될 수 있다.
따라서 TS에서는 구조적 타이핑을 이용해서 Buffer도 들어올 수 있도록 인자를 좁히면 된다. (내가 직접 Buffer와 호환되는 인터페이스를 1개 만들라는 것이다.)

Buffer도 들어올 수 있는 커스텀 타입을 만드는 것을 바로 Mirror Type이라고 하는 것 같다.
구조적 타이핑을 이용한 인터페이스 설계이다.

다만 주의할점은 Buffer안에 있는 메서드를 모두 사용한다? 그러면 @types/node를 받는 것이 나을 수도 있다.
어디까지나 mirror type을 만드는 것은 외부 라이브러리의 극히 일부 타입만 사용할 때이다.
