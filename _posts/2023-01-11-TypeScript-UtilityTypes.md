---
layout: post
title: 타입스크립트 유틸리티 기능 만들어보기
author: admin
date: 2023-01-11 18:46:00 +900
lastmod: 2023-01-31 11:44:00 +900
sitemap:
  changefreq: monthly
  priority: 0.5
categories: [FrontEnd, TypeScript]
tags: [Pick, Partial, ReturnType, Record, Readonly, Exclude, Omit]
---

> 해당 포스트는 `TypeScript`의 `UtilityTypes`을 직접 만들어서 기록하는 게시글입니다.<br />
{: .prompt-info}

`이펙티브 타입스크립트`를 읽으면서 유틸리티 타입이 나오는 경우 답을 보지 않고 먼저 만들고 비교해보는 방식으로 작성합니다.<br />
아마도 이 책에서 언급을 한다면 여태까지 알려줬던 개념들을 활용하면 이해할 수 있다고 판단해서 알려준다고 생각해서 이해를 했다면 구현도 할 수 있을거니까 직접 구현해보는 방식으로 작성하고 있습니다.<br />

## 📌 `Pick<T, K>`
> 이펙티브 타입스크립트 79p
{: .prompt-tip }

특정 객체 타입과 키들을 받아서 해당 키들만 있는 객체 타입으로 리턴하는 유틸리티 타입입니다.<br />

```ts
/**
 * + 송신
 * 원본 타입
 * 원본 타입의 key들
 *
 * + 수신
 * 원본 타입에서 해당하는 key들의 집합체
 */
type MyPick<T, K extends keyof T> = {
  [key in K]: T[key];
};

type Person = {
  name: string;
  age: number;
  gender: boolean;
};

type MyPerson = MyPick<Person, "name" | "age">;

// ===== 정답 =====
type Pick<T, K extends keyof T> = {
  [P in K]: T[P];
};
```

## 📌 `Partial<T>`
> 이펙티브 타입스크립트 80p
{: .prompt-tip }

특정 타입을 받아서 모두 옵셔널로 바꾸는 유틸리티 타입입니다.<br />

```ts
/**
 * + 송신
 * 타입
 *
 * + 수신
 * 모두 옵셔널로 변경된 타입
 */
type MyPartial<T> = {
  [key in keyof T]?: T[key];
};

type Person = {
  name: string;
  age: number;
  gender: boolean;
};

type MyPerson = MyPartial<Person>;

// ===== 정답 =====
type Partial<T> = {
  [P in keyof T]?: T[P];
};
```

## 📌 `ReturnType` TODO:
> 이펙티브 타입스크립트 82p
{: .prompt-tip }

```ts
/**
 * + 송신
 * 함수 형태의 타입
 *
 * + 수신
 * 함수의 리턴 값의 타입
 */
type MyReturnType<T> = {};

type Func = (x: number, y: number) => string;

const func: Func = (x, y) => x + y + "";
```

## 📌 `Record<T, K>`
> 이펙티브 타입스크립트 88p
{: .prompt-tip }

특정 키들과 값의 타입을 받아서 객체의 타입을 리턴하는 유틸리티 타입입니다.<br /><br />

```ts
/**
 * + 송신
 * 1. 키들
 * 2. 키들이 가질 타입
 *
 * + 수신
 * 키와 타입을 갖는 객체
 */
type MyRecord<T extends string | number | symbol, K> = {
  [key in T]: K;
};

type Pos = MyRecord<"x" | "y" | "z", number>;

const pos: Pos = { x: 1, y: 1, z: 1 };

// ===== 정답 =====
type Record<K extends keyof any, T> = {
  [P in K]: T;
};
```

## 📌 `Readonly<T>`
> 이펙티브 타입스크립트 100p
{: .prompt-tip }

특정 객체를 받아서 `readonly`를 적용한채로 반환하는 유틸리티 타입입니다.<br />

```ts
/**
 * + 송신
 * 1. 특정 객체
 *
 * + 수신
 * readonly가 적용된 객체
 */
type MyReadonly<T> = {
  readonly [key in keyof T]: T[key];
};

type Person = {
  name: string;
  age: number;
  vision: {
    left: number;
    right: number;
  };
};

const person: MyReadonly<Person> = {
  name: "alice",
  age: 26,
  vision: {
    left: 1.0,
    right: 1.2,
  },
};

// 불가능
person.name = "blue";

// 가능
person.vision.left = 1.5;

// ===== 정답 =====
type Readonly<T> = {
  readonly [P in keyof T]: T[P];
};
```

## 📌 `Exclude<T, U>`
> 아래 코드가 이해가지 않는다면 [조건부 타입](/posts/조건부-타입/){:target="_blank"}를 읽어보시는 것을 추천드립니다!<br />( 교재에서 언급하진 않았지만 `Omit<T, K>`를 구현하기 위해 작성했습니다. )<br />
{: .prompt-tip }

`T`타입에서 `U`타입에 포함되는 것을 제외하는 유틸리티 타입입니다.<br />

```ts
type MyExclude<T, U> = T extends U ? never : T;

type MyType1 = Exclude<"a" | "b" | "c" | "d", "a" | "b">; // "c" | "d"
type MyType2 = MyExclude<"a" | "b" | "c" | "d", "a" | "b">; // "c" | "d"

// ===== 정답 =====
type Exclude<T, U> = T extends U ? never : T;
```

## 📌 `Omit<T, K>`
> 이펙티브 타입스크립트 165p
{: .prompt-tip }

`T`타입에서 `K`라는 `key`를 갖는 `property`를 제외하는 유틸리티 타입입니다.<br />
( 대부분 답 보고 풀었습니다... 😢 )<br />

조금 궁금한게 굳이 `K extends keyof any`를 할 필요가 있는지 궁금하긴 합니다.<br />
추측으로는 어떤 타입이든 상관없이 받으려고 그런 것 같은데 이렇게 만들면 자동완성을 도와주지 않아서 불편하네요..<br />

```ts
type MyPick<T, K extends keyof T> = { [key in K]: T[key] };
type MyExclude<T, U> = T extends U ? never : T;

type MyOmit<T, K extends keyof T> = MyPick<T, MyExclude<keyof T, K>>;

type Champion = {
  name: string;
  speed: number;
  line: "TOP" | "JUG" | "MID" | "AD" | "SUP";
};

type MyType1 = Omit<Champion, "name">;
type MyType2 = MyOmit<Champion, "name">;
/**
 * {
 *   speed: number;
 *   line: "TOP" | "JUG" | "MID" | "AD" | "SUP";
 * }
 */

// ===== 정답 =====
type Omit<T, K extends keyof any> = Pick<T, Exclude<keyof T, K>>;
```

## 📮 레퍼런스
1. [1-blue - 조건부 타입](/posts/조건부-타입/){:target="_blank"}