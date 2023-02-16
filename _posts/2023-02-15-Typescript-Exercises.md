---
layout: post
title: TypeScript Exercises
author: admin
date: 2023-02-15 19:23:00 +900
lastmod: 2023-02-16 09:49:00 +900
sitemap:
  changefreq: monthly
  priority: 0.5
categories: [FrontEnd, TypeScript]
tags: [TypeScript Exercises, Union, Type Guard, 사용자 정의 타입 가드, Omit, Partial, Intersection, Generic]
---

> [`typescript exercises`](https://typescript-exercises.github.io/){:target="_blank"}의 문제 풀이를 작성한 포스트입니다.<br />
{: .prompt-info}

# 📝 Intro
해당 포스트는 [프리온보딩 프론트엔드 챌린지 2월](https://www.wanted.co.kr/events/pre_challenge_fe_6){:target="_blank"}을 수강하면서 마지막 과제로 내주신 [`typescript exercises`](https://typescript-exercises.github.io/){:target="_blank"}의 문제 풀이를 작성한 포스트입니다.<br />
과제는 9번까지 풀어보는 것이지만 나중에 추가로 더 풀어서 추가 업로드할 예정입니다.<br />

## 0️⃣ 문제 요약
1. 사용된 값을 보고 타입을 정의하는 문제
2. [`union`(`|`)](/posts/이펙티브-타입스크립트-2장/#1%EF%B8%8F⃣-union-vs-intersection){:target="_blank"}의 사용법에 대한 문제
3. [타입 좁히기 / 타입 가드](/posts/이펙티브-타입스크립트-3장/#-item-22--타입-좁히기-){:target="_blank"}에 대한 문제
4. [사용자 정의 타입 가드](/posts/이펙티브-타입스크립트-3장/#3%EF%B8%8F⃣-사용자-정의-타입-가드){:target="_blank"}에 대한 문제
5. `TypeScript Utility Types`의 사용법에 대한 문제 ( [`Omit<T, K>`](/posts/TypeScript-UtilityTypes/#-omitt-k){:target="_blank"}, [`Partial<T>`](/posts/TypeScript-UtilityTypes/#-partialt){:target="_blank"} )
6. `TypeScript`의 [함수 오버로딩](/posts/TypeScript-Function-Overloading/){:target="_blank"}에 대한 문제
7. 기본적인 제네릭 사용에 대한 문제
8. [`intersection`(`&`)](/posts/이펙티브-타입스크립트-3장/#-item-22--타입-좁히기-){:target="_blank"}의 사용법에 문제
9. 제네릭을 활용하는 문제

## 1️⃣ 못 푼 문제 ( 답지 확인 )
1. 6번: `TypeScript`의 함수 오버로딩에 대한 개념이 아예 없어서 문제에 대한 접근 조차도 힘들었음

# ✍️ 문제와 풀이
## 1️⃣ 1번 문제 ⭕ ( 타입 정의 )
> [1번 문제](https://typescript-exercises.github.io/#exercise=1&file=%2Findex.ts){:target="_blank"}
{:.prompt-info}

`unknown`으로 정의된 `User`의 타입을 완성시키는 문제입니다.<br />
( 데이터를 보고 타입 정의를 할 수 있는지에 대한 문제 )<br />

### 1. 문제
```ts
export type User = unknown;

export const users: unknown[] = [
  {
    name: "Max Mustermann",
    age: 25,
    occupation: "Chimney sweep",
  },
  {
    name: "Kate Müller",
    age: 23,
    occupation: "Astronaut",
  },
];

export function logPerson(user: unknown) {
  console.log(` - ${user.name}, ${user.age}`);
}

console.log("Users:");
users.forEach(logPerson);
```

### 2. 풀이
```ts
// TODO: 변경
export type User = {
  name: string;
  age: number;
  occupation: string;
};

// TODO: 변경
export const users: User[] = [
  {
    name: "Max Mustermann",
    age: 25,
    occupation: "Chimney sweep",
  },
  {
    name: "Kate Müller",
    age: 23,
    occupation: "Astronaut",
  },
];

// TODO: 변경
export function logPerson(user: User) {
  console.log(` - ${user.name}, ${user.age}`);
}
```

## 2️⃣ 2번 문제 ⭕ ( Union )
> [2번 문제](https://typescript-exercises.github.io/#exercise=2&file=%2Findex.ts){:target="_blank"}
{:.prompt-info}

유니온 타입(`|`)의 사용법과 목적을 알고 있는지에 대한 문제입니다.<br />
[유니온은 타입](/posts/이펙티브-타입스크립트-2장/#1%EF%B8%8F⃣-union-vs-intersection){:target="_blank"}의 합집합인 타입으로 결과를 내는 연산자입니다.<br />

### 1. 문제
```ts
interface User {
  name: string;
  age: number;
  occupation: string;
}

interface Admin {
  name: string;
  age: number;
  role: string;
}

export type Person = unknown;

export const persons: User[] /* <- Person[] */ = [
  {
    name: "Max Mustermann",
    age: 25,
    occupation: "Chimney sweep",
  },
  {
    name: "Jane Doe",
    age: 32,
    role: "Administrator",
  },
  {
    name: "Kate Müller",
    age: 23,
    occupation: "Astronaut",
  },
  {
    name: "Bruce Willis",
    age: 64,
    role: "World saver",
  },
];

export function logPerson(user: User) {
  console.log(` - ${user.name}, ${user.age}`);
}

persons.forEach(logPerson);
```

### 2. 풀이
`Person`이 `User`거나 `Admin`이면 되기 때문에 유니온을 이용해서 `Person`을 정의하면 됩니다.<br />

```ts
interface User {
  name: string;
  age: number;
  occupation: string;
}

interface Admin {
  name: string;
  age: number;
  role: string;
}

// TODO: 변경
export type Person = User | Admin;

// TODO: 변경
export const persons: Person[] /* <- Person[] */ = [
  {
    name: "Max Mustermann",
    age: 25,
    occupation: "Chimney sweep",
  },
  {
    name: "Jane Doe",
    age: 32,
    role: "Administrator",
  },
  {
    name: "Kate Müller",
    age: 23,
    occupation: "Astronaut",
  },
  {
    name: "Bruce Willis",
    age: 64,
    role: "World saver",
  },
];

// TODO: 변경
export function logPerson(user: Person) {
  console.log(` - ${user.name}, ${user.age}`);
}

persons.forEach(logPerson);
```

## 3️⃣ 3번 문제 ⭕ ( 타입 좁히기 / 타입 가드 )
> [3번 문제](https://typescript-exercises.github.io/#exercise=3&file=%2Findex.ts){:target="_blank"}
{:.prompt-info}

[타입 좁히기 / 타입 가드](/posts/이펙티브-타입스크립트-3장/#-item-22--타입-좁히기-){:target="_blank"}에 대한 개념을 가졌는지 대한 문제입니다.<br />

### 1. 문제
```ts
interface User {
  name: string;
  age: number;
  occupation: string;
}

interface Admin {
  name: string;
  age: number;
  role: string;
}

export type Person = User | Admin;

export const persons: Person[] = [
  {
    name: "Max Mustermann",
    age: 25,
    occupation: "Chimney sweep",
  },
  {
    name: "Jane Doe",
    age: 32,
    role: "Administrator",
  },
  {
    name: "Kate Müller",
    age: 23,
    occupation: "Astronaut",
  },
  {
    name: "Bruce Willis",
    age: 64,
    role: "World saver",
  },
];

export function logPerson(person: Person) {
  let additionalInformation: string;
  if (person.role) {
    additionalInformation = person.role;
  } else {
    additionalInformation = person.occupation;
  }
  console.log(` - ${person.name}, ${person.age}, ${additionalInformation}`);
}

persons.forEach(logPerson);
```

### 2. 풀이
`TypeScript`에서는 `in`, `instanceof` 등을 이용해서 타입을 좁힐 수 있습니다.<br />
`union`을 통해서 넓힌 타입은 타입 좁히기를 이용해서 다시 타입을 좁힐 수 있습니다.<br />

```ts
interface User {
  name: string;
  age: number;
  occupation: string;
}

interface Admin {
  name: string;
  age: number;
  role: string;
}

export type Person = User | Admin;

export const persons: Person[] = [
  {
    name: "Max Mustermann",
    age: 25,
    occupation: "Chimney sweep",
  },
  {
    name: "Jane Doe",
    age: 32,
    role: "Administrator",
  },
  {
    name: "Kate Müller",
    age: 23,
    occupation: "Astronaut",
  },
  {
    name: "Bruce Willis",
    age: 64,
    role: "World saver",
  },
];

export function logPerson(person: Person) {
  let additionalInformation: string;
  // TODO: 변경
  if ("role" in person) {
    additionalInformation = person.role;
  } else {
    additionalInformation = person.occupation;
  }
  console.log(` - ${person.name}, ${person.age}, ${additionalInformation}`);
}

persons.forEach(logPerson);
```

## 4️⃣ 4번 문제 ⭕ ( 사용자 정의 타입 가드 )
> [4번 문제](https://typescript-exercises.github.io/#exercise=4&file=%2Findex.ts){:target="_blank"}
{:.prompt-info}

[사용자 정의 타입 가드](/posts/이펙티브-타입스크립트-3장/#3%EF%B8%8F⃣-사용자-정의-타입-가드){:target="_blank"}에 대한 이해가 있는지에 대한 문제입니다.<br />

### 1. 문제
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

export type Person = User | Admin;

export const persons: Person[] = [
  {
    type: "user",
    name: "Max Mustermann",
    age: 25,
    occupation: "Chimney sweep",
  },
  { type: "admin", name: "Jane Doe", age: 32, role: "Administrator" },
  { type: "user", name: "Kate Müller", age: 23, occupation: "Astronaut" },
  { type: "admin", name: "Bruce Willis", age: 64, role: "World saver" },
];

export function isAdmin(person: Person) {
  return person.type === "admin";
}

export function isUser(person: Person) {
  return person.type === "user";
}

export function logPerson(person: Person) {
  let additionalInformation: string = "";
  if (isAdmin(person)) {
    additionalInformation = person.role;
  }
  if (isUser(person)) {
    additionalInformation = person.occupation;
  }
  console.log(` - ${person.name}, ${person.age}, ${additionalInformation}`);
}

console.log("Admins:");
persons.filter(isAdmin).forEach(logPerson);

console.log();

console.log("Users:");
persons.filter(isUser).forEach(logPerson);
```

### 2. 풀이
사용자 정의 타입 가드란 매개변수의 타입을 명시적으로 좁히고 그 좁혀진 타입으로 반환하는 방식으로 동작합니다.<br />
( 즉, 함수를 이용해서 명시적으로 좁혀진 타입으로 반환할 수 있습니다. )<br />

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

export type Person = User | Admin;

export const persons: Person[] = [
  {
    type: "user",
    name: "Max Mustermann",
    age: 25,
    occupation: "Chimney sweep",
  },
  { type: "admin", name: "Jane Doe", age: 32, role: "Administrator" },
  { type: "user", name: "Kate Müller", age: 23, occupation: "Astronaut" },
  { type: "admin", name: "Bruce Willis", age: 64, role: "World saver" },
];

// TODO: 수정
export function isAdmin(person: Person): person is Admin {
  return person.type === "admin";
}

// TODO: 수정
export function isUser(person: Person): person is User {
  return person.type === "user";
}

export function logPerson(person: Person) {
  let additionalInformation: string = "";
  if (isAdmin(person)) {
    additionalInformation = person.role;
  }
  if (isUser(person)) {
    additionalInformation = person.occupation;
  }
  console.log(` - ${person.name}, ${person.age}, ${additionalInformation}`);
}

console.log("Admins:");
persons.filter(isAdmin).forEach(logPerson);

console.log();

console.log("Users:");
persons.filter(isUser).forEach(logPerson);
```

## 5️⃣ 5번 문제 ⭕ ( TypeScript Utility Types )
> [5번 문제](https://typescript-exercises.github.io/#exercise=5&file=%2Findex.ts){:target="_blank"}
{:.prompt-info}

특정 `property`는 제외하고, 모든 `property`에 옵셔널을 넣는 방법에 대한 문제입니다.( `type`을 제외 )<br />

### 1. 문제
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

export type Person = User | Admin;

export const persons: Person[] = [
  {
    type: "user",
    name: "Max Mustermann",
    age: 25,
    occupation: "Chimney sweep",
  },
  {
    type: "admin",
    name: "Jane Doe",
    age: 32,
    role: "Administrator",
  },
  {
    type: "user",
    name: "Kate Müller",
    age: 23,
    occupation: "Astronaut",
  },
  {
    type: "admin",
    name: "Bruce Willis",
    age: 64,
    role: "World saver",
  },
  {
    type: "user",
    name: "Wilson",
    age: 23,
    occupation: "Ball",
  },
  {
    type: "admin",
    name: "Agent Smith",
    age: 23,
    role: "Administrator",
  },
];

export const isAdmin = (person: Person): person is Admin =>
  person.type === "admin";
export const isUser = (person: Person): person is User =>
  person.type === "user";

export function logPerson(person: Person) {
  let additionalInformation = "";
  if (isAdmin(person)) {
    additionalInformation = person.role;
  }
  if (isUser(person)) {
    additionalInformation = person.occupation;
  }
  console.log(` - ${person.name}, ${person.age}, ${additionalInformation}`);
}

export function filterUsers(persons: Person[], criteria: User): User[] {
  return persons.filter(isUser).filter((user) => {
    const criteriaKeys = Object.keys(criteria) as (keyof User)[];
    return criteriaKeys.every((fieldName) => {
      return user[fieldName] === criteria[fieldName];
    });
  });
}

console.log("Users of age 23:");

filterUsers(persons, {
  age: 23,
}).forEach(logPerson);
```

### 2. 풀이
`TypeScript Utility type`인 [`Omit<T, K>`](/posts/TypeScript-UtilityTypes/#-omitt-k){:target="_blank"}를 이용하면 특정 타입을 제외할 수 있고, [`Partial<T>`](/posts/TypeScript-UtilityTypes/#-partialt){:target="_blank"}을 이용하면 모든 타입을 옵셔널로 만들 수 있습니다.<br />

`Omit<T, K>`를 이용해서 `Person`의 `property`인 `type`을 제외하고, `Partial<T>`을 이용해서 `Person`을 모두 옵셔널로 만들면 됩니다.<br />

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

export type Person = User | Admin;

// TODO: 추가
type OptionalUser = Partial<Omit<User, "type">>;

export const persons: Person[] = [
  {
    type: "user",
    name: "Max Mustermann",
    age: 25,
    occupation: "Chimney sweep",
  },
  {
    type: "admin",
    name: "Jane Doe",
    age: 32,
    role: "Administrator",
  },
  {
    type: "user",
    name: "Kate Müller",
    age: 23,
    occupation: "Astronaut",
  },
  {
    type: "admin",
    name: "Bruce Willis",
    age: 64,
    role: "World saver",
  },
  {
    type: "user",
    name: "Wilson",
    age: 23,
    occupation: "Ball",
  },
  {
    type: "admin",
    name: "Agent Smith",
    age: 23,
    role: "Administrator",
  },
];

export const isAdmin = (person: Person): person is Admin =>
  person.type === "admin";
export const isUser = (person: Person): person is User =>
  person.type === "user";

export function logPerson(person: Person) {
  let additionalInformation = "";
  if (isAdmin(person)) {
    additionalInformation = person.role;
  }
  if (isUser(person)) {
    additionalInformation = person.occupation;
  }
  console.log(` - ${person.name}, ${person.age}, ${additionalInformation}`);
}

// TODO: 수정
export function filterUsers(persons: Person[], criteria: OptionalUser): User[] {
  return persons.filter(isUser).filter((user) => {
    // TODO: 수정
    const criteriaKeys = Object.keys(criteria) as (keyof OptionalUser)[];
    return criteriaKeys.every((fieldName) => {
      return user[fieldName] === criteria[fieldName];
    });
  });
}

console.log("Users of age 23:");

filterUsers(persons, {
  age: 23,
}).forEach(logPerson);
```

## 6️⃣ 6번 문제 ❌ ( 함수 오버로딩 )
> [6번 문제](https://typescript-exercises.github.io/#exercise=6&file=%2Findex.ts){:target="_blank"}
{:.prompt-info}

[함수 오버로딩](/posts/TypeScript-Function-Overloading/){:target="_blank"}에 대한 문제입니다.<br />
( 함수 오버로딩에 대한 개념이 없어서 못 풀고 답지를 확인하고 다시 공부하고 정리했습니다... 🥲 )<br />
( 시도했던 방법은 제네릭, 조건부 타입, 사용자 정의 타입 가드를 사용했는데 모두 실패했고 계속 고민하다가 답지 보고 이해했습니다... 😥 )<br />

받는 타입에 따라서 반환하는 타입을 미리 정하는 함수 오버로딩을 통해 타입 체커가 반환 타입을 제대로 추론하도록 설계하는 문제입니다.<br />

### 1. 문제
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

export type Person = User | Admin;

export const persons: Person[] = [
  {
    type: "user",
    name: "Max Mustermann",
    age: 25,
    occupation: "Chimney sweep",
  },
  { type: "admin", name: "Jane Doe", age: 32, role: "Administrator" },
  { type: "user", name: "Kate Müller", age: 23, occupation: "Astronaut" },
  { type: "admin", name: "Bruce Willis", age: 64, role: "World saver" },
  { type: "user", name: "Wilson", age: 23, occupation: "Ball" },
  { type: "admin", name: "Agent Smith", age: 23, role: "Anti-virus engineer" },
];

export function logPerson(person: Person) {
  console.log(
    ` - ${person.name}, ${person.age}, ${
      person.type === "admin" ? person.role : person.occupation
    }`
  );
}

export function filterPersons(
  persons: Person[],
  personType: string,
  criteria: unknown
): unknown[] {
  return persons
    .filter((person) => person.type === personType)
    .filter((person) => {
      let criteriaKeys = Object.keys(criteria) as (keyof Person)[];
      return criteriaKeys.every((fieldName) => {
        return person[fieldName] === criteria[fieldName];
      });
    });
}

export const usersOfAge23 = filterPersons(persons, "user", { age: 23 });
export const adminsOfAge23 = filterPersons(persons, "admin", { age: 23 });

console.log("Users of age 23:");
usersOfAge23.forEach(logPerson);

console.log();

console.log("Admins of age 23:");
adminsOfAge23.forEach(logPerson);
```

### 2. 풀이
[5번 문제](/posts/Typescript-Exercises/#5%EF%B8%8F⃣-5번-문제---typescript-utility-types-)처럼 조건에 맞게 타입을 변경시키고, 매개변수로 각 타입을 받았을 때 반환하는 타입을 설정하는 함수 오버로딩을 이용해서 반환 타입을 추론할 수 있도록 했습니다.<br />

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

export type Person = User | Admin;

export const persons: Person[] = [
  {
    type: "user",
    name: "Max Mustermann",
    age: 25,
    occupation: "Chimney sweep",
  },
  { type: "admin", name: "Jane Doe", age: 32, role: "Administrator" },
  { type: "user", name: "Kate Müller", age: 23, occupation: "Astronaut" },
  { type: "admin", name: "Bruce Willis", age: 64, role: "World saver" },
  { type: "user", name: "Wilson", age: 23, occupation: "Ball" },
  { type: "admin", name: "Agent Smith", age: 23, role: "Anti-virus engineer" },
];

export function logPerson(person: Person) {
  console.log(
    ` - ${person.name}, ${person.age}, ${
      person.type === "admin" ? person.role : person.occupation
    }`
  );
}

// TODO: 추가
type OptionalUser = Partial<Omit<User, "type">>;
type OptionalAdmin = Partial<Omit<Admin, "type">>;
type OptionalPerson = Partial<Omit<Person, "type">>;

// TODO: 추가
const getKeys = <T>(criteria: T) => Object.keys(criteria) as (keyof T)[];

export function filterPersons(
  persons: Person[],
  personType: "user",
  // TODO: 수정
  criteria: OptionalUser
): User[];
export function filterPersons(
  persons: Person[],
  personType: "admin",
  // TODO: 수정
  criteria: OptionalAdmin
): Admin[];
export function filterPersons(
  persons: Person[],
  personType: string,
  // TODO: 수정
  criteria: OptionalPerson
): Person[] {
  return persons
    .filter((person) => person.type === personType)
    .filter((person) => {
      // TODO: 수정
      let criteriaKeys = getKeys(criteria);
      return criteriaKeys.every((fieldName) => {
        return person[fieldName] === criteria[fieldName];
      });
    });
}

export const usersOfAge23 = filterPersons(persons, "user", { age: 23 });
export const adminsOfAge23 = filterPersons(persons, "admin", { age: 23 });

console.log("Users of age 23:");
usersOfAge23.forEach(logPerson);

console.log();

console.log("Admins of age 23:");
adminsOfAge23.forEach(logPerson);
```

## 7️⃣ 7번 문제 ⭕ ( 기본 Generic )
> [7번 문제](https://typescript-exercises.github.io/#exercise=7&file=%2Findex.ts){:target="_blank"}
{:.prompt-info}

매개변수의 타입을 반환하는 타입과 유사하게 만드는 문제입니다.<br />
즉, 제네릭 타입의 사용 방법에 대한 문제입니다.<br />

### 1. 문제
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

function logUser(user: User) {
  const pos = users.indexOf(user) + 1;
  console.log(` - #${pos} User: ${user.name}, ${user.age}, ${user.occupation}`);
}

function logAdmin(admin: Admin) {
  const pos = admins.indexOf(admin) + 1;
  console.log(` - #${pos} Admin: ${admin.name}, ${admin.age}, ${admin.role}`);
}

const admins: Admin[] = [
  {
    type: "admin",
    name: "Will Bruces",
    age: 30,
    role: "Overseer",
  },
  {
    type: "admin",
    name: "Steve",
    age: 40,
    role: "Steve",
  },
];

const users: User[] = [
  {
    type: "user",
    name: "Moses",
    age: 70,
    occupation: "Desert guide",
  },
  {
    type: "user",
    name: "Superman",
    age: 28,
    occupation: "Ordinary person",
  },
];

export function swap(v1, v2) {
  return [v2, v1];
}

function test1() {
  console.log("test1:");
  const [secondUser, firstAdmin] = swap(admins[0], users[1]);
  logUser(secondUser);
  logAdmin(firstAdmin);
}

function test2() {
  console.log("test2:");
  const [secondAdmin, firstUser] = swap(users[0], admins[1]);
  logAdmin(secondAdmin);
  logUser(firstUser);
}

function test3() {
  console.log("test3:");
  const [secondUser, firstUser] = swap(users[0], users[1]);
  logUser(secondUser);
  logUser(firstUser);
}

function test4() {
  console.log("test4:");
  const [firstAdmin, secondAdmin] = swap(admins[1], admins[0]);
  logAdmin(firstAdmin);
  logAdmin(secondAdmin);
}

function test5() {
  console.log("test5:");
  const [stringValue, numericValue] = swap(123, "Hello World");
  console.log(` - String: ${stringValue}`);
  console.log(` - Numeric: ${numericValue}`);
}

[test1, test2, test3, test4, test5].forEach((test) => test());
```

### 2. 풀이
`swap()`의 매개변수를 제네릭으로 받아서 반환 타입을 결정하면 됩니다.<br />

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

function logUser(user: User) {
  const pos = users.indexOf(user) + 1;
  console.log(` - #${pos} User: ${user.name}, ${user.age}, ${user.occupation}`);
}

function logAdmin(admin: Admin) {
  const pos = admins.indexOf(admin) + 1;
  console.log(` - #${pos} Admin: ${admin.name}, ${admin.age}, ${admin.role}`);
}

const admins: Admin[] = [
  {
    type: "admin",
    name: "Will Bruces",
    age: 30,
    role: "Overseer",
  },
  {
    type: "admin",
    name: "Steve",
    age: 40,
    role: "Steve",
  },
];

const users: User[] = [
  {
    type: "user",
    name: "Moses",
    age: 70,
    occupation: "Desert guide",
  },
  {
    type: "user",
    name: "Superman",
    age: 28,
    occupation: "Ordinary person",
  },
];

// TODO: 수정
export function swap<T, U>(v1: T, v2: U): [U, T] {
  return [v2, v1];
}

function test1() {
  console.log("test1:");
  const [secondUser, firstAdmin] = swap(admins[0], users[1]);
  logUser(secondUser);
  logAdmin(firstAdmin);
}

function test2() {
  console.log("test2:");
  const [secondAdmin, firstUser] = swap(users[0], admins[1]);
  logAdmin(secondAdmin);
  logUser(firstUser);
}

function test3() {
  console.log("test3:");
  const [secondUser, firstUser] = swap(users[0], users[1]);
  logUser(secondUser);
  logUser(firstUser);
}

function test4() {
  console.log("test4:");
  const [firstAdmin, secondAdmin] = swap(admins[1], admins[0]);
  logAdmin(firstAdmin);
  logAdmin(secondAdmin);
}

function test5() {
  console.log("test5:");
  const [stringValue, numericValue] = swap(123, "Hello World");
  console.log(` - String: ${stringValue}`);
  console.log(` - Numeric: ${numericValue}`);
}

[test1, test2, test3, test4, test5].forEach((test) => test());
```

## 8️⃣ 8번 문제 ⭕ ( Intersection )
> [8번 문제](https://typescript-exercises.github.io/#exercise=8&file=%2Findex.ts){:target="_blank"}
{:.prompt-info}

특정 타입을 활용해서 새로운 타입을 만드는 문제입니다.<br />
즉, `intersection`(`&`)의 사용 방법에 대한 문제입니다.<br />

### 1. 문제
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

type PowerUser = unknown;

export type Person = User | Admin | PowerUser;

export const persons: Person[] = [
  {
    type: "user",
    name: "Max Mustermann",
    age: 25,
    occupation: "Chimney sweep",
  },
  { type: "admin", name: "Jane Doe", age: 32, role: "Administrator" },
  { type: "user", name: "Kate Müller", age: 23, occupation: "Astronaut" },
  { type: "admin", name: "Bruce Willis", age: 64, role: "World saver" },
  {
    type: "powerUser",
    name: "Nikki Stone",
    age: 45,
    role: "Moderator",
    occupation: "Cat groomer",
  },
];

function isAdmin(person: Person): person is Admin {
  return person.type === "admin";
}

function isUser(person: Person): person is User {
  return person.type === "user";
}

function isPowerUser(person: Person): person is PowerUser {
  return person.type === "powerUser";
}

export function logPerson(person: Person) {
  let additionalInformation: string = "";
  if (isAdmin(person)) {
    additionalInformation = person.role;
  }
  if (isUser(person)) {
    additionalInformation = person.occupation;
  }
  if (isPowerUser(person)) {
    additionalInformation = `${person.role}, ${person.occupation}`;
  }
  console.log(`${person.name}, ${person.age}, ${additionalInformation}`);
}

console.log("Admins:");
persons.filter(isAdmin).forEach(logPerson);

console.log();

console.log("Users:");
persons.filter(isUser).forEach(logPerson);

console.log();

console.log("Power users:");
persons.filter(isPowerUser).forEach(logPerson);
```

### 2. 풀이
`Omit<T, K>`와 `intersection`(`&`)을 이용해서 기존 타입들을 기반으로 새로운 타입을 만드는 문제입니다.<br />

중요한 것은 기존 타입을 기반으로 만들어서 기존 타입에 의존적이게 만드는 것입니다.<br />
기존 타입에 의존적이라면 기존 타입이 바뀌면 새로운 타입은 자동으로 동기화되는 이점이 있습니다.<br />

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

// TODO: 수정
type PowerUser = Omit<User, "type"> & Omit<Admin, "type"> & { type: "powerUser" };

export type Person = User | Admin | PowerUser;

export const persons: Person[] = [
  {
    type: "user",
    name: "Max Mustermann",
    age: 25,
    occupation: "Chimney sweep",
  },
  { type: "admin", name: "Jane Doe", age: 32, role: "Administrator" },
  { type: "user", name: "Kate Müller", age: 23, occupation: "Astronaut" },
  { type: "admin", name: "Bruce Willis", age: 64, role: "World saver" },
  {
    type: "powerUser",
    name: "Nikki Stone",
    age: 45,
    role: "Moderator",
    occupation: "Cat groomer",
  },
];

function isAdmin(person: Person): person is Admin {
  return person.type === "admin";
}

function isUser(person: Person): person is User {
  return person.type === "user";
}

function isPowerUser(person: Person): person is PowerUser {
  return person.type === "powerUser";
}

export function logPerson(person: Person) {
  let additionalInformation: string = "";
  if (isAdmin(person)) {
    additionalInformation = person.role;
  }
  if (isUser(person)) {
    additionalInformation = person.occupation;
  }
  if (isPowerUser(person)) {
    additionalInformation = `${person.role}, ${person.occupation}`;
  }
  console.log(`${person.name}, ${person.age}, ${additionalInformation}`);
}

console.log("Admins:");
persons.filter(isAdmin).forEach(logPerson);

console.log();

console.log("Users:");
persons.filter(isUser).forEach(logPerson);

console.log();

console.log("Power users:");
persons.filter(isPowerUser).forEach(logPerson);
```

## 9️⃣ 9번 문제 ⭕ ( 활용 Generic )
> [9번 문제](https://typescript-exercises.github.io/#exercise=9&file=%2Findex.ts){:target="_blank"}
{:.prompt-info}

제네릭을 활용해서 비슷한 유형을 여러 타입들을 하나의 타입으로 처리하는 문제입니다.<br />

### 1. 문제
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

const admins: Admin[] = [
  { type: "admin", name: "Jane Doe", age: 32, role: "Administrator" },
  { type: "admin", name: "Bruce Willis", age: 64, role: "World saver" },
];

const users: User[] = [
  {
    type: "user",
    name: "Max Mustermann",
    age: 25,
    occupation: "Chimney sweep",
  },
  { type: "user", name: "Kate Müller", age: 23, occupation: "Astronaut" },
];

export type ApiResponse<T> = unknown;

type AdminsApiResponse =
  | {
      status: "success";
      data: Admin[];
    }
  | {
      status: "error";
      error: string;
    };

export function requestAdmins(callback: (response: AdminsApiResponse) => void) {
  callback({
    status: "success",
    data: admins,
  });
}

type UsersApiResponse =
  | {
      status: "success";
      data: User[];
    }
  | {
      status: "error";
      error: string;
    };

export function requestUsers(callback: (response: UsersApiResponse) => void) {
  callback({
    status: "success",
    data: users,
  });
}

export function requestCurrentServerTime(
  callback: (response: unknown) => void
) {
  callback({
    status: "success",
    data: Date.now(),
  });
}

export function requestCoffeeMachineQueueLength(
  callback: (response: unknown) => void
) {
  callback({
    status: "error",
    error: "Numeric value has exceeded Number.MAX_SAFE_INTEGER.",
  });
}

function logPerson(person: Person) {
  console.log(
    ` - ${person.name}, ${person.age}, ${
      person.type === "admin" ? person.role : person.occupation
    }`
  );
}

function startTheApp(callback: (error: Error | null) => void) {
  requestAdmins((adminsResponse) => {
    console.log("Admins:");
    if (adminsResponse.status === "success") {
      adminsResponse.data.forEach(logPerson);
    } else {
      return callback(new Error(adminsResponse.error));
    }

    console.log();

    requestUsers((usersResponse) => {
      console.log("Users:");
      if (usersResponse.status === "success") {
        usersResponse.data.forEach(logPerson);
      } else {
        return callback(new Error(usersResponse.error));
      }

      console.log();

      requestCurrentServerTime((serverTimeResponse) => {
        console.log("Server time:");
        if (serverTimeResponse.status === "success") {
          console.log(
            `   ${new Date(serverTimeResponse.data).toLocaleString()}`
          );
        } else {
          return callback(new Error(serverTimeResponse.error));
        }

        console.log();

        requestCoffeeMachineQueueLength((coffeeMachineQueueLengthResponse) => {
          console.log("Coffee machine queue length:");
          if (coffeeMachineQueueLengthResponse.status === "success") {
            console.log(`   ${coffeeMachineQueueLengthResponse.data}`);
          } else {
            return callback(new Error(coffeeMachineQueueLengthResponse.error));
          }

          callback(null);
        });
      });
    });
  });
}

startTheApp((e: Error | null) => {
  console.log();
  if (e) {
    console.log(
      `Error: "${e.message}", but it's fine, sometimes errors are inevitable.`
    );
  } else {
    console.log("Success!");
  }
});
```

### 2. 나만의 풀이
제가 풀이한 방식은 답지와는 다르지만 버리기는 아까워서 가져왔습니다.<br />
[조건부 타입](){:target="_blank"}을 이용해서 매개변수로 `Person` 타입이 들어오면 배열 타입 정하고, 그게 아니면 일반 타입으로 정하는 방식으로 풀었습니다.<br />

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

const admins: Admin[] = [
  { type: "admin", name: "Jane Doe", age: 32, role: "Administrator" },
  { type: "admin", name: "Bruce Willis", age: 64, role: "World saver" },
];

const users: User[] = [
  {
    type: "user",
    name: "Max Mustermann",
    age: 25,
    occupation: "Chimney sweep",
  },
  { type: "user", name: "Kate Müller", age: 23, occupation: "Astronaut" },
];

// TODO: 수정
export type ApiResponse<T = Person> =
  | {
      status: "success";
      data: T extends Person ? T[] : T;
    }
  | {
      status: "error";
      error: string;
    };

// TODO: "AdminsApiResponse" 제거

export function requestAdmins(
  // TODO: 수정
  callback: (response: ApiResponse<Admin>) => void
) {
  callback({
    status: "success",
    data: admins,
  });
}

// TODO: "UsersApiResponse" 제거

// TODO: 수정
export function requestUsers(callback: (response: ApiResponse<User>) => void) {
  callback({
    status: "success",
    data: users,
  });
}

export function requestCurrentServerTime(
  // TODO: 수정
  callback: (response: ApiResponse<number>) => void
) {
  callback({
    status: "success",
    data: Date.now(),
  });
}

export function requestCoffeeMachineQueueLength(
  // TODO: 수정
  callback: (response: ApiResponse<number>) => void
) {
  callback({
    status: "error",
    error: "Numeric value has exceeded Number.MAX_SAFE_INTEGER.",
  });
}

function logPerson(person: Person) {
  console.log(
    ` - ${person.name}, ${person.age}, ${
      person.type === "admin" ? person.role : person.occupation
    }`
  );
}

function startTheApp(callback: (error: Error | null) => void) {
  requestAdmins((adminsResponse) => {
    console.log("Admins:");
    if (adminsResponse.status === "success") {
      adminsResponse.data.forEach(logPerson);
    } else {
      return callback(new Error(adminsResponse.error));
    }

    console.log();

    requestUsers((usersResponse) => {
      console.log("Users:");
      if (usersResponse.status === "success") {
        usersResponse.data.forEach(logPerson);
      } else {
        return callback(new Error(usersResponse.error));
      }

      console.log();

      requestCurrentServerTime((serverTimeResponse) => {
        console.log("Server time:");
        if (serverTimeResponse.status === "success") {
          console.log(
            `   ${new Date(serverTimeResponse.data).toLocaleString()}`
          );
        } else {
          return callback(new Error(serverTimeResponse.error));
        }

        console.log();

        requestCoffeeMachineQueueLength((coffeeMachineQueueLengthResponse) => {
          console.log("Coffee machine queue length:");
          if (coffeeMachineQueueLengthResponse.status === "success") {
            console.log(`   ${coffeeMachineQueueLengthResponse.data}`);
          } else {
            return callback(new Error(coffeeMachineQueueLengthResponse.error));
          }

          callback(null);
        });
      });
    });
  });
}

startTheApp((e: Error | null) => {
  console.log();
  if (e) {
    console.log(
      `Error: "${e.message}", but it's fine, sometimes errors are inevitable.`
    );
  } else {
    console.log("Success!");
  }
});
```

### 3. 답지의 풀이
답지의 풀이가 저랑 달라서 따로 작성하겠습니다.<br />
( 답지랑 완벽하게 같은 풀이는 아닙니다. )<br />

제가 생각하지 못한 방법이 제네릭으로 배열 타입을 받는 방법입니다.<br />
애초에 `ApiResponse<User[]>`형태로 받으면 되는데 `ApiResponse<User>`로만 받으려고 해서 위처럼 조건부 타입을 이용해서 타입에 맞게 배열 타입인지 아닌지를 결정했습니다.<br />

답지처럼 푸는 게 조금 더 유연하게 타입을 받을 수 있어서 좋은 것 같습니다.<br />
근데 또 다르게 생각해보면 `ApiResponse`로 정해진 타입이 아닌 다른 타입이 올 수 없기 때문에 타입을 명시적으로 작성하지 않으면 사용할 수 없습니다.<br />
따라서 더 귀찮지만 "조금 더 안정성 있는 타입을 만들도록 설계됐다고 볼 수 있지 않을까?"라고 개인적으로 생각해봅니다... 🤔

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

const admins: Admin[] = [
  { type: "admin", name: "Jane Doe", age: 32, role: "Administrator" },
  { type: "admin", name: "Bruce Willis", age: 64, role: "World saver" },
];

const users: User[] = [
  {
    type: "user",
    name: "Max Mustermann",
    age: 25,
    occupation: "Chimney sweep",
  },
  { type: "user", name: "Kate Müller", age: 23, occupation: "Astronaut" },
];

// TODO: 수정
export type ApiResponse<T = unknown> =
  | {
      status: "success";
      data: T;
    }
  | {
      status: "error";
      error: string;
    };

export function requestAdmins(
  // TODO: 수정
  callback: (response: ApiResponse<Admin[]>) => void
) {
  callback({
    status: "success",
    data: admins,
  });
}

export function requestUsers(
  // TODO: 수정
  callback: (response: ApiResponse<User[]>) => void
) {
  callback({
    status: "success",
    data: users,
  });
}

export function requestCurrentServerTime(
  // TODO: 수정
  callback: (response: ApiResponse<number>) => void
) {
  callback({
    status: "success",
    data: Date.now(),
  });
}

export function requestCoffeeMachineQueueLength(
  // TODO: 수정
  callback: (response: ApiResponse) => void
) {
  callback({
    status: "error",
    error: "Numeric value has exceeded Number.MAX_SAFE_INTEGER.",
  });
}

function logPerson(person: Person) {
  console.log(
    ` - ${person.name}, ${person.age}, ${
      person.type === "admin" ? person.role : person.occupation
    }`
  );
}

function startTheApp(callback: (error: Error | null) => void) {
  requestAdmins((adminsResponse) => {
    console.log("Admins:");
    if (adminsResponse.status === "success") {
      adminsResponse.data.forEach(logPerson);
    } else {
      return callback(new Error(adminsResponse.error));
    }

    console.log();

    requestUsers((usersResponse) => {
      console.log("Users:");
      if (usersResponse.status === "success") {
        usersResponse.data.forEach(logPerson);
      } else {
        return callback(new Error(usersResponse.error));
      }

      console.log();

      requestCurrentServerTime((serverTimeResponse) => {
        console.log("Server time:");
        if (serverTimeResponse.status === "success") {
          console.log(
            `   ${new Date(serverTimeResponse.data).toLocaleString()}`
          );
        } else {
          return callback(new Error(serverTimeResponse.error));
        }

        console.log();

        requestCoffeeMachineQueueLength((coffeeMachineQueueLengthResponse) => {
          console.log("Coffee machine queue length:");
          if (coffeeMachineQueueLengthResponse.status === "success") {
            console.log(`   ${coffeeMachineQueueLengthResponse.data}`);
          } else {
            return callback(new Error(coffeeMachineQueueLengthResponse.error));
          }

          callback(null);
        });
      });
    });
  });
}

startTheApp((e: Error | null) => {
  console.log();
  if (e) {
    console.log(
      `Error: "${e.message}", but it's fine, sometimes errors are inevitable.`
    );
  } else {
    console.log("Success!");
  }
});
```

# 📮 레퍼런스
1. [typescript exercises](https://typescript-exercises.github.io/){:target="_blank"}
2. [1-blue - `union`(`|`)과 `intersection`(`&`)](/posts/이펙티브-타입스크립트-2장/#1%EF%B8%8F⃣-union-vs-intersection){:target="_blank"}
3. [1-blue - 타입 좁히기 / 타입 가드](/posts/이펙티브-타입스크립트-3장/#-item-22--타입-좁히기-){:target="_blank"}
4. [1-blue - 사용자 정의 타입 가드](/posts/이펙티브-타입스크립트-3장/#3%EF%B8%8F⃣-사용자-정의-타입-가드){:target="_blank"}
5. [1-blue - `Omit<T, K>`](/posts/TypeScript-UtilityTypes/#-omitt-k){:target="_blank"}
6. [1-blue - `Partial<T>`](/posts/TypeScript-UtilityTypes/#-partialt){:target="_blank"}
7. [1-blue - 함수 오버로딩](/posts/TypeScript-Function-Overloading/){:target="_blank"}
8. [1-blue - 조건부 타입](){:target="_blank"}