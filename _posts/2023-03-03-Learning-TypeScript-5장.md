---
layout: post
title: 러닝 타입스크립트 5장
author: admin
date: 2023-03-03 21:18:00 +900
lastmod: 2023-03-15 21:44:00 +900
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

> 해당 포스트는 `러닝 타입스크립트` 5장을 읽고 정리한 포스트입니다.<br />책의 모든 내용을 작성하는 것이 아닌 주관적인 기준에 따라 필요한 정보만 정리했습니다.<br />
{: .prompt-info}

# 📥 함수 매개변수
함수의 매개변수도 타입을 지정할 수 있습니다.<br />

```ts
const func = (v: string) => {
  console.log("v >> ", v);
};
```

## 0️⃣ 필수 매개변수
`TypeScript`에서는 매개 변수의 타입과 개수를 정확하게 일치하는지 확인합니다.<br />

```ts
const func = (v: string) => {
  console.log("v >> ", v);
};

func(); // Error: Expected 1 arguments, but got 0.
func("a", "b"); // Error: Expected 1 arguments, but got 2.
func(1); // Error: Argument of type 'number' is not assignable to parameter of type 'string'.
```

## 1️⃣ 선택적 매개변수
이전에 봤던 [타입 애너테이션](/posts/Learning-TypeScript-4장/#3%EF%B8%8F⃣-선택적-속성){:target="_blank"}처럼 선택적으로 매개변수를 받을 수 있습니다.<br />

선택적 매개변수는 유니온 타입으로 `undefined`를 타입으로 갖습니다.<br />

```ts
const func = (v: string, n?: number) => {
  console.log("v >> ", v);

  if (n) console.log("n >> ", n);
};

func("a");
func("a", 2);
func("a", undefined);
func("a", null); // Error: Argument of type 'null' is not assignable to parameter of type 'number | undefined'
```

하지만 `| undefined`를 사용하는 것과는 동일하게 동작하지는 않습니다.<br />

```ts
const func = (v: string, n: number | undefined) => {
  console.log("v >> ", v);

  if (n) console.log("n >> ", n);
};

func("a"); // Error: Expected 2 arguments, but got 1.
func("a", 2);
func("a", undefined);
func("a", null); // Error: Argument of type 'null' is not assignable to parameter of type 'number | undefined'
```

선택적 매개변수는 항상 파라미터의 마지막에 작성해야합니다.<br />

```ts
// Error: 'n' is declared but its value is never read.
const func = (v?: string, n: number) => {};
```

단, 객체의 경우에는 순서가 없기 때문에 옵셔널의 순서도 상관 없습니다.<br />
함수의 매개변수와 헷갈릴 수 있는 부분이라 추가적으로 작성했습니다.<br />

```ts
type Person = {
  name?: string;
  age: number;
};

const func = (person: Person) => {};
```

## 2️⃣ 기본 매개변수
기본 매개변수를 사용하면 굳이 타입 명시하지 않아도 타입을 추론해주고 옵셔널로 바꿔줍니다.<br />

```ts
// func: (v: string, n?: number) => void
const func = (v: string, n = 10) => {
  console.log("v >> ", v);
  console.log("n >> ", n);
};

func("a");
func("a", 2);
func("a", undefined);
func("a", null); // Error: Argument of type 'null' is not assignable to parameter of type 'number | undefined'.
```

## 3️⃣ 나머지 매개변수
나머지 매개변수를 사용하는 경우에도 매개변수의 타입을 지정해줘야 합니다.<br />

```ts
const sum = (...args: number[]) => args.reduce((acc, curr) => acc + curr);
```

# 📤 반환 타입
함수가 반환할 수 있는 모든 값을 이해하면 함수가 반환하는 타입을 알 수 있습니다.<br />
만약 여러 개의 값을 반환하면 그 값들의 타입의 조합으로 반환 값이 유추됩니다.<br />

```ts
// const func: (s: string, n: number) => string | number
const func = (s: string, n: number) => (Math.random() > 0.5 ? s : n);
```

## 0️⃣ 명시적 반환 타입
일반적으로는 함수의 반환 타입을 명시하는 것보다는 알아서 추론하게 하는 것이 좋습니다.<br />

+ 함수의 반환 타입을 명시하면 좋은 경우
  1. 정해진 반환 타입이 확실해서 실수 방지 및 명시하고 싶은 경우
  1. 재귀 함수의 경우 ( 추론하지 못함 )
  1. 큰 프로젝트의 경우 ( 많은 타입을 추론하는데 오래 걸림 )

아래의 함수의 경우에 `number`를 반환 타입을 명시적으로 작성해줬기 때문에 반환 타입의 예외가 발생한 부분을 타입 체커가 알려줍니다.<br />

```ts
// Error: Function lacks ending return statement and return type does not include 'undefined'.
const func = (month: string): number => {
  if (month === "j") return 0;
  if (month === "f") return 1;
  if (month === "m") return 2;
};
```

# 📦 함수 타입
함수 타입은 화살표 함수와 유사하게 사용합니다.<br />

```ts
const func: (n:number) => number = (n) => n * 2;
```

주로 아래와 같이 함수를 인자로 받는 함수에 사용(`(1)`)됩니다.<br />

```ts
const outer1 = (d: number, inner: (n: number) => void) => {};

// (1)
type InnerFunc = (n: number) => void;
const outer2 = (d: number, inner: InnerFunc) => {}
```

## 0️⃣ 함수 타입 괄호
함수 타입을 사용할 때는 괄호를 사용해서 명시적으로 구분하는 것이 좋습니다.<br />

```ts
// const func1: () => string | number ( 함수의 반환 값이 "string | number" )
declare const func1: () => string | number;

// const func2: number | (() => string) ( "string을 반환하는 함수"거나 "number" )
declare const func2: (() => string) | number;
```

## 1️⃣ 매개변수 타입 추론
굳이 인라인으로 함수의 매개변수에 각각 타입을 작성하지 않아도 함수의 타입에 명시된 매개변수의 타입을 보고 추론해줍니다.<br />

```ts
type Func = (n: number) => void;
const outer2:Func = (n) => {
  // n의 타입은 number
}
```

## 2️⃣ 함수 타입 별칭
개인적인 생각으로는 함수 타입을 리터럴로 사용하면 코드의 가독성이 안 좋습니다.<br />
일반적으로 함수 타입의 별칭을 사용하는 것이 좋다고 생각합니다.<br />

아래처럼 함수 타입 별칭을 사용하면 가독성도 더 좋고 코드가 깔끔하게 정리됩니다.<br />
또한 변수처럼 재사용도 가능합니다.<br />

```ts
type Inner = (n: number) => void;
type Outer = (d: number, inner: Inner) => void;

const outer: Outer = (d, inner) => inner(d);
```

# 🫧 그 외 반환 타입
## 0️⃣ void
> `void`는 `JavaScript`에서도 존재하지만 여기서 사용하는 `void`는 `TypeScript`에서 사용하는 키워드입니다.<br />따라서 컴파일하고 나면 사라집니다.
{:.prompt-tip}

`void`는 어떤 값도 반환하지 않는다는 의미를 갖는 타입입니다.<br />
`early return`이나 `undefined`를 리턴하는 경우는 허용합니다.<br />

```ts
type Func = () => void;

const func1: Func = () => undefined;
const func2: Func = () => null;
const func3: Func = () => void; // Error
```

하지만 함수의 반환 값은 무시하고 사용할 수 없습니다.<br />

```ts
type Func = () => void;

const func1: Func = () => undefined;

// const v: void
const v = func1();
```

## 1️⃣ never
존재할 수 없는 값을 의미하는 타입입니다.<br />
함수의 반환 타입에 사용하면 반환을 하면 안되는 함수임을 의미합니다.<br />

항상 오류를 발생하거나, 무한 루프의 함수의 반환 타입으로 사용됩니다.<br />

```ts
type Func = () => never;

// Error: Type 'undefined' is not assignable to type 'never'
const func1: Func = () => undefined;

// Error: Type 'void' is not assignable to type 'never'.
const func2: Func = () => {};

const func3: Func = () => {
  throw new Error("My Error");
};

try {
  func3();
} catch (error) {
  console.error(error);
}
```

# ⛈️ 함수 오버로드
> 함수 오버로드는 함부로 남용하기보다는 설명하기 어려운 함수 타입에 사용하는 최후의 수단으로 사용해야 합니다.<br />( 뒤에서 배울 다른 방법들로 대부분 대체할 수 있습니다. )<br />
{:.prompt-tip}

매개 변수의 타입과 반환 타입이 다르거나, 호출 개수가 다른 함수 시그니처 타입을 여러 개 선언해서 모호한 상황에 적당한 타입을 선택하도록 하는 기술입니다.<br />

**오버로드 시그니처들을 여러 개 선언**하고, 최종으로 **하나의 구현 시그니처를 선언**해야합니다.<br />
그리고 하나의 구현 시그니처는 모든 오버로드 시그니처에 호환돼야 합니다.<br />

만약 아래와 같이 작성하면 에러가 발생하게 됩니다.<br />
왜냐하면 `(2)`의 오버로드 시그니처에 최종 구현 시그니처가 호환되지 않기 때문입니다.<br />
( `(2)`의 `n: number`가 없기 때문에 `(3)`과 호환되지 않습니다. )<br />

`(1)`과 `(2)`는 오버로드 시그니처, `(3)`은 구현 시그니처입니다.<br />

```ts
// (1)
function func(v: string, n: number): string;
// (2) Error: This overload signature is not compatible with its implementation signature.
function func(v: number): string;
// (3)
function func(v: string | number, n: number): string {
  return v + (n + "");
}
```

따라서 아래와 같이 사용해야 문제 없이 동작합니다.<br />

```ts
function func(v: string, n: number): string;
function func(v: number): string;
function func(v: string | number, n?: number): string {
  return v + (n + "");
}
```

## 0️⃣ 함수 오버로드 사용 예시
일반 함수, 함수 오버로드, 제네릭의 사용에 대한 간단한 차이에 대한 예시입니다.<br />

너무 간단한 예시라 실제로 이런 차이를 위해서 사용하는지는 모르겠지만, 함수 오버로딩을 쓰면 아래와 같은 차이가 있다는 것을 정리하기 위해서 작성했습니다.<br />

```ts
// 일반 함수
function funcS(v: string | number | boolean) {
  return v;
}
const vs1 = funcS(""); // vs1: string | number | boolean
const vs2 = funcS(1); // vs2: string | number | boolean
const vs3 = funcS(true); // vs3: string | number | boolean

// 함수 오버로드
function funcO(v: string): string;
function funcO(v: number): number;
function funcO(v: boolean): boolean;
function funcO(v: string | number | boolean) {
  return v;
}
const vo1 = funcO(""); // vo1: string
const vo2 = funcO(1); // vo2: number
const vo3 = funcO(true); // vo3: boolean

// 제네릭
function funcG<T>(v: T) {
  return v;
}
const vg1 = funcG(""); // vg1: ""
const vg2 = funcG(1); // vg2: 1
const vg3 = funcG(true); // vg3: true
```

아래는 조금 더 실질적인 예시입니다.<br />

어떻게 보면 제네릭을 사용하는 게 더 편하고 합리적인 것 같습니다.<br />
그래서 아마 교재에서도 함수 오버로드는 최후의 수단으로 사용하라고 했던 것 같습니다.<br />

그래도 위의 예시를 보면 잘 사용한다면 제네릭으로 해결할 수 없는 함수 오버로딩만의 강점이 있을 것 같습니다.<br />
아직 그 부분까지는 어떤 예시를 만들어야 할지 모르겠어서 이후에 알게 된다면 추가로 작성하겠습니다.<br />

```ts
interface User {
  type: "user";
  name: string;
  age: number;
  role: string;
}
interface Student {
  type: "admin";
  name: string;
  age: number;
  school: string;
}
type Person = User | Student;

declare const user: User;
declare const student: Student;

// 일반 함수
function funcS(person: Person) {
  return person;
}
const vs1 = funcS(user); // vs1: Person
const vs2 = funcS(student); // vs2: Person

// 함수 오버로드
function funcO(person: User): User;
function funcO(person: Student): Student;
function funcO(person: User | Student) {
  return person;
}
const vo1 = funcO(user); // vo1: User
const vo2 = funcO(student); // vo2: Student

// 제네릭
function funcG<T>(person: T) {
  return person;
}
const vg1 = funcG(user); // vg1: User
const vg2 = funcG(student); // vg2: Student
```

# ❓ 의문
## 0️⃣ 반환 타입으로 사용한 void
> [typescript playground](https://tsplay.dev/WYLVdm){:target="_blank"}에서 확인할 수 있습니다.<br />
{:.prompt-info}

반환 타입으로 `void`를 사용했는데 일반적인 값의 반환을 허용하는 경우가 있어서 정확하게 어떤 차이가 있고 어떤 것 때문에 허용하는지 의문을 해결하지 못해서 기록합니다.<br />

```ts
type Func = () => void;

const func1: Func = () => undefined;
const func2: Func = () => null;
const func3: Func = () => 1;
const func4: Func = () => "1";

const func11: () => void = () => undefined;
const func22: () => void = () => null;
const func33: () => void = () => 1;
const func44: () => void = () => "1";

const func111 = (): void => undefined;
const func222 = (): void => null; // Error: Type 'null' is not assignable to type 'void'.
const func333 = (): void => 1; // Error: Type 'number' is not assignable to type 'void'.
const func444 = (): void => "1"; // Error: Type 'string' is not assignable to type 'void'.

function func1111(): void {
  return undefined;
}
function func2222(): void {
  return null; // Error: Type 'null' is not assignable to type 'void'.
}
function func3333(): void {
  return 1; // Error: Type 'number' is not assignable to type 'void'.
}
function func4444(): void {
  return "1"; // Error: Type 'string' is not assignable to type 'void'.
}

// 아래 함수 모두 반환 값이 void
const result1 = func1();
const result2 = func2();
const result3 = func3();
const result4 = func4();
```

# 📮 레퍼런스
1. « 러닝 타입스크립트 5장 » ( 조시 골드버그 지음, 고승원 옮김, 한빛미디어, 2023 )

2. [1-blue - 타입 애너테이션](/posts/Learning-TypeScript-4장/#3%EF%B8%8F⃣-선택적-속성){:target="_blank"}
3. [1-blue - 함수 오버로드](/posts/TypeScript-Function-Overloading/){:target="_blank"}