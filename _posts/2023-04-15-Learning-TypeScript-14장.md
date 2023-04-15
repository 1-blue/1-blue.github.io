---
layout: post
title: 러닝 타입스크립트 14장 ( 구문 확장 )
author: admin
date: 2023-04-15 00:00:00 +900
lastmod: 2023-04-15 23:56:00 +900
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

> 해당 포스트는 `러닝 타입스크립트` 14장을 읽고 정리한 포스트입니다.<br />책의 모든 내용을 작성하는 것이 아닌 주관적인 기준에 따라 필요한 정보만 정리했습니다.<br />
{: .prompt-info}

`JavaScript`의 구문을 확장한 **`TypeScript`만의 구문**에 대한 내용입니다.<br />
구문 확장은 기존 언어와 충돌될 수 있기 때문에 나쁜 사례로 간주합니다.<br />

# 🥚 클래스 매개변수 속성
매개변수 속성은 `TypeScript`만의 구문으로 `JavaScript`에서는 사용할 수 없습니다.<br />

`(1)`은 일반적인 구문이고, `(2)`는 매개변수 속성 구문입니다.<br />
매개변수 속성을 사용하면 코드를 단축시킬 수 있지만, 실제로 사용하는 것에 대한 의견이 갈린다고 합니다.<br />
`JavaScript`로 변환하면 코드가 늘어나는 특이한 문법이라고 생각해서 저는 사용하지 않을 것 같습니다.<br />

```ts
// (1)
class Person1 {
  public name: string;

  constructor(name: string) {
    this.name = name;
  }
}

// (2)
class Person2 {
  constructor(public name: string) {
    this.name = name;
  }
}
```

# 🐣 실험적인 데코레이터
TODO: 이게 무슨 말이야... 🥲

# 🐔 열거형
연관된 값 집합을 나타내는 문법입니다.<br />
타입이 `number` or `string`인 리터럴 값들을 갖는 객체를 생성하기 위한 문법입니다.<br />

```ts
enum StatusCode {
  A = 500,
  B = 400,
  C = 200,
}

let statusCode: StatusCode;
statusCode = StatusCode.A;
statusCode = 400;

// Error: Type '300' is not assignable to type 'StatusCode'.
statusCode = 300;
```

열거형은 `JavaScript` 런타임 구문 구조를 추가하지 않는다는 `TypeScript`의 만트라를 위반하기 때문에 자주 사용하지 않는다고 합니다.<br />
( 런타임에 새로운 객체를 생성 )<br />

## 0️⃣ 자동 숫잣값
열거형은 명시적으로 값을 지정하지 않으면 `0`부터 시작해서 순서대로 값이 들어가게 됩니다.<br />

```ts
enum StatusCode {
  A,
  B,
  C,
}

StatusCode.A; // 0
StatusCode.B; // 1
StatusCode.C; // 2
```

만약 초깃값을 숫자로 지정하면 해당 숫자부터 순서대로 값이 들어갑니다.<br />

```ts
enum StatusCode {
  A,
  B = 100,
  C,
}

StatusCode.A; // 0
StatusCode.B; // 100
StatusCode.C; // 101
```

## 1️⃣ 문자열값을 갖는 열거형
문자열도 들어갈 수 있지만 문자열 바로 뒤에 빈 값을 넣으면 안됩니다.<br />

```ts
enum StatusCode {
  A = "a",
  B = 100,
  C,
  D = "d",
  E, // Error: Enum member must have initializer
}

StatusCode.A; // "a"
StatusCode.B; // 100
StatusCode.C; // 101
StatusCode.D; // "d"
```

## 2️⃣ const 열거형
`JavaScript`로 변환할 때 객체의 정의와 속성 조회를 생략합니다.<br />

기존 열거형은 `JavaScript`로 변환하는 경우 런타임에 새로운 객체가 생성됩니다.<br />
하지만 const 열거형은 객체가 생성되지 않고 리터럴 값으로 들어가게 됩니다.<br />
( `preserveConstEnum` 옵션을 사용하면 열거형 객체는 생성하되 리터럴 값을 사용합니다. )<br />

