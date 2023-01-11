---
layout: post
title: 타입스크립트 유틸리티 기능 만들어보기
author: admin
date: 2023-01-11 18:46:00 +900
lastmod: 2023-01-11 18:46:00 +900
sitemap:
  changefreq: monthly
  priority: 0.5
categories: [TypeScript, UtilityTypes]
tags: [Pick, Partial, ReturnType, Record]
---

> 해당 포스트는 `TypeScript`의 `UtilityTypes`을 직접 만들어서 기록하는 게시글입니다.<br />
{: .prompt-info}

`이펙티브 타입스크립트`를 읽으면서 유틸리티 타입이 나오는 경우 답을 보지 않고 먼저 만들고 비교해보는 방식으로 작성합니다.<br />
아마도 이 책에서 언급을 한다면 여태까지 알려줬던 개념들을 활용하면 이해할 수 있다고 판단해서 알려준다고 생각해서 이해를 했다면 구현도 할 수 있을거니까 직접 구현해보는 방식으로 작성하고 있습니다.<br />

## 📌 Pick
> 이펙티브 타입스크립트 79p

특정 객체 타입과 키들을 받아서 해당 키들만 있는 객체 타입으로 리턴하는 유틸리티<br />

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

## 📌 Partial
> 이펙티브 타입스크립트 80p

특정 객체 타입을 받아서 모두 옵셔널로 바꾸는 유틸리티<br />

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

## 📌 ReturnType
> 이펙티브 타입스크립트 82p

FIXME: 아직 감도 안잡혀요...

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

## 📌 Record
> 이펙티브 타입스크립트 88p

특정 키들과 값의 타입을 받아서 객체의 타입을 리턴하는 유틸리티<br />

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