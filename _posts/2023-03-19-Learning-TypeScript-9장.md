---
layout: post
title: 러닝 타입스크립트 9장 ( 타입 제한자 )
author: admin
date: 2023-03-19 00:35:00 +900
lastmod: 2023-03-19 00:35:00 +900
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

> 해당 포스트는 `러닝 타입스크립트` 9장을 읽고 정리한 포스트입니다.<br />책의 모든 내용을 작성하는 것이 아닌 주관적인 기준에 따라 필요한 정보만 정리했습니다.<br />
{: .prompt-info}

# 🗼 top 타입
top 타입은 시스템에서 가능한 모든 값을 나타내는 타입입니다.<br />

`unknown`은 top 타입이고 `any`도 top 타입이라고 볼 수 있습니다.<br />

## 0️⃣ any 다시보기
`any` 모든 타입에서 할당될 수 있는 타입이라서 top 타입처럼 작동할 수 있습니다.<br />
( `unknown`을 제외한 모든 타입의 top 타입 )<br />

`any`는 할당 가능성 또는 멤버에 대해 타입 검사를 수행하지 않습니다.<br />
즉, 모든 타입에 할당이 가능하면서, `TypeScript`의 도움을 전혀 받을 수 없습니다.<br />
`any`를 잘못 사용하면 런타임에 오류가 발생하는 경우를 체크할 수 없게 됩니다.<br />

```ts
const v: any = 1;

// 런타임에 오류가 발생하는 코드
v.length;
```

## 1️⃣ unknown
`unknown`은 모든 타입의 top 타입입니다.<br />
`any`처럼 모든 타입을 할당할 수 있지만, 사용할 때 제한이 걸립니다.<br />

1. `unknown` 타입의 값의 속성에 직접적으로 접근할 수 없음
1. top 타입이 아닌 타입에서 할당할 수 없음 ( 할당되는 거는 어떤 타입이라도 상관없음 )

즉, `unknown`은 어떤 타입이라도 할당될 수 있지만, 마음대로 사용할 수는 없는 타입입니다.<br />
`any`와는 다르게 사용에 제한이 걸리기 때문에 훨씬 더 안전합니다.<br />

```ts
interface FuncHandler {
  (v: unknown): void
}

const func: FuncHandler = (v) => {
  // Error: 'v' is of type 'unknown'.
  v.length;
};

// 정상 동작
func("string");
```

`unknown`을 사용하기 위해서는 타입 좁히기(`(1)`)나 타입 어서션(`(2)`)을 사용해야 합니다.<br />

```ts
class Person {
  name = "Aatrox";
}

interface FuncHandler {
  (v: unknown): void;
}

const func: FuncHandler = (v) => {
  // (1)
  if (typeof v === "string") v.length;
  
  // (1)
  if (v instanceof Person) v.name;

  // // (2) 동작은 하지만 위험하기 때문에 위처럼 타입 가드를 사용하는 것이 좋음
  // (v as string).length;
};

// 정상 동작
func("string"); // 6
func(new Person()); // "Aatrox"
```

# 🛡️ 타입 서술어 ( 사용자 정의 타입 가드 )
`instanceof`, `typeof`를 이용해서 충분히 타입을 좁힐 수 있습니다.<br />
하지만 함수에서 타입을 체크하고 밖으로 나오게 되면 좁혀진 타입을 다시 원상복구됩니다.<br />

```ts
const isNumber: IsNumberHandler = (value: unknown) => {
  return typeof value === "number";
};

const v: unknown = 26;

if (isNumber(v)) {
  // Error: 'v' is of type 'unknown'
  v.toFixed();
}
```

함수를 이용한 타입 좁히기를 사용하는 방법이 타입 서술어(사용자 정의 타입 가드)입니다.<br />
`매개변수 is 타입` 형식으로 사용하고 `true`를 반환하면 해당 타입으로 추론됩니다.<br />

```ts
const isNumber = (value: unknown): value is number => {
  return typeof value === "number";
};

const v: unknown = 26;

if (isNumber(v)) {
  // v: number;
  v.toFixed();
}
```

기본 타입만이 아니라 임의로 만들어진 타입에도 사용할 수 있습니다.<br />

```ts
class Pet {
  name = "애완";
}
class Fish extends Pet {
  swim() {
    console.log("헤엄");
  }
}
class Bird extends Pet {
  fly() {
    console.log("날아");
  }
}

const isFish = (pet: Pet): pet is Fish => pet instanceof Fish;

declare const pet: Pet;

if(isFish(pet)) {
  // pet: Fish
  pet;
} else {
  // pet: Pet
  pet;
}
```

