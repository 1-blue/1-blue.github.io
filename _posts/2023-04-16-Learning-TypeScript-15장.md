---
layout: post
title: 러닝 타입스크립트 15장 ( 타입 운영 )
author: admin
date: 2023-04-16 21:36:00 +900
lastmod: 2023-04-20 10:26:00 +900
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

> 해당 포스트는 `러닝 타입스크립트` 15장을 읽고 정리한 포스트입니다.<br />책의 모든 내용을 작성하는 것이 아닌 주관적인 기준에 따라 필요한 정보만 정리했습니다.<br />
{: .prompt-info}

# 🧩 매핑된 타입
매핑된 타입은 키 집합의 각 키에 대한 새로운 속성을 만들어 새로운 타입을 생성하는 것을 의미합니다.<br />
인덱스 시그니처(`[i: string]`)와 유사하지만 `:`대신 `in`을 사용합니다.<br />

`[key in Animals]`에서 `key`는 변수처럼 아무 값이나 들어갈 수 있고, `Animals`의 `key`들을 하나씩 뽑아서 등록합니다.<br />

```ts
type Animals = "cat" | "dog";

type Animal = {
  [key in Animals]: number;
};

/**
 * type Animal = {
 *   cat: number;
 *   dog: number;
 * }
 */
```

