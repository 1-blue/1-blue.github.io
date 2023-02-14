---
layout: post
title: 타입스크립트 함수 오버로딩 ( Overloading )
author: admin
date: 2023-02-14 16:19:00 +900
lastmod: 2023-02-14 16:19:00 +900
sitemap:
  changefreq: monthly
  priority: 0.5
categories: [FrontEnd, TypeScript]
tags: [TypeScript, Overloading]
---

> [typescript exercises 6번](https://typescript-exercises.github.io/#exercise=6&file=/index.ts){:target="_blank"}을 풀면서 몰랐던 기능인 함수 오버로딩에 대한 정리 포스트입니다.<br />( 함수 오버로딩을 몰랐기 때문에 당연히 문제도 못 풀었습니다... 😥 )
{: .prompt-info}

# 📚 함수 오버로딩
동일한 이름이지만 전달 받는 매개변수/반환 값이 다른 함수를 여러 개 선언하는 방법을 의미합니다.<br />

## 0️⃣ 함수 오버로딩을 사용하는 경우
1. 함수의 매개변수의 타입에 따라 다른 응답을 하고 싶은 경우
2. 함수의 매개변수의 개수에 따라 다른 응답을 하고 싶은 경우

## 1️⃣ 함수 오버로딩에 대해
일단 함수 오버로딩을 위해 작성한 시그니처들은 외부에서 볼 수 없습니다.<br />
즉, 컴파일 후에 사라지는 `TypeScript`에만 존재하는 구문입니다.<br />

또한 **함수 오버로딩 시그니처들과 함수 구현체는 모두 호환**되어야 합니다.<br />
하나라도 호환되지 않으면 타입 에러가 발생합니다.<br />

코드 예시는 아래에서 자세하게 살펴보겠습니다.<br />

# 📝 함수 오버로딩 예시
## 0️⃣ 좋은 오버로딩 예시
아래 예시는 정상적으로 오버로딩을 구현한 예시입니다.<br />
함수 오버로딩 시그니처에서 정의한 매개변수의 타입을 모두 받고 / 제대로 반환합니다.<br />

```ts
function func(x: string): string;
function func(x: number): number;
function func(x: boolean): boolean;

function func(x: string | number) {
  return x;
}
```

## 1️⃣ 잘못된 오버로딩 예시
이전에 함수 오버로딩에 대해 설명할 때 **함수 오버로딩 시그니처들과 함수 구현체는 모두 호환되어야 한다**고 했습니다.<br />
아래 두 가지 예시는 시그니처와 구현체가 호환되지 않는 두 가지 예시입니다.<br />

먼저 구현체에 매개변수의 타입을 제대로 작성하지 않은 경우입니다.<br />

```ts
function func(x: string): string;
function func(x: number): number;
function func(x: boolean): boolean; // Error: 이 오버로드 시그니처는 해당 구현 시그니처와 호환되지 않습니다.

function func(x: string | number) {
  return x;
}
```

다음은 구현체에 반환 타입을 제대로 작성하지 않은 경우입니다.<br />

```ts
function func(x: string): string;
function func(x: number): number; // Error: 이 오버로드 시그니처는 해당 구현 시그니처와 호환되지 않습니다.
function func(x: boolean): boolean;

function func(x: string | number): string {
  /**
   * Error:
   * 'string | number | boolean' 형식은 'string' 형식에 할당할 수 없습니다.
   * 'number' 형식은 'string' 형식에 할당할 수 없습니다.
   */
  return x;
}
```

## 2️⃣ 조금 더 구체적인 예시
> [typescript exercises 6번](https://typescript-exercises.github.io/#exercise=6&file=/index.ts){:target="_blank"}의 일부분을 수정한 예시입니다.<br />
{: .prompt-info}

아래 타입을 모두 가지고 있다고 가정하고 예시 코드를 작성하겠습니다.<br />

```ts
interface User {
  type: "user";
  name: string;
  age: number;
  occupation: string;
}

interface Admin {
  type: "admin";
  name: string;
  age: number;
  role: string;
}

type Person = User | Admin;

const me_user: User = {
  type: "user",
  name: "박상은",
  age: 26,
  occupation: "student",
};
const me_admin: Admin = {
  type: "admin",
  name: "1-blue",
  age: 26,
  role: "developer",
};
```

`Person`타입을 받아서 이름을 `"무명"`으로 바꾸고 새로운 객체를 반환하는 함수입니다.<br />

`(1)`에서 보면 반환 타입이 `Person`으로 변합니다.<br />
일반적으로 생각하면 `User`를 넣으면 당연히 `User`가 나오고, `Person`을 넣으면 당연히 `Person`이 나와야 하는데 타입 체커는 매개변수와 반환 값의 타입을 보고 반환 타입을 결정합니다.<br />
따라서 저희는 반환 타입을 결정할 방법을 생각해야 하는데 일단 생각나는 방법이 사용자 정의 타입 가드입니다.<br />
근데 사용자 정의 타입 가드를 사용하려면 `boolean`을 반환해야 하는데 반환 타입이 정해져 있으니 제외해야합니다.<br />
그러면 다음으로 생각난 방법이 함수 오버로딩입니다.<br />
( 사실은 생각은 못 했고 설명의 흐름을 위한 내용입니다... 🥲 )<br />

```ts
// 리턴 타입을 작성하지 않아도 결국 "Person"을 반환한다고 추론합니다.
function changeName(person: Person): Person {
  return { ...person, name: "무명" };
}

// (1) "me1", "me2"의 타입은 모두 "Person"입니다.
const me1 = changeName(me_user);
const me2 = changeName(me_admin);
```

`(2)`처럼 함수 오버로딩을 하면 매개변수의 타입에 맞게 반환 타입도 정해집니다.<br />

```ts
// (2)
function getMe(person: User): User;
function getMe(person: Admin): Admin;
function getMe(person: Person): Person {
  return { ...person, name: "무명" };
}

const me1 = getMe(me_user); // User
const me2 = getMe(me_admin); // Admin
```

하지만 제네릭으로도 같은 결과를 얻을 수 있습니다.<br />
공부하고 예시를 작성하면서 함수 오버로딩으로만 해결이 가능한 경우를 작성하고 싶었는데 아직 실력과 경험이 부족해서 그런 예시를 작성하지는 못했습니다.<br />
FIXME: 나중에 더 공부하고 깨달음을 얻으면 그때 예시를 추가하도록 하겠습니다.<br />

```ts
function getMe<T>(person: T): T {
  return { ...person, name: "무명" };
}

const me1 = getMe(me_user); // User
const me2 = getMe(me_admin); // Admin
```

## 3️⃣ 매개변수 타입과 반환 타입이 다른 오버로딩
매개변수의 타입과 개수가 다르고 그에 따라서 반환하는 타입도 다른 예시입니다.<br />

만약 함수 오버로딩 시그니처들(`(3)`)이 존재하지 않는다면 `(4)`에서 에러가 발생하지 않습니다.<br />

반환 타입이 정해져 있지 않기 때문에 `number`를 반환해도 상관 없지만 함수 시그니처가 존재하면 단일 인수에는 `string`을 받아 `string`을 반환하는 것이 고정이기 때문에 타입 에러가 발생하는 것입니다.<br />

```ts

// (3)
function func(x: string): string;
function func(x: number, y: number): number;
function func(x: number | string, y?: number) {
  if (typeof x === "number" && typeof y === "number") {
    return x + y;
  }

  return x;
}

func(1, 2);
func("Aatrox");

// (4)
func(1); // Error: 'number' 형식의 인수는 'string' 형식의 매개 변수에 할당될 수 없습니다.
```

# 📮 레퍼런스
1. [typescript exercises](https://typescript-exercises.github.io/){:target="_blank"}
2. [TypeScript 공식 문서 - 함수 오버로드](https://www.typescriptlang.org/ko/docs/handbook/2/functions.html#%ED%95%A8%EC%88%98-%EC%98%A4%EB%B2%84%EB%A1%9C%EB%93%9C){:target="_blank"}