타입 서술어가 유용하긴 하지만 정해진 목적이외의 다른 의도로 사용하는 것은 예상하지 못한 동작을 초래할 수 있습니다.<br />
`(1)`에서 `str`이 `string`이지만 `TypeScript`에서는 `never`로 추론하게 됩니다.<br />

```ts
const isLongString = (value: unknown): value is string => {
  return typeof value === "string" && value.length > 7;
};

let str = "apple";

if (isLongString(str)) {
  // str: string
  str;
} else {
  // (1) str: never
  str;
}
```

# 🗿 타입 연산자
객체의 `key` 혹은 `value`의 타입을 추출하는 연산자입니다.<br />

## 0️⃣ keyof
제공되는 **"타입"**의 `key` 값들의 유니언인 타입을 만들어줍니다.<br />
`(1)`과 같은 경우에 사용하면 휴먼 에러도 없이 자동으로 업데이트됩니다.<br />

```ts
interface Person {
  name: string;
  age: number;
}

// Error: Element implicitly has an 'any' type because expression of type 'string' can't be used to index type 'Person'.
// No index signature with a parameter of type 'string' was found on type 'Person'
const func1 = (person: Person, key: string) => person[key];

// (1) 정상 동작
const func2 = (person: Person, key: keyof Person) => person[key];

// MyKey: "name" | "age"
type MyKey = keyof Person;

const k1: MyKey = "age";
const k2: MyKey = "name";
```

`keyof`는 타입에만 사용할 수 있는 연산자입니다.<br />
일반 값에 사용하면 아래와 같이 오류가 발생합니다.<br />

```ts
const person = {
  name: "",
  age: 0,
};

// Error: 'person' refers to a value, but is being used as a type here. Did you mean 'typeof person'?
type P = keyof person;
```

## 1️⃣ typeof
제공되는 **"값"**의 타입을 반환해주는 연산자입니다.<br />

```ts
const person = {
  name: "",
  age: 0,
};

type Person = typeof person;

const v: Person = {
  name: "Aatrox",
  age: 26,
}
```

`JavaScript`에서 타입을 확인할 때 사용하는 `typeof`와는 다르게 동작합니다.<br />
`TypeScript` 전용 문법이기 때문에 컴파일 후에는 사라집니다.<br />
( 사용되는 위치가 타입이라면 `TypeScript`, 값이라면 `JavaScript`로 동작합니다. )<br />

## 2️⃣ keyof typeof
`keyof typeof`를 사용하면 제공되는 값의 타입을 얻고 그 타입의 `key`를 얻어내는 방법으로 사용됩니다.<br />
즉, 특정 값의 `key`들만 추출해낼 수 있는 방법입니다.<br />

```ts
const person = {
  name: "",
  age: 0,
  gender: true,
};

// Key = "name" | "age" | "gender"
type Key = keyof typeof person;
```

제네릭과 활용하면 아래와 같이 사용할 수 있습니다.<br />

```ts
const func = <T extends {}>(v: T, k: keyof typeof v) => v[k];

const person = {
  name: "Aatrox",
  age: 26,
  gender: true,
};

func(person, "name"); // "Aatrox"
func(person, "age"); // 26
func(person, "gender"); // true

// Error: Argument of type '"weight"' is not assignable to parameter of type '"name" | "age" | "gender"'
func(person, "weight");
```

# 🪦 타입 어서션
`TypeScript`는 강력하게 타입화되는 경우에 가장 잘 동작합니다.<br />
하지만 런타임 이전에 타입을 추론할 수 없는 불가피한 경우가 존재합니다.<br />
그런 경우에는 타입 어서션을 사용할 수 밖에 없는 혹은 사용하면 더 유용한 경우가 있습니다.<br />

`JSON.parse()` 같은 경우는 실제로 실행해보기 전에는 반환 타입을 유추할 수 없기 때문에 `any`를 반환합니다.<br />
`fetch()`나 `catch()`의 첫 번째 인자인 `Error` 객체도 마찬가지입니다.<br />
( `fetch()`는 `any`, `catch`는 `unknown` )<br />

## 0️⃣ 포착된 오류 타입 어서션
`JavaScript`에서 `try ~ catch`를 사용하는 경우 `Error` 객체를 이용하는 것이 모범 사례지만, 그렇게 사용하지 않아도 동작합니다.<br />

첫 번째 인자인 `error`는 기본적으로 `unknown`이기 때문에 타입 좁히기를 이용해서 안전하게 사용하는 것이 좋습니다.<br />

