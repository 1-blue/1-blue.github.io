---
layout: post
title: 러닝 타입스크립트 2장
author: admin
date: 2023-02-27 21:17:00 +900
lastmod: 2023-03-07 07:34:00 +900
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

> 해당 포스트는 `러닝 타입스크립트` 2장을 읽고 정리한 포스트입니다.<br />
{: .prompt-info}

# 🧮 타입
타입이란 `JavaScript`에서 다루는 값의 **형태**에 대한 설명입니다.<br />
( 형태란 값에 존재하는 속성과 메서드 그리고 내장되어 있는 `typeof` 연산자의 결과를 의미 )<br />

`TypeScript`는 값을 아래의 일곱 가지 기본 원시 타입으로 간주합니다.<br />

+ 일곱 가지 기본 원시 타입
  1. `null`
  1. `undefined`
  1. `boolean`
  1. `string`
  1. `number`
  1. `bigint`
  1. `symbol`

> `Boolean`, `Number`, `String`과 같은 래퍼 객체와는 다릅니다.
{:.prompt-warning}

## 0️⃣ 타입 추론
기본적으로 초깃값으로 변수의 타입을 추론합니다.<br />
`(1)`처럼 복잡한 초깃값도 제대로 추론합니다.<br />

```ts
// myValue1: string
let myValue1 = "string";

// myValue2: number
let myValue2 = 2;

// (1) myValue3: string | number
let myValue3 = Math.random() > 0.5 ? "string" : 2;
```

## 1️⃣ 타입 시스템
값의 사용법이 타입과 일치하지 않으면 오류를 발생합니다.<br />

```ts
let myValue1 = "2";
let myValue2 = 2;

myValue1.length;
myValue2.length; // Error: Property 'length' does not exist on type 'number'.
```

## 2️⃣ 오류 종류
`noEmitOnError: false` 옵션을 이용해서 구문/타입 오류를 무시하고 컴파일을 진행할 수 있습니다.<br />
`npx tsc --init`으로 `tsconfig.json`을 생성하면 `noEmitOnError: true`가 기본 값입니다.<br />
( 개인적인 생각으로는 구문 오류든 타입 오류든 다 해결하는 것이 좋다고 생각해서 굳이 바꿀 필요는 없다고 생각합니다. )<br />

```json
{
  "compilerOption": {
    "noEmitOnError": false, /* Disable emitting files if any type checking errors are reported. */
    // ... 생략
  }
}
```

### 1. 구문 오류
`TypeScript`가 코드로 이해할 수 없는 잘못된 구문을 감지할 때 발생합니다.<br />
기본적으로 구문 오류가 발생하는 경우에는 컴파일을 중지합니다.<br />
( 즉, `JavaScript` 파일을 생성하지 않습니다. )<br />

```ts
// "JavaScript"에서도 같은 에러 발생
let let myValue; // Error: ','이(가) 필요합니다.
```

### 2. 타입 오류
`TypeScript`의 추론으로 타입에서 오류를 발견한 경우 발생합니다.<br />
일반적으로 타입 오류가 발생했다고 컴파일을 중지하지는 않습니다.<br />

개발자가 타입 체커에게 거짓말을 하지 않았다면 대부분은 런타임에서 오류가 발생하기 때문에 타입 오류를 해결하고 컴파일하는 것이 좋습니다.<br />

```ts
// myValue1: string
let myValue1 = "string";
// Error: 'toFixed' 속성이 'string' 형식에 없습니다. 'fixed'을(를) 사용하시겠습니까?
myValue1.toFixed();

// myValue2: number
let myValue2 = 2;
// Error: 'number' 형식에 'length' 속성이 없습니다.
myValue2.length;

// myValue3: string | number
let myValue3 = Math.random() > 0.5 ? "string" : 2;
// Error: 'string | number' 형식에 'toFixed' 속성이 없습니다. 'string' 형식에 'toFixed' 속성이 없습니다.
myValue3.toFixed();
// Error: 'string | number' 형식에 'length' 속성이 없습니다. 'number' 형식에 'length' 속성이 없습니다.
myValue3.length;
```