```ts
const enum StatusCode {
  A = "a",
  B = 100,
  C,
}

StatusCode.A; // "a"
StatusCode.B; // 100
StatusCode.C; // 101

// 변환된 이후
"a" /* StatusCode.A */; // "a"
100 /* StatusCode.B */; // 100
101 /* StatusCode.C */; // 101
```

# 🍗 네임스페이스
네임스페이스는 특정 이름으로 묶인 하나의 큰 객체입니다.<br />
실제로 변환된 것을 보면 네임스페이스와 동일한 이름의 객체를 인자로 받는 즉시 실행 함수를 실행합니다.<br />

```ts
namespace MySpace {
  const v = 10;
  const func = () => console.log(v);
}

namespace MySpace {
  const vv = 10;
}

// 컴파일 후
var MySpace;
(function (MySpace) {
  const v = 10;
  const func = () => console.log(v);
})(MySpace || (MySpace = {}));
(function (MySpace) {
  const vv = 10;
})(MySpace || (MySpace = {}));
```

## 0️⃣ 네임스페이스 내보내기
네임스페이스의 내부에서 정의한 것을 사용하기 위해서는 `export`를 사용해야 합니다.<br />
`export`를 사용하면 즉시 실행 함수의 인자로 넘겨준 객체에 값을 넣게 됩니다.<br />

따라서 이름이 겹치지 않고 외부에서 그대로 사용할 수 있게 됩니다.<br />
만약 네임스페이스의 이름이 겹친다면 같은 이름에 추가되게 됩니다.<br />
또한 다른 파일에 같은 이름의 네임스페이스가 선언되어도 상관없이 같은 객체에 속성으로 들어가게 됩니다.<br />

```ts
namespace MySpace {
  export const v = 10;
  export const func = () => console.log(v);
}
namespace MySpace {
  export const vv = 10;
}
MySpace.v; // 10

// 컴파일 후
var MySpace;
(function (MySpace) {
  MySpace.v = 10;
  MySpace.func = () => console.log(MySpace.v);
})(MySpace || (MySpace = {}));
(function (MySpace) {
  MySpace.vv = 10;
})(MySpace || (MySpace = {}));
MySpace.v; // 10
```

## 1️⃣ 중첩된 네임스페이스
`.`을 이용해서 네임스페이스를 중첩시킬 수 있습니다.<br />

```ts
namespace MySpace.Nested {
  export const v = 10;
  export const func = () => console.log(v);
}
MySpace.Nested.v; // 10

// 컴파일 후
var MySpace;
(function (MySpace) {
  var Nested;
  (function (Nested) {
    Nested.v = 10;
    Nested.func = () => console.log(Nested.v);
  })(Nested = MySpace.Nested || (MySpace.Nested = {}));
})(MySpace || (MySpace = {}));
MySpace.Nested.v; // 10
```

## 2️⃣ 타입 정의에서 네임스페이스
TODO: 무슨 말이야... 🥲

## 3️⃣ 네임스페이스보다 모듈을 선호함
네임스페이스로 구조화된 타입스크립트 코드는 웹팩과 같은 번들러에서 [트리쉐이킹](/posts/Tree-Shaking){:target="_blank"}하는 것이 쉽지 않습니다.<br />

```ts
// v.ts
export const name = "name";

// a.ts
import { name } from "./a.ts"
```

## 4️⃣ 타입 전용 가져오기와 내보내기
`import`와 `export`의 앞에 `type`을 붙여서 타입 전용으로 가져오기와 내보내기를 할 수 있습니다.<br />
타입으로 가져오거나 내보내면 런타임 시 없어져야 하는 것을 트랜스파일러에게 명시할 수 있습니다.<br />

```ts
// a.ts
const myValue = 10;
type MyType = string;

export { myValue, type MyType };

// b.ts
import { myValue, type MyType } from "./a.ts";
```

# 📮 레퍼런스
1. « 러닝 타입스크립트 14장 » ( 조시 골드버그 지음, 고승원 옮김, 한빛미디어, 2023 )