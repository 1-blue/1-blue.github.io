---
layout: post
title: 러닝 타입스크립트 7장
author: admin
date: 2023-03-16 22:23:00 +900
lastmod: 2023-03-16 22:23:00 +900
sitemap:
  changefreq: monthly
  priority: 0.5
categories: [Study, 러닝 타입스크립트]
tags: [러닝 타입스크립트]
image:
  path: https://www.hanbit.co.kr/data/books/B9711663545_l.jpg
  width: 400
  height: 250
  alt: 러닝 타입스크립트 교재 이미지
---

> 해당 포스트는 `러닝 타입스크립트` 7장을 읽고 정리한 포스트입니다.<br />책의 모든 내용을 작성하는 것이 아닌 주관적인 기준에 따라 필요한 정보만 정리했습니다.<br />
{: .prompt-info}

# 🍻 타입 별칭 vs 인터페이스
인터페이스와 타입 별칭은 거의 동일하게 동작합니다.<br />
하지만 미묘한 차이가 있고 그 차이를 이해하고 상황에 맞게 사용하는 것이 중요합니다.<br />

+ 인터페이스의 특징
  1. 인터페이스에서 타입 검사기가 더 빨리 동작함<br />
  1. 인터페이스는 오류 메시지를 더 쉽게 읽을 수 있음<br />
  1. 인터페이스는 속성 증가를 위해 [병합](/posts/Learning-TypeScript-7장/#-인터페이스-병합)할 수 있음<br />
  1. 인터페이스는 클래스의 타입 구조를 확인하는데 사용 가능 ( 8장 )<br />
  1. `readonly` 사용 가능

`1`과 `2`만 봐도 타입 별칭보다는 인터페이스를 사용하는 것이 더 좋은 것 같습니다.<br />
( 특별한 이유가 있지 않다면 인터페이스를 사용하는 것이 성능에서도 좋고 의미적으로도 좋은 것 같습니다. )<br />

## 0️⃣ 타입 별칭에서만 할 수 있는 것
그렇다면 인터페이스는 할 수 없고, 타입 별칭에서만 할 수 있는 것은 무엇이 있을까요?<br />

`(1)`처럼 타입의 유니언을 사용하는 경우에 인터페이스는 하나의 깊이를 더 들어가야만 구현할 수 있는 차이가 있습니다.<br />
하지만 타입 별칭을 사용하면 깊이 없이 바로 타입의 유니언을 사용할 수 있습니다.<br />

이런 특별한 경우를 제외한다면 인터페이스를 사용하는 것이 좋은 것 같습니다.<br />

```ts
interface XPos {
  x: number;
}
interface YPos {
  y: number;
}

interface IPos1 extends XPos, YPos {
  z: number;
}
interface IPos2 {
  // (1)
  pos: XPos | YPos;
  z: number;
}

// (1)
type TPos = { z: number } & (XPos | YPos);

const iPos1: IPos1 = {
  x: 1,
  y: 1,
  z: 1,
};
const iPos2: IPos2 = {
  pos: {
    x: 1,
    // y: 1,
  },
  z: 1,
};

const tPos: TPos = {
  x: 1,
  // y: 1,
  z: 1,
};
```

# 🥥 속성 타입

## 0️⃣ 선택적 속성
선택적 속성은 생략될 수 있습니다.<br />

```ts
interface Pos {
  x: number;
  y?: number;
}

const pos: Pos = {
  x: 1,
  // y: 1,
};
```

## 1️⃣ 읽기 전용 속성
속성 앞에 `readonly`를 사용하면 다른 값으로 설정될 수 없음(`(1)`)을 설정할 수 있습니다.<br />
`TypeScript`에서만 존재하고 인터페이스에서만 사용할 수 있습니다.<br />

```ts
interface Pos {
  readonly x: number;
}

const pos: Pos = {
  x: 1,
};

// Cannot assign to 'x' because it is a read-only property
pos.x = 10;
```

읽기 전용 속성과 읽기 전용이 아닌 속성을 서로 할당할 수 있습니다.<br />

```ts
interface ReadOnlyPos {
  readonly x: number;
}
interface Pos {
  x: number;
}

let readonlyPos: ReadOnlyPos = {
  x: 1
};
let pos: Pos = {
  x: 1
}

readonlyPos = pos;
// readonlyPos.x = 10;

pos = readonlyPos;
pos.x = 10;
```

아래 예시는 함수의 매개변수로 받는 객체에 `readonly`를 사용해서 객체를 안전하게 보호하는 예시입니다.<br />

`(2)`에서 선언한 변수는 `readonly`가 없는 객체 타입이고 `(3)`의 `read()`는 `readonly`인 프로퍼티를 갖는 객체를 매개변수로 받는 함수입니다.<br />
이 함수는 함수의 선언만 보고 매개변수가 변하지 않음을 보증할 수 있습니다.<br />
( 넣어주는 인자`(2)`가 `readonly`가 아니지만 보증할 수 있습니다. )<br />

```ts
interface Page {
  readonly text: string;
}
interface ReadHanlder {
  (page: Page): void;
}

// (3)
const read: ReadHanlder = (page) => {
  // (1) Error: Cannot assign to 'text' because it is a read-only property.
  page.text = "쿠키... 헤더에 담았지...";
};

// (2)
const pageIsh = {
  text: "첫 번째 페이지",
};

pageIsh.text += " + 수정";

read(pageIsh);
```


## 2️⃣ 함수와 메서드
`TypeScript`의 인터페이스에서는 함수 선언의 두 가지 방법을 제공합니다.<br />

+ 메서드 구문: `func(): void` 형태로 사용
+ 속성 구문: `func: () => void` 형태로 사용

```ts
interface Func {
  // 메서드 구문
  mFunc(): void;

  // 속성 구문
  aFunc: () => void;
}
```

+ 차이점
  1. 속성만 `readonly` 사용 가능
  1. [인터페이스 병합에서 다르게 처리](/posts/Learning-TypeScript-7장/#-인터페이스-병합)
  1. 일부 작업에서 다르게 처리 ( 15장 )

`this`를 참조할 수 있다는 것을 알고 있다면 메서드 함수 사용하고, 그 외의 경우에는 속성 함수 사용하는 것이 좋다고 합니다.<br />
대부분 `this`를 사용하지 않으니까 속성 함수를 사용하는 것이 좋은 것 같습니다.<br />

## 3️⃣ 호출 시그니처
호출 시그니처는 값을 함수처럼 호출하는 방식에 대한 타입 시스템 설명입니다.<br />
즉, 인터페이스로 함수의 타입을 구현하는 것을 의미합니다.<br />

메서드 구문에서 함수의 이름 즉, 프로퍼티의 이름만 제외하면 됩니다.<br />

```ts
/**
 * 함수면서 "count"를 프로퍼티를 갖고 있다는 의미
 */
interface FunctionWithCount {
  // 메서드 구문 -> func(): void
  (): void;
  count: number;
}

const keepsTrackOfCalls = () => {
  keepsTrackOfCalls.count += 1;
  console.log("count >> ", keepsTrackOfCalls.count);
};
keepsTrackOfCalls.count = 0;

const hasCallCount1: FunctionWithCount = keepsTrackOfCalls;

// Error: Property 'count' is missing in type '() => void' but required in type 'FunctionWithCount'.
const hasCallCount2: FunctionWithCount = () => {
  keepsTrackOfCalls.count += 1;
  console.log("count >> ", keepsTrackOfCalls.count);
};
// hasCallCount2.count = 0;
```

## 4️⃣ 인덱스 시그니처
객체가 임의의 키를 받고 특정 타입을 반환할 수 있음을 타입으로 정의하는 방법입니다.<br />

```ts
interface WordCounts {
  [key: string]: number;
}

const counts: WordCounts = {};

counts.apple = 0;
counts.blue = 1;

// Error: Type 'string' is not assignable to type 'number'.
counts.color = "red";
```

매우 유용해보이지만 인덱스 시그니처는 타입 안정성을 완벽하게 보장해주지 않습니다.<br />
( + 자동 완성 지원 X )<br />
`(1)`의 경우 존재하지 않는 프로퍼티지만 `Date` 타입으로 인식합니다.<br />
`string`인 어떤 `key`도 받을 수 있기 때문에 `string`인 `key`로 접근하면 검사하지 않고 있다고 판단하고 실행하게 됩니다.<br />

```ts
interface DatesByName {
  [key: string]: Date;
}

const pulishDates: DatesByName = {};

// (1) date: Date
const date = pulishDates.Beloved;
```

키/값을 쌍으로 저장하지만 어떤 키가 들어올지 예측할 수 없다면 `Map`을 사용하는 것이 더 안전합니다. ( 9장 )<br />

### 1. 속성과 인덱스 시그니처 혼합
인덱스 시그니처를 사용하면서 특정 속성을 받고 싶은 경우 아래처럼 사용하면 됩니다.<br />
단, 인덱스 시그니처의 타입과 특정 속성의 타입이 일치해야 합니다.<br />

```ts
interface Person {
  name: string;
  country: "kr" | "us";
  [key: string]: string;
}

const person1: Person = {
  name: "",
  country: "kr",
};
// Error: Property 'name' is missing in type '{ country: "us"; }' but required in type 'Person'
const person2: Person = {
  country: "us",
};
```

### 2. 숫자 인덱스 시그니처
`JavaScript`에서는 `number` 타입의 `key`를 가질 수 있지만, 암묵적으로 객체 `key`를 `string`으로 변환합니다.<br />

따라서 `number` 인덱스 시그니처는 `string` 인덱스 시그니처의 타입에 포함될 수 있어야 합니다.<br />
( `(1)`은 `string | null`이 `string`에 포함될 수 없기 때문에 오류 발생 )<br />

```ts
// (1) 'number' index type 'string | null' is not assignable to 'string' index type 'string'
interface MoreNarrowNumber {
  [key: number]: string | null;
  [key: string]: string;
}

// (2)
interface MoreNarrowString {
  [key: number]: string;
  [key: string]: string | null;
}
```

## 5️⃣ 중첩 인터페이스
인터페이스도 다른 인터페이스를 객체 타입의 속성으로 가질 수 있습니다.<br />

```ts
interface Author {
  name: string;
}

interface Nobel {
  author: Author;
  page: number;
}

const nobel: Nobel = {
  author: {
    name: ""
  },
  page: 0
}
```

# 🍛 인터페이스 확장
`extends`를 사용하며 인터페이스가 다른 인터페이스의 모든 멤버의 정의를 복사해서 선언하는 방법입니다.<br />

인터페이스 확장은 엔티티 타입이 다른 엔티티 타입의 모든 멤버를 포함하는 상위 집합을 나타내는 실용적인 방법입니다.<br />

```ts
interface Person {
  age: number;
}

interface Author extends Person {
  name: string;
}

const author: Author = {
  name: "",
  age: 0,
};
```

## 0️⃣ 재정의된 속성
인터페이스 확장을 이용해서 이미 존재하는 프로퍼티를 재정의할 수 있습니다.<br />
하지만 이미 존재하는 프로퍼티를 수정하는 경우에는 **더 구체적으로 재정의는 가능**하지만, **더 넓은 타입으로 확장하는 것은 불가능**합니다.<br />

`(1)`의 경우에는 `number | undefined`를 `number`로 재정의하기 때문에 더 구체적으로 수정하는 것이라 가능합니다.<br />
하지만 `(2)`의 경우에는 `number`를 `number | undefined`로 확장하기 때문에 불가능합니다.<br />

```ts
// (1)
interface Person1 {
  age: number | undefined;
}
interface Author1 extends Person1 {
  name: string;
  age: number;
}

// (2)
interface Person2 {
  age: number;
}
interface Author2 extends Person2 {
  name: string;
  age: number | undefined;
}
/**
 * Interface 'Author2' incorrectly extends interface 'Person2'.
 * Types of property 'age' are incompatible.
 *   Type 'number | undefined' is not assignable to type 'number'.
 *     Type 'undefined' is not assignable to type 'number
 */
```

## 1️⃣ 다중 인터페이스 확장
`,`를 기준으로 여러 인터페이스를 작성하면 다중 인터페이스로 확장도 가능합니다.<br />

```ts
interface MyString {
  s: string;
}
interface MyNumber {
  n: number;
}
interface MyValue extends MyString, MyNumber  {
  b: boolean;
}

const func = (v: MyValue) => {
  v.s;
  v.n;
  v.b;
}
```

# 🥗 인터페이스 병합
두 개의 인터페이스가 동일한 이름으로 동일한 스코프에 선언되는 경우 선언된 모든 필드를 포함하는 더 큰 인터페이스로 병합됩니다.<br />

```ts
interface Merged {
  s: string;
}
interface Merged {
  n: number;
}

const merged: Merged = {
  s: "",
  n: 0,
};
```

일반적으로 위와 같이 사용하기 보다는 외부에서 받은 패키지에 뭔가를 덧붙이는 경우 사용합니다.<br />

```ts
interface Window {
  myValue: string;
}

window.myValue;
```

## 0️⃣ 이름이 충돌되는 멤버
병합된 인터페이스는 같은 이름의 멤버를 여러 번 선언할 수 없습니다.<br />
만약에 사용한다면 나중에 병합된 인터페이스에서도 동일한 타입으로 선언해야 합니다.<br />

하지만 [메서드 방식](/posts/Learning-TypeScript-7장/#2%EF%B8%8F⃣-함수와-메서드)을 사용하는 경우 [함수 오버로드](/posts/Learning-TypeScript-5장/#%EF%B8%8F-함수-오버로드){:target="_blank"}로 인지하고 정삭적으로 동작합니다.<br />

```ts
// 속성 방식
interface Merged1 {
  func: (v: number) => string;
}
// Error: Subsequent property declarations must have the same type.
// Property 'func' must be of type '(v: number) => string',
// but here has type '(v: string) => string'.
interface Merged1 {
  func: (v: string) => string;
}

// 메서드 방식 ( 함수 오버로드 발생 )
interface Merged2 {
  func(v: number): string;
}
interface Merged2 {
  func(v: string): string;
}

const merged2: Merged2 = {
  func: (v: string | number) => "",
}
```

# 📮 레퍼런스
1. « 러닝 타입스크립트 7장 » ( 조시 골드버그 지음, 고승원 옮김, 한빛미디어, 2023 )

1. [1-blue - 함수 오버로드](/posts/Learning-TypeScript-5장/#%EF%B8%8F-함수-오버로드){:target="_blank"}