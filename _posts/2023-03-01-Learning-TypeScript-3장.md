---
layout: post
title: 러닝 타입스크립트 3장
author: admin
date: 2023-03-01 18:01:00 +900
lastmod: 2023-03-08 19:29:00 +900
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

> 해당 포스트는 `러닝 타입스크립트` 3장을 읽고 정리한 포스트입니다.<br />
{: .prompt-info}

# 🧬 유니언 ( union )
값에 허용된 타입을 두 개 이상의 가능한 타입으로 확장하는 것을 의미합니다.<br />

타입을 확장했기 때문에 더 많은 타입이 들어갈 수 있도록 유연하긴 하지만 사용에 제한이 걸립니다.<br />
유니언 타입 모두가 갖고 있는 공통된 멤버에만 접근할 수 있습니다.<br />

```ts
// myValue1: string | number
let myValue1: string | number;

// myValue2: string | number
let myValue2 = Math.random() > 0.5 ? "Aatrox" : 2;

// "string"과 "number" 모두 갖고 있는 메서드
myValue2.toString();

// Error: Property 'toFixed' does not exist on type 'string | number'.
myValue2.toFixed(); // "number"만 갖고 있는 메서드
```

# 🛤️ 내로잉 ( narrowing )
값에 허용된 타입들(유니언)을 좁히는 것을 의미합니다.<br />
즉, 여러 타입들중에서 더 구체적인 타입인 것을 타입 체커에게 알려주는 것입니다.<br />

## 0️⃣ 값 할당을 통한 내로잉
유니온 타입의 경우 값 할당을 통해서 타입을 좁힐 수 있습니다.<br />

```ts
let myValue: number | string = "Aarox";

// Error: Property 'toFixed' does not exist on type 'string'.
myValue.toFixed();
```

## 1️⃣ 조건 검사를 통한 내로잉
조건문을 통해서 유니온 타입을 좁힐 수 있습니다.<br />

```ts
let myValue = Math.random() > 0.5 ? "Aatrox" : 2;

// 타입 좁히기
if (myValue === 2) {
  // myValue: 2
  myValue.toFixed();
}
// 타입 좁히기
if (myValue === "Aatrox") {
  // myValue: "Aatrox"
  myValue.toUpperCase();
}
```

## 2️⃣ typeof 검사를 통한 내로잉
`typeof` 연산자를 이용하면 조금 더 넓게 타입을 좁힐 수 있습니다.<br />
단순한 조건 검사를 통한 내로잉보다 더 범용적으로 사용하는 방법입니다.<br />

```ts
let myValue = Math.random() > 0.5 ? "Aatrox" : 2;

myValue.toString();

if (typeof myValue === "number") {
  // myValue: number
  myValue.toFixed();
}
if (typeof myValue === "string") {
  // myValue: string
  myValue.toUpperCase();
}
```

## 3️⃣ 논리 연산자를 통한 내로잉
`if`, `&&`, `||`, 삼항 연산자 등을 이용해서 `falsy`한 값을 제외한 내로잉이 가능합니다.<br />

```ts
declare let myValue: string | undefined;

if (myValue) {
  // myValue: string
  myValue.toUpperCase();
}

// myValue: string
myValue && myValue.toUpperCase();

// myValue: string
!myValue || myValue.toUpperCase();

// myValue: string
myValue ? myValue.toUpperCase() : null;
```

단 주의해야 할 점은 `""`과 `0` 같은 `falsy`한 값이 있기 때문에 타입이 원하는 대로 추론되지 않을 수 있습니다.<br />

```ts
declare let myValue: string | undefined;

if (myValue) {
  // myValue: string
  myValue.toUpperCase();
} else {
  // myValue: string | undefined
  myValue; // 빈 문자열("")도 "falsy"한 값이기 때문에 "undefined"로 확신할 수 없음
}
```

# 💀 리터럴 타입 ( literal type )
원시 타입중 하나가 아닌 특정 원싯값으로 알려진 타입을 의미합니다.<br />
즉, `string`이 아닌 `"Aatrox"`, `10` 등의 구체적인 타입을 리터럴 타입이라고 합니다.<br />
( `string`, `number` 같은 원시 타입은 해당 타입의 모든 리터럴 타입의 집합체입니다. )<br />

```ts
// myValue: "Aatrox"
const myValue = "Aatrox";
```


## 0️⃣ 리터럴 타입 할당 가능성
동일한 원시 타입이라고 하더라도 서로 다른 리터럴 타입이라면 할당이 불가능합니다.<br />

```ts
let myValue: "Aatrox" = "Aatrox";
myValue = "Puppy"; // Error: Type '"Puppy"' is not assignable to type '"Aatrox"'
```

또한 넓은 타입에서 좁은 타입으로 할당이 가능(`(1)`)하지만, 반대는 불가능(`(2)`)합니다.<br />