```ts
import axios, { AxiosError } from "axios";

try {
  // // 대충 axios 에러 혹은 Error를 발생시키는 코드
  // throw new Error("내가 만든 에러... 객체에 담았지...");
} catch (error) {
  // error: unknown

  if (error instanceof AxiosError) {
    // error: AxiosError<any, any>
    console.error("AxiosError >> ", error);
  }
  if (error instanceof Error) {
    // error: Error
    console.error("Error >> ", error);
  }
}
```

## 1️⃣ non-null 어서션
`!`를 사용하고 `null`과 `undefined` 타입을 제외합니다.<br />

```ts
const value = Math.random() > 0.5 ? "" : null;

// Error: 'value' is possibly 'null'.
value.length;

// 정상 동작
value!.length;
```

## 2️⃣ 타입 어서션 주의사항
타입 어서션은 타입 체크의 동작을 어느정도 혹은 아예 억제하기 때문에 제대로 타입 시스템이 동작하지 않을 수 있습니다.<br />
따라서 확실하게 안전할 때만 사용해야 합니다.<br />

`(1)`과 같은 경우는 확실하게 값이 있다고 생각해서 아래와 같이 사용했지만, 물론 실제로도 값이 존재하지만 이후에 코드가 어떻게 변할지 모르고 `Map`을 수정하고 `(1)`을 바꾸지 않으면 런타임에 에러가 발생해 추적하기 힘들게 됩니다.<br />

```ts
const map = new Map([["x1", 1]]);

// (1)
const v = map.get("x1")!;

// v: number;
v;
```

### 1. 어서션 vs 선언
어서션을 이용해서 타입을 강제하는 것과 타입 애너테이션을 이용해서 타입을 강제하는 것에는 차이가 있습니다.<br />

```ts
interface Person {
  name: string;
  age: number;
}

// Error: Property 'age' is missing in type '{ name: string; }' but required in type 'Person'
const p1: Person = {
  name: "",
};

// 에러 없음
const p2 = {
  name: "",
} as Person;

// 런타임에 에러 발생
p2.age;
```

### 2. 어서션 할당 가능성
`as`를 통한 어서션은 항상 사용 가능한 것은 아닙니다.<br />
타입 중 하나가 다른 타입에 할당 가능한 경우에만 두 타입 간의 타입 어서션을 허용합니다.<br />

```ts
// Conversion of type 'string' to type 'number' may be a mistake because neither type sufficiently overlaps with the other.
// If this was intentional, convert the expression to 'unknown' first.
const v = "" as number;
```

서로 완전히 다른 타입인 경우 이중으로 어서션을 사용해야 합니다.<br />
하지만 이중 타입 어셔션은 위험하고 코드의 타입이 잘못되었다는 징후이기 때문에 조심해야합니다.<br />

```ts
// v: number
const v = "" as unknown as number;
```

# ⛑️ const 어서션
`as const`를 사용하면 사용되는 타입에 따라서 조금씩 다르게 동작합니다.<br />

1. 배열인 경우 읽기 전용 튜플로 전환
1. 리터럴인 경우 원시 타입이 아닌 구체적인 리터럴로 전환
1. 객체의 경우 속성이 읽기 전용으로 전환

## 0️⃣ 리터럴에서 원시 타입으로
특정 필드가 더 구체적인 리터럴 값을 갖도록 하고 싶을 때 사용하면 됩니다.<br />

```ts
// getName1: () => string
const getName1 = () => "Aatrox";

// getName1: () => "Aatrox"
const getName2 = () => "Aatrox" as const;

/**
 * person: {
 *   name: string;
 *   age: 26;
 * }
 */
const person = {
  name: "Aatrox",
  age: 26 as const,
};
```

## 1️⃣ 읽기 전용 객체
객체에 `const` 어서션을 사용하면 모든 속성이 `readonly`가 되고 값은 구체적인 타입으로 변합니다.<br />

배열은 `readonly` 튜플이 되고, 원시 타입은 리터럴이 되고, 객체 또한 `readonly`가 됩니다.<br />
재귀적으로 적용되어 객체 내부의 모든 속성에 적용이 됩니다.<br />

```ts
/**
 * person: {
 *   readonly name: "Aatrox";
 *   readonly arr: readonly [1, 2, 3];
 *   readonly obj: {
 *     readonly age: 26;
 *   };
 * }
 */
const person = {
  name: "Aatrox",
  arr: [1, 2, 3],
  obj: {
    age: 26,
  },
} as const;
```

# 📮 레퍼런스
1. « 러닝 타입스크립트 9장 » ( 조시 골드버그 지음, 고승원 옮김, 한빛미디어, 2023 )