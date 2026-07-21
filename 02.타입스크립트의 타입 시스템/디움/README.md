## 아이템 6

### 타입스크립트 서버는 자동으로 켜지는 것일까?
-> VS Code 자체에 TypeScript가 내장되어 있어서 자동으로 켜진다.

VS Code 설치
→ 내장 TypeScript 언어 서비스가 있음
→ .ts/.tsx 파일을 열면 tsserver 자동 실행

프로젝트에 typescript 설치
→ 프로젝트에 맞는 TypeScript 버전 사용 가능
→ tsc 명령어와 빌드·타입 검사에도 사용

## 아이템 7

**할당 가능한 값의 집합 = 타입**

### & 연산자
& 연산자는 두 타입의 교집합을 계산한다.
이때 타입의 교집합은 **속성 이름의 교집합**이 아니라, 그 타입에 할당 가능한 **값의 교집합**이다.

Q. 구조적 타이핑의 원리가 들어간걸까?

A. `&` 자체가 구조적 타이핑의 원리를 따르는 연산자라기보다, “두 타입을 동시에 만족”시키는 연산자이고, 객체 타입에 적용될 때 TypeScript의 구조적 타이핑 때문에 속성이 합쳐져 보이는 것이다.

객체 타입에서는 타입 이름보다 **값이 어떤 구조를 가졌는지**를 기준으로 판단한다.
즉,  `A & B`는 구조적 타이핑에 따라 **A의 구조와 B의 구조를 모두 만족하는 타입**이다.

``` ts
interface Person {
	name: string;
}

interface Lifespan{
	birth: Date;
	death?: Date;
}

type PersonSpan = Person & Lifespan;
```

당연히 `name, birth, death`  세 가지보다 더 많은 속성을 가지는 값도 `PersonSpan` 타입에 속한다. 

-> TS는 구조적 타이핑을 따르기 때문

### 템플릿 리터럴 타입
문자열을 조합해서 새로운 문자열 타입을 만드는 기능

-> 자바스크립트의 템플릿 리터럴이랑 모양이 비슷하다

``` ts
type EventName = `on${string}`;

const a: EventName = 'onClick'; // 가능
const b: EventName = 'onChange'; // 가능
const c: EventName = 'click'; // 오류
```

### 핵심
- 타입은 값의 집합이다.
- & 연산자는 구조적 타이핑의 원리를 생각해보면 결과를 이해할 수 있다.

### 타입 계층도
<img width="780" height="270" alt="image" src="https://github.com/user-attachments/assets/c0aa6478-fba0-42c0-b262-ff2b824c95d8" />