# 🎲 할당 가능성
이전에 초깃값을 기반으로 변수의 타입을 추론하는 것을 확인했습니다.<br />
이전에 추론했던 타입이 재할당에도 그대로 유지되며 사용됩니다.<br />

```ts
// myValue1: string
let myValue1 = "string";
myValue1 = "apple";

// myValue2: number
let myValue2 = 2;
myValue2 = 26;

// myValue3: string | number
let myValue3 = Math.random() > 0.5 ? "string" : 2;
myValue3 = myValue1;
myValue3 = myValue2;
```

단 변수를 `const`로 선언하면 기본적으로 재할당도 불가능하고 더 구체적인 타입으로 추론하기 때문에 `TypeScript` 측면에서도 재할당이 불가능합니다.<br />

```ts
// const myValue1: "string"
const myValue1 = "string";
```

# 🩻 타입 애너테이션
변수에 초깃값을 작성하지 않는 경우가 있을 수 있습니다.<br />
기본적으로는 `any` 타입으로 추론하게 됩니다.<br />

## 0️⃣ 진화하는 any
`any`로 추론된 변수는 이후에 할당되는 값의 타입을 갖습니다.<br />
( 즉 기존 `any` 타입에서 진화됩니다. )<br />

```ts
// let myValue: any
let myValue;

// let myValue: any
myValue = 1;

// let myValue: number
myValue.length; // Error: Property 'length' does not exist on type 'number'.

// let myValue: number
myValue = "1";

// let myValue: string
myValue.toFixed(); // Error: Property 'toFixed' does not exist on type 'string'.
```

초깃값이 정해지지 않은 변수라면 타입 애너테이션을 이용해서 타입을 지정해줄 수 있습니다.<br />
( 런타임 코드에는 전혀 영향을 주지 않고 컴파일 시 제거됩니다. )<br />

```ts
// let myValue1: any
let myValue1;

// let myValue2: string
let myValue2: string;

myValue2 = "apple";
myValue2 = 26; // Error: 'number' 형식은 'string' 형식에 할당할 수 없습니다.
```

만약 타입 애너테이션을 지정하지 않고 초깃값을 정해주지 않으면 `any`가 할당되어 타입 체커의 도움을 받을 수 없으며, 런타임에 문제가 발생할 가능성이 높습니다.<br />

```ts
let myValue: any;

myValue = "str";

// ( 런타임에 발생 ) Error: TypeError: myValue.toFixed is not a function
myValue.toFixed();
```

## 1️⃣ 불필요한 타입 애너테이션
만약 타입 체커에 의해 자동으로 추론되는 타입이라면 타입 애너테이션을 사용하지 않는 것이 좋습니다.<br />
( 물론 상황에 따라 다르지만 일반적인 경우에 대한 이야기입니다. )<br />

```ts
let myValue1: string = "apple";
let myValue2: number = 26;
```

## 2️⃣ noImplicitAny
`TypeScript`에서 `any`로 추론하는 것을 막고 싶다면 `noImplicitAny` 설정을 사용하면 됩니다.<br />

```json
{
  "compilerOption": {
    "noImplicitAny": true, /* Enable error reporting for expressions and declarations with an implied 'any' type. */
    // ... 생략
  }
}
```

위와 같은 설정을 하면 `(2)`에서 매개변수의 타입을 정의하지 않았다는 타입 오류를 발생시킵니다.<br />
( 근데 왜 `(1)`에서는 오류를 발생시키지 않을까요...? 🥲 )<br />

```ts
// (1) 근데 이건 왜 에러가 안 나죠...?
let v;

v = 10;
v = "a";

// (2) Error: Parameter 'x' implicitly has an 'any' type.
const func = (x) => {};
``` 

# 🔭 타입 형태
타입이 정해졌다면 해당 타입이 가능한 행위를 쉽게 사용할 수 있도록 자동완성해주고, 잘못된 부분이 있는지 체크해줍니다.<br />