## 0️⃣ 타입에서 매핑된 타입
[`keyof`](/posts/Learning-TypeScript-9장/#0%EF%B8%8F⃣-keyof){:target="_blank"}를 사용해서 특정 타입의 `key`를 뽑아와서 매핑된 타입에 사용할 수 있습니다.<br />

```ts
interface Animals {
  dog: string;
  cat: number;
}

// 기존 타입의 "key"를 추출해서 boolean으로 변경
type AnimalCopy = {
  [key in keyof Animals]: boolean;
};

/**
 * type AnimalCount = {
 *   dog: boolean;
 *   cat: boolean;
 * }
 */
```

매핑된 타입을 사용하면 메서드를 모두 속성 구문으로 바꿀 수 있습니다.<br />
( 클래스에 구현하지 않는 이상 두 구문의 차이는 없는 것으로 알고 있습니다. )<br />

+ 메서드 구문: `func(): void` ( 객체들끼리 공유하는 메서드 )
+ 속성 구문: `func: () => void` ( 각 객체가 독립적으로 갖는 메서드 )

```ts
interface Funcs {
  func1(): void;
  func2: () => void;
}

type JustProperties<T> = {
  [key in keyof T]: T[key];
};

type Func = JustProperties<Funcs>;

/**
 * type Func = {
 *   func1: () => void;
 *   func2: () => void;
 * }
 */
```

위처럼 동작하는 이유는 메서드 속성을 꺼내서 사용하면 아래와 같이 속성 구문으로 변환되기 때문입니다.<br />

```ts
interface Funcs {
  func1(): void;
  func2: () => void;
}

// F1 = () => void
type F1 = Funcs["func1"];

// F2 = () => void
type F2 = Funcs["func2"];
```

## 1️⃣ 제한자 변경 & 제네릭 매핑된 타입
기존 타입에 `readonly`, `?`, `-readonly`, `-?`를 붙이면 해당하는 제한자가 적용됩니다.<br />

```ts
interface MyType {
  name: string;
  age?: number;
  readonly gender: boolean;
}

// "Readonly<T>"와 같음
type MakeReadonly<T> = {
  readonly [key in keyof T]: T[key];
};
// "Partial<T>"와 같음
type MakeOptional<T> = {
  [key in keyof T]?: T[key];
};
type ExcludeReadonly<T> = {
  -readonly [key in keyof T]: T[key];
};
// "Required<T>"와 같음
type ExcludeOptional<T> = {
  [key in keyof T]-?: T[key];
};

type MyReadOnlyType = MakeReadonly<MyType>;
/**
 * type MyReadOnlyType = {
 *   readonly name: string;
 *   readonly age?: number | undefined;
 *   readonly gender: boolean;
 * } 
 */

type MyOptionalType = MakeOptional<MyReadOnlyType>;
/**
 * type MyOptionalType = {
 *   readonly name?: string | undefined;
 *   readonly age?: number | undefined;
 *   readonly gender?: boolean | undefined;
 * }
 */

type MyExcludeReadonlyType = ExcludeReadonly<MyOptionalType>;
/**
 * type MyExcludeReadonlyType = {
 *   name?: string | undefined;
 *   age?: number | undefined;
 *   gender?: boolean | undefined;
 * }
 */

type MyExcludeOptionalType = ExcludeOptional<MyExcludeReadonlyType>;
/**
 * type MyExcludeOptionalType = {
 *   name: string;
 *   age: number;
 *   gender: boolean;
 * }
 */
```

# 🧸 조건부 타입
조건부 타입은 특정 타입을 기준으로 조건에 의해서 두 가지 타입 중 하나의 타입으로 정해지는 것을 의미합니다.<br />

`JavaScript`의 삼항 연산자와 유사하지만 `extends`를 사용하는 차이가 있습니다.<br />
그 이외에는 유사한 구조로 동작합니다.<br />

```ts
const s = "s";
const n = 0;

// MyType1 = true
type MyType1 = (typeof s) extends string ? true : false;
// MyType2 = false
type MyType2 = (typeof n) extends string ? true : false;
```

## 0️⃣ 제네릭 조건부 타입
제네릭과 조건부 타입을 같이 사용하면 더 유연하게 사용할 수 있습니다.<br />

```ts
const s = "s";
const n = 0;

type CheckString<T> = T extends number ? true : false;

// MyType1 = false
type MyType1 = CheckString<typeof s>;
// MyType2 = true
type MyType2 = CheckString<typeof n>;
```

아래는 조금은 더 유용하게 사용하는 예시입니다.<br />

```ts
const f = () => 0;
const s = "s";

type MakeCallable<T> = T extends () => any ? T : () => T;

// MyType1 = () => number
type MyType1 = MakeCallable<typeof f>;
// MyType2 = () => "s"
type MyType2 = MakeCallable<typeof s>;
```

아래는 특정 타입을 제외하는 `Utility Types`의 `Exclude<T, U>`와 `Omit<T, U>`에 대한 정의입니다.<br />
제네릭 조건부 타입을 사용하기도 하고 처음 두 개를 사용하면 둘의 사용 목적이 헷갈리기 때문에 내용에 추가했습니다.<br />

```ts
type Exclude<T, U> = T extends U ? never : T;
type Omit<T, K extends keyof any> = Pick<T, Exclude<keyof T, K>>;
```

`Exclude<T, U>`와 `Omit<T, U>`은 아래와 같이 사용할 수 있습니다.<br />

```ts
interface User {
  name: string;
  age: number;
  gender?: boolean;
}
// UserKeys = "name" | "age" | "gender"
type UserKeys = keyof User;

// MyUserKeys = "name" | "gender"
type MyUserKeys = Exclude<UserKeys, "age">;

// "Omit<T, U>"은 내부적으로 "Exclude<T, U>" 사용
type MyUser = Omit<User, "age">;
/**
 * type MyUser = {
 *   name: string;
 *   gender?: boolean | undefined;
 * }
 */
```

## 1️⃣ 타입 분산과 분산 조건부 타입
> [inpa - 조건부 타입과 분산 조건부 타입](https://inpa.tistory.com/entry/TS-%F0%9F%93%98-%ED%83%80%EC%9E%85%EC%8A%A4%ED%81%AC%EB%A6%BD%ED%8A%B8-%EC%A1%B0%EA%B1%B4%EB%B6%80-%ED%83%80%EC%9E%85-%EC%99%84%EB%B2%BD-%EC%9D%B4%ED%95%B4%ED%95%98%EA%B8%B0){:target="_blank"}를 참고하시면 매우 도움이 됩니다. 🙂<br />
{:.prompt-tip}

조건부 타입은 제네릭의 유무에 따라서 동일하게 생긴 형태에 대한 결과가 다르게 나옵니다.<br />
`(1)`과 `(2)`는 제네릭의 차이를 제외하고는 동일한 형태의 조건부 타입입니다.<br />
따라서 같은 결과가 나올거라고 추측하는데 실제로 동작하는 것을 보면 전혀 다른 결과가 나옵니다.<br />

그 이유는 제네릭을 이용한 조건부 타입은 분산 조건부 타입으로 동작합니다.<br />
( 조건부 타입에 `(naked) type parameter`를 사용하면 분산 방식으로 동작한다고 합니다. )<br />
( `(naked) type parameter`란 제네릭과 같은 의미가 없는 타입 파라미터입니다. )<br />

분산 조건부 타입은 각 타입별로 조건부 타입이 적용되고 그 결괏값들의 유니언으로 결정됩니다. (`(3)`)<br />

```ts
type MyType<T> = T extends string ? true : false;

// (1) MyValue1 = false
type MyValue1 = (string | number) extends string ? true : false;

// (2) MyValue2 = boolean
type MyValue2 = MyType<string | number>;
// (3)
// 1. (string extends string ? true : false) | (number extends string ? true : false)
// 2. true | false
// 3. boolean
```

## 2️⃣ 유추된 타입
`infer`와 새로운 타입(`U`)을 선언하면 조건부 타입이 `true`인 경우 새로운 타입을 사용할 수 있습니다.<br />
즉, `infer`는 들어온 타입을 추론해서 새로운 타입 변수로 만들어주는 기능입니다.<br />

이 기능이 대체 어디에 사용되냐를 살펴보면 함수 매개변수/리턴 값의 타입을 찾는데 사용됩니다.<br />

아래 두 코드는 실제 `utility types`인데 모두 `infer`를 이용해서 함수의 타입을 추론합니다.<br />

```ts
type Parameters<T extends (...args: any) => any> = T extends (...args: infer P) => any ? P : never;
type ReturnType<T extends (...args: any) => any> = T extends (...args: any) => infer R ? R : any;
```

실제로 사용을 해보면 아래와 같은 타입을 얻을 수 있습니다.<br />
`(1)`의 `P`는 `...args`의 타입이기 때문에 배열 형태(튜플)로 얻게됩니다.<br />

```ts
interface Func {
  (x: number, y: number): boolean;
}

// (1) P = [x: number, y: number]
type P = Parameters<Func>;

// R = boolean
type R = ReturnType<Func>;
```

## 3️⃣ 매핑된 조건부 타입
제네릭과 매핑된 조건부 타입을 같이 사용하면 각 멤버를 같은 형태의 타입으로 변환시킬 수 있습니다.<br />

```ts
type MakeFunc<T> = {
  [key in keyof T]: T[key] extends (...args: any[]) => any
    ? T[key]
    : () => T[key];
};

type Funcs = MakeFunc<{
  func1: () => string;
  func2: number;
}>;

/**
 * type Funcs = MakeFunc<{
 *   func1: () => string;
 *   func2: number;
 * }>;
 */
```

# 🪀 never
일반적으로 `never` 타입은 아무것도 가질 수 없는 타입입니다.<br />
반환을 하지 않는 즉, 예외를 던지는 함수의 반환 타입으로 사용할 수 있습니다.<br />

하지만 `never`가 유니언(`|`)과 결합되면 **해당 값이 무시되는 기능**을 합니다.<br />
이 기능을 이용해서 특정 값을 제외시키는데 활용할 수 있습니다.<br />

## 0️⃣ never와 교차, 유니언 타입
교차 타입(`&`)은 모두 `never`로 만들고, 유니언 타입(`|`)은 모두 유니언 대상인 타입으로 만듭니다.<br />

```ts
// MyType1 = never
type MyType1 = never & string;

// MyType2 = string
type MyType2 = never | string;
```

## 1️⃣ never과 조건부 타입
> 아래 코드가 명확하게 이해가 안 간다면 [inpa - 조건부 타입과 분산 조건부 타입](https://inpa.tistory.com/entry/TS-%F0%9F%93%98-%ED%83%80%EC%9E%85%EC%8A%A4%ED%81%AC%EB%A6%BD%ED%8A%B8-%EC%A1%B0%EA%B1%B4%EB%B6%80-%ED%83%80%EC%9E%85-%EC%99%84%EB%B2%BD-%EC%9D%B4%ED%95%B4%ED%95%98%EA%B8%B0){:target="_blank"}을 참고해주세요!<br />
{:.prompt-tip}

`never`은 유니언에서 무시되기 때문에 `never`과 조건부 타입을 같이 사용하면 `never`에 해당하는 것은 결과에 포함하지 않습니다.<br />
( `Parameters<F>`에도 조건부 타입에 `never`를 사용합니다. )<br />

```ts
type OnlyStrings<T> = T extends string ? T : never;

// MyType1 = "x"
type MyType1 = "a" | "b" extends string ? "x" : never;
// MyType2 = never
type MyType2 = "a" | "b" | 10 | true extends string ? "x" : never;
// MyType3 = "a" | "b"
type MyType3 = OnlyStrings<"a" | "b" | 10 | true>;
```

## 2️⃣ never와 매핑된 타입
`(1)`처럼 원하는 조건에 맞는 프로퍼티가 아닌 경우 `value`의 타입을 `never`로 만들고 `keyof`로 `key`들의 유니언을 만들면 `never`가 제외되기 때문에 원하는 타입의 키들의 유니언을 얻을 수 있습니다.<br />

```ts
type OnlyStringProperties<T> = {
  [key in keyof T]: T[key] extends string ? key : never;
};

interface Person {
  name: string;
  age: number;
  gender: boolean;
  aa: string;
}

/**
 * (1)
 * type MyType1 = {
 *   name: "name";
 *   age: never;
 *   gender: never;
 *   aa: "aa"
 * }
 */
type MyType1 = OnlyStringProperties<Person>;

// (2) MyType2 = "name" | "aa"
type MyType2 = OnlyStringProperties<Person>[keyof Person];
```

# 🧲 템플릿 리터럴 타입
문자열 타입의 패턴에 의한 타입을 구현할 수 있는 문법입니다.<br />

```ts
type MyString = `Hello, ${string}`;

// Error: Type '"Hello,JavaScript"' is not assignable to type '`Hello, ${string}`'
const str1: MyString = "Hello,JavaScript";

const str2: MyString = "Hello, TypeScript";
```

문자열 리터럴 타입의 유니언을 활용하면 더 구체적으로 타입을 선언할 수 있습니다.<br />

```ts
type Color = "red" | "yellow";
type Fruits = "apple" | "banana";

type MyString = `${Color} | ${Fruits}`;

const str1: MyString = "red | apple";
const str2: MyString = "yellow | banana";
const str3: MyString = "red | banana";
const str4: MyString = "yellow | apple";

// Error: Type '"green | apple"' is not assignable to type '"red | apple" | "red | banana" | "yellow | apple" | "yellow | banana"'. Did you mean '"red | apple"'
const str5: MyString = "green | apple";
```

`string`뿐만이 아니라 `number`, `bigint`, `boolean`, `null`, `undefined`와 같은 원시 타입은 사용가능합니다.<br />

```ts
type MyString = `a ${number} b`;

const str1: MyString = "a 2 b";
const str2: MyString = "a 22.22 b";
```

## 0️⃣ 고유 문자열 조작 타입
아래 네 가지의 문자열 조작 타입이 있습니다.<br />

1. `Uppercase<S>`: 문자열 리터럴을 대문자로 변환
1. `Lowercase<S>`: 문자열 리터럴을 소문자로 변환
1. `Capitalize<S>`: 문자열 리터럴의 첫 번째 글자를 대문자로 변환
1. `Uncapitalize<S>`: 문자열 리터럴의 첫 번째 글자를 소문자로 변환

```ts
// MyType1 = "APPLE"
type MyType1 = Uppercase<"apple">;
// MyType2 = "apple"
type MyType2 = Lowercase<"APPLE">;
// MyType3 = "Apple"
type MyType3 = Capitalize<"apple">;
// MyType4 = "aPPLE"
type MyType4 = Uncapitalize<"APPLE">;
```

## 1️⃣ 템플릿 리터럴 키
템플릿 리터럴은 템플릿 문자열이 들어갈 수 있는 어떤 위치라도 들어갈 수 있습니다.<br />
따라서 다음을 응용해서 매핑된 타입의 인덱스 시그니처로 사용할 수 있습니다.<br />

```ts
type Key = "apple" | "blue" | "color";

type MyType = {
  [key in `my${Capitalize<Key>}`]: string;
};
/**
 * type MyType = {
 *   myApple: string;
 *   myBlue: string;
 *   myColor: string;
 * }
 */
```

## 2️⃣ 매핑된 타입 키 다시 매핑하기
템플릿 리터럴 키를 사용해서 특정 값을 기반으로 새로운 타입을 매핑할 수 있습니다.<br/ >

```ts
const fruits = {
  apple: 300,
  banana: 200,
  melon: 100,
};

type Fruits = {
  [key in keyof typeof fruits as `my${Capitalize<key>}`]: Promise<number>;
};
/**
 * type Fruits = {
 *   myApple: Promise<number>;
 *   myBanana: Promise<number>;
 *   myMelon: Promise<number>;
 * }
 */
```

템플릿 리터럴 키에는 `symbol`이 들어갈 수 없기 때문에 `(1)`에서는 타입 오류가 발생합니다.<br />
객체의 `key`에는 `string` or `symbol`이 들어올 수 있기 때문에 `T`라는 타입에는 `string` or `symbol`이라서 안전하지 않은 타입에 대한 오류가 발생합니다.<br />

따라서 `(2)`처럼 `T & string`을 이용해서 `symbol`이라면 `symbol & string` 즉, `never`가 되기 때문에 타입 문제 없이 안전하게 사용할 수 있게 됩니다.<br />

```ts
// (1) Error: Type 'T' does not satisfy the constraint 'string'.
type MyValue1<T> = {
  [key in keyof T as `get${Capitalize<T>}`]: T[key];
};

// (2)
type MyValue2<T> = {
  [key in keyof T as `get${Capitalize<T & string>}`]: T[key];
};
```

# 📮 레퍼런스
1. « 러닝 타입스크립트 15장 » ( 조시 골드버그 지음, 고승원 옮김, 한빛미디어, 2023 )

1. [inpa - 조건부 타입과 분산 조건부 타입](https://inpa.tistory.com/entry/TS-%F0%9F%93%98-%ED%83%80%EC%9E%85%EC%8A%A4%ED%81%AC%EB%A6%BD%ED%8A%B8-%EC%A1%B0%EA%B1%B4%EB%B6%80-%ED%83%80%EC%9E%85-%EC%99%84%EB%B2%BD-%EC%9D%B4%ED%95%B4%ED%95%98%EA%B8%B0){:target="_blank"}

1. [1-blue - keyof](/posts/Learning-TypeScript-9장/#0%EF%B8%8F⃣-keyof){:target="_blank"}