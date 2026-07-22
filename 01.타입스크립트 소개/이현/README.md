- ### 아이템 4 - 구조적 타이핑에 익숙해지기

  JS는 덕 타이핑이고, 타입스크립트도 이를 그대로 모델링한다.

  ```js
   interface Vector2D {
     x: number;
     y: number;
   }
  ```

  이러한 Vector2D 인터페이스가 존재하고,

  ```js
   function calculateLength(v: Vector2D) {
     return Math.sqrt(v.x ** 2 + v.y ** 2);
   }
  ```

  길이를 계산하는 함수가 있을 때,

  ```js
   interface NamedVector {
     name: string;
     x: number;
     y: number;
   }
  ```

  name이 들어간 벡터를 만들고, calculateLength 메서드의 파라미터로 NamedVector 인스턴스를 만들어 넣어주면 잘 작동한다.
  이것이 바로 덕 타이핑이다.

  x와 y가 있기 때문에 문제없이 작동하는 것이다. 하지만 우리는 Vector2D와 NamedVector와의 관계를 전혀 선언하지 않았다.
  JS의 덕타이핑을 모방한 것이 TS의 구조적 타이핑이다.

  차이점이라면 덕타이핑은 JS의 런타임에 작동한다는 것?

  `타입은 열려있다!!`
  -> ❗️ 함수의 매개변수 타입을 지정할 때 정확히 그 타입만 가지는 매개변수가 들어올 것이라고 생각하지 말자

  ````js
   interface Vector3D {
     x: number;
     y: number;
     z: number;
   }

   function calculateLengthL1(v: Vector3D) {
     let length = 0;
     for (const axis of Object.keys(v)) {
       const coord = v[axis];
       //            ~~~~~~~ Element implicitly has an 'any' type because ...
       //                    'string' can't be used to index type 'Vector3D'
       length += Math.abs(coord);
     }
     return length;
   }

   // 왜 여기서 오류가 날까? calculateLengthL1의 매개변수로 오는 v가 x,y,z만 가지고 있을 것이라고 함부로 판단할 수 없다.

   const vec3D = {x: 3, y: 4, z: 1, address: '123 Broadway'};
   calculateLengthL1(vec3D);  // OK, returns NaN

   // 이것도 작동할 것이기 때문에 calculateLengthL1의 v[axis] 구문에서 coord는 number라는 값을 보장받지 못하는 것이다.
   ```
  ````

  구조적 타이핑은 클래스 개념에도 적용된다. num이라는 인스턴스를 가지고, 생성자가 있는 클래스를 받는 자리에 num이라는 값을 가지는 객체가 들어갈 수 있는 이유도 구조적 타이핑 덕분이다.

  구조적 타이핑은 테스트에 도움이 된다. 인터페이스만 작성해놓으면, 나머지 세부구현은 구조적 타이핑으로 인해 어떻게 구현해도 상관 없다.

### 아이템 5 any 타입 지양하기

any 타입을 사용하면

- 런타임에 에러가 생길 수 있다
- 자동완성이 제공되지 않는다
- any 타입은 쓰지 말자