```ts
// 타입 애너테이션을 이용해서 구체적인 타입을 정해줘도 되지만 비효율적이고 가독성도 안 좋음
const champion1: {
  firstName: string;
  age: number;
} = {
  firstName: "Aatrox",
  age: 26,
};

const champion2 = {
  firstName: "Aatrox",
  age: 26,
};

// Error: 'firstname' 속성이 '{ firstName: string; age: number; }' 형식에 없습니다. 'firstName'을(를) 사용하시겠습니까?
champion2.firstname;
```

# 🍻 모듈과 스크립트
`TypeScript`에서는 [`ES6`의 모듈 시스템인 `ESM`](https://www.typescriptlang.org/ko/docs/handbook/modules.html){:target="_blank"}을 사용합니다.<br />

+ 모듈: `export | import`가 있는 파일
+ 스크립트: 모듈이 아닌 모든 파일 ( `export | import`가 없는 파일 )

## 0️⃣ 모듈
모듈에서 선언 변수는 다른 파일과 충돌되지 않습니다.<br />
즉, 각 파일에 종속적이고 `export`로 내보내지 않으면 다른 파일에서 사용할 수 없습니다.<br />

아래 예시의 `myValue`는 각 파일의 독립적인 모듈로 존재하는 값입니다.<br />
따라서 서로의 존재를 모르기 때문에 이름이 같아도 충돌나지 않습니다.<br />

```ts
// a.ts
export const myValue = "Aatrox";

// b.ts
export const myValue = "Aatrox";

// index.ts
import { myValue } from "./a";
console.log(myValue);
```

## 1️⃣ 스크립트
스크립트에서는 동일한 변수가 다른 파일에 선언되어 있어도 충돌이 발생합니다.<br />

아래 예시의 `myValue`는 각 파일이 달라도 해당 파일을 전역 스코프로 간주하기 때문에 같은 이름에 대한 충돌이 발생합니다.<br />
( 솔직히 스크립트로 사용할 필요가 없다고 생각해서 즉, `import`, `export` 없이 사용할 이유가 없다고 생각해서 아래와 같은 충돌에 대해서는 신경쓰지 않아도 된다고 개인적으로 생각합니다.  )<br />

```ts
// a.ts
const myValue = "Aatrox"; // Error: 블록 범위 변수 'myValue'을(를) 다시 선언할 수 없습니다.

// b.ts
const myValue = "Aatrox"; // Error: 블록 범위 변수 'myValue'을(를) 다시 선언할 수 없습니다.

// index.ts
console.log(myValue); // undefined
```

충돌을 해결하는 방법은 스크립트를 모듈로 만들어주면 됩니다.<br />
즉, 단순하게 원하는 것을 내보내거나 `export {}`처럼 아무것도 내보내지 않고 `export`를 사용하면 됩니다.<br />

```ts
// a.ts
const myValue = "Aatrox";
export {};

// b.ts
const myValue = "Aatrox";
export {};

// index.ts
console.log(myValue); // "undefined"
```

# 📮 레퍼런스
1. « 러닝 타입스크립트 2장 » ( 조시 골드버그 지음, 고승원 옮김, 한빛미디어, 2023 )
1. [TypeScript - Module](https://www.typescriptlang.org/ko/docs/handbook/modules.html){:target="_blank"}
1. [inpa - TS 모듈 & 네임 스페이스 시스템 이해하기](https://inpa.tistory.com/entry/TS-%F0%9F%93%98-%ED%83%80%EC%9E%85%EC%8A%A4%ED%81%AC%EB%A6%BD%ED%8A%B8-%EB%AA%A8%EB%93%88-%EB%84%A4%EC%9E%84%EC%8A%A4%ED%8E%98%EC%9D%B4%EC%8A%A4-%EC%8B%9C%EC%8A%A4%ED%85%9C-%EC%9D%B4%ED%95%B4%ED%95%98%EA%B8%B0){:target="_blank"}