```ts
// (1)
let myValue1: string;
myValue1 = "Aatrox"; // "Aatrox" => string

// (2)
let myValue2: "Aatrox";
// Error: Type 'string' is not assignable to type '"Aatrox"'.
myValue2 = myValue1; // string => "Aatrox"
```

# ⛑️ 엄격한 null 검사
모든 타입에 `null`과 `undefined`를 허용할지 여부를 결정하는 것입니다.<br />

## 0️⃣ strictNullChecks
`strictNullChecks` 옵션을 이용해서 설정할 수 있습니다.<br />
( 엄격하게 `null`을 검사하는 것이 더 모범적이라고 합니다. )<br />

`strictNullChecks: false`로 설정하면 모든 타입에 `| null | undefined`가 추가된 것처럼 동작합니다.<br />

+ `tsconfig.json`

```json
{
  "compilerOption": {
    "strictNullChecks": true, /* When type checking, take into account null and undefined */
    // ... 생략
  }
}
```

아래 예시는 `null` 체크 여부에 `null`을 넣었을 때의 결과입니다.<br />

```ts
let myValue: string;

// strictNullChecks: true
myValue = null; // Error: Type 'null' is not assignable to type 'string'.
myValue = undefined; // Error: Type 'undefined' is not assignable to type 'string'.

// strictNullChecks: false
myValue = null; // 문제 없음
myValue = undefined; // 문제 없음
```

## 1️⃣ 초깃값이 없는 변수
초깃값을 넣어주지 않은 변수라면 타입이 정해져 있더라도 타입 체커가 인지하고 타입 에러를 띄워줍니다.<br />

```ts
let myValue1: string;

// Error: Variable 'myValue1' is used before being assigned.
myValue1.length;
// Error: Variable 'myValue1' is used before being assigned.
myValue1?.length;

let myValue2: string | undefined;

// Error: Variable 'myValue2' is used before being assigned.
myValue2.length;
// 정상 동작
myValue2?.length;
```

# ⛳ 타입 별칭
`type` 키워드를 이용해서 타입에 대한 변수인 타입 별칭을 생성할 수 있습니다.<br />

```ts
type MyValue = string | number;

let myValue: MyValue = "Aatrox";
```

## 0️⃣ 타입 별칭은 TypeScript만의 문법
타입 별칭은 `JavaScript`에 존재하지 않습니다.<br />
즉, 컴파일하고 나면 사라집니다. ( 런타임에 존재하지 않음 )<br />

바로 위의 코드를 컴파일하면 아래와 같은 결과가 나옵니다.<br />
( `tsconfig.json`의 옵션에 따라 조금씩 달라질 수 있지만, 핵심은 `type`이 없어진다는 것입니다. )<br />

```ts
"use strict";
let myValue = "Aatrox";
```

타입 별칭은 유니온 / 인터섹션 등으로 결합될 수 있습니다.<br />
여기서 중요한 점은 `MyValueWithNumber`가 `MyValue`를 참조하기 때문에 `MyValue`가 변하면 `MyValueWithNumber`도 동기화 된다는 점입니다.<br />

```ts
type MyValue = string | undefined;

// 즉, "type MyValueWithNumber = number | string | undefined"
type MyValueWithNumber = MyValue | number;
```

# ❓ 의문
## 0️⃣ 교재 72p ( strictNullCheck )
**"`strictNullCheck`를 비활성화면 코드의 모든 타입에 `| null | undefined`를 추가해야 모든 변수가 `null` 또는 `undefined`를 할당할 수 있습니다."**라고 적혀있는데 "비활성화"가 아니라 "활성화하면"이라고 적어야 맞지 않나라고 생각해서 기록합니다.<br />

활성화해야 `null` 체크를 엄격하게 하기 때문에 `null`을 따로 추가해줘야 변수에 할당할 수 있는 것이 맞지 않나요...?<br />

## 1️⃣ 타입 내로잉
~~[타입 내로잉](https://tsplay.dev/wXO6kW){:target="_blank"}에서는 왜 `falsy`한 값을 제대로 추론하지 않을까요...?~~<br />

`string`에는 `falsy`한 값인 `""`가 들어있기 때문에 `else`에서 `"" | undefined`라서 `string | undefined`로 추론하는 것 같네요 🥲<br />

```ts
let myValue = Math.random() > 0.5 ? "Aatrox" : undefined;

if (myValue) {
  // myValue: string
  myValue.toUpperCase();
} else {
  // myValue: string | undefined
  myValue; // Q: 왜 "" | undefined로 추론하지 않을까??
}

// 아래 코드는 제대로 추론함
if (typeof myValue === "string") {
  // myValue: string
  myValue.toUpperCase();
} else {
  // myValue: undefined
  myValue;
}
```

# 📮 레퍼런스
1. « 러닝 타입스크립트 3장 » ( 조시 골드버그 지음, 고승원 옮김, 한빛미디어, 2023 )