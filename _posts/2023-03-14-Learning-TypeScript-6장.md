---
layout: post
title: 러닝 타입스크립트 6장
author: admin
date: 2023-03-14 08:29:00 +900
lastmod: 2023-03-14 08:29:00 +900
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

> 해당 포스트는 `러닝 타입스크립트` 6장을 읽고 정리한 포스트입니다.<br />책의 모든 내용을 작성하는 것이 아닌 주관적인 기준에 따라 필요한 정보만 정리했습니다.<br />
{: .prompt-info}

# 📇 배열 타입
배열은 초깃값을 기반으로 타입을 추론합니다.<br />

```ts
const arr1 = [1, 2]; // number[]
const arr2 = [1, "2"]; // (number | string)[]
```

## 0️⃣ 배열과 함수 타입
타입 애너테이션을 이용해서 초깃값 없이 변수의 타입을 배열로 지정할 수 있습니다.<br />

```ts
let arr1: string[];

arr1 = ["1"];
arr1 = [1]; // Error: Type 'number' is not assignable to type 'string'
```

## 1️⃣ 유니언 타입의 배열
배열 타입을 사용할 때는 괄호로 정확하게 구분해야하는 경우가 있습니다.<br />

```ts
// string 배열을 반환하는 함수
const func1: () => string[] = () => ["1", "2"];

// string을 반환하는 함수의 배열
const arr1: (() => string)[] = [() => "1", () => "2"];

// string 이거나 number 배열
const v1: string | number[] = "str";

// "string | number"의 배열
const v2: (string | number)[] = [1, "str"];
```

## 2️⃣ any 배열의 진화
초깃값을 부여하지 않은 배열은 `any[]`가 되고 새로운 값이 추가되면 그 값에 해당하는 타입도 추가됩니다.<br />

```ts
let arr = []; // any[]

arr.push(""); // string[]

arr[0] = 0; // (string | number)[]
```

## 3️⃣ 다차원 배열
차원의 수 만큼 `[]`를 붙이면 다차원 배열이 됩니다.<br />

```ts
const arr: string[][] = [
  [""],
  ["", ""],
  ["", ""],
];
```

# 🃏 배열 멤버
배열으로 얻은 값은 해당 배열의 타입과 같습니다.<br />

```ts
const arr = [0, ""];
const v = arr[0]; // v: string | number
```

## 0️⃣ 불안정한 멤버
`TypeScript`에서는 검색된 배열의 멤버가 존재하는지에 대해서는 확인하지 않습니다.<br />

```ts
const func = (arr: string[]) => {
  arr[9999].length; // 타입 에러를 발생하지 않음
};

func(["1", "2"]);
```

# 🗑️ 스프레드와 나머지 매개변수
## 0️⃣ 스프레드
스프레드 연산자를 이용해서 배열을 생성하면 가능한 모든 타입의 배열로 추론됩니다.<br />

```ts
const arr1 = [0]; // arr1: number[]
const arr2 = [""]; // arr2: string[]
const arr3 = [...arr1, ...arr2]; // arr3: (number | string)[]
```

## 1️⃣ 나머지 매개변수 스프레드
나머지 매개변수도 동일하게 타입을 검사합니다.<br />

```ts
const func = (v: number, ...args: string[]) => {};

func(1);
func(1, "a", "b");
func(1, ...["a", "b"]);
func(1, ...["a", 1]); // Error: Argument of type 'string | number' is not assignable to parameter of type 'string'
```

# 🗃️ 튜플
고정된 크기의 배열을 튜플이라고 합니다.<br />
일반 배열보다 더 구체적인 타입입니다.<br />

```ts
const tuple1: [string, number] = ["", 0];
const tuple2: [string, number] = ["", 0, ""]; // Error: Type '[string, number, string]' is not assignable to type '[string, number]'.
const tuple3: [string, number] = ["", ""]; // Error: Type 'string' is not assignable to type 'number'.
```

## 0️⃣ 튜플 할당 가능성
튜플은 일반 배열보다 더 구체적으로 추론됩니다.<br />
따라서 일반 배열을 튜플에 할당할 수 없습니다.<br />

```ts
const arr1 = ["", ""]; // arr1: string[]
let tuple1: [string, string] = ["", ""]; // tuple1: [string, string]

// Error: Type 'string[]' is not assignable to type '[string, string]'.
tuple1 = arr1;
tuple1 = [...arr1];

tuple1 = [arr1[0], arr1[1]]; // 정상 동작
```

## 1️⃣ 나머지 매개변수로서의 튜플
함수의 매개변수는 순서가 존재하기 때문에 튜플을 사용해서 순서에 맞는 정확한 타입을 스프레드로 사용하지 않으면 타입 오류가 발생합니다.<br />

```ts
const func = (s: string, n: number) => {};

const arr1 = ["", 0]; // arr1: (string | number)[]
const tuple1: [string, number] = ["", 0]; // tuple1: [string, number]

func(...arr1); // A spread argument must either have a tuple type or be passed to a rest parameter.
func(...tuple1);
```

## 2️⃣ 튜플 추론
일반적으로 배열을 반환하면 배열 내부의 모든 값들의 타입의 유니온으로 추론됩니다.<br />
하지만 튜플을 반환하면 정해진 위치에 정해진 타입으로 반환 값을 받을 수 있습니다.<br />

```ts
const getArr = () => ["", 0]; // getArr: () => (string | number)[]
const getTuple: () => [string, number] = () => ["", 0]; // getTuple: () => [string, number]

const [s1, n1] = getArr();
const [s2, n2] = getTuple();

/**
 * s1: string | number
 * n1: string | number
 * s2: string
 * n2: number
 */
```

## 3️⃣ const 어서션
`as const`를 사용해서 `readonly`인 튜플을 쉽게 만들 수 있습니다.<br />
함수의 반환 값에 사용하면 원하던 결과를 쉽게 얻을 수 있습니다.<br />

```ts
const arr1 = ["", 0] as const; // arr1: readonly ["", 0]
arr1[0] = "1"; // Cannot assign to '0' because it is a read-only property.
arr1[1] = 1; // Cannot assign to '1' because it is a read-only property.

const func = () => ["", 0] as const;
const [s, n] = func();
/**
 * s: ""
 * n: 0
 */
```

# 📮 레퍼런스
1. « 러닝 타입스크립트 6장 » ( 조시 골드버그 지음, 고승원 옮김, 한빛미디어, 2023 )