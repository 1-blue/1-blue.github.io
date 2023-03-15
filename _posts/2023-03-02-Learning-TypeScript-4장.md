---
layout: post
title: 러닝 타입스크립트 4장
author: admin
date: 2023-03-02 19:24:00 +900
lastmod: 2023-03-15 08:42:00 +900
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

> 해당 포스트는 `러닝 타입스크립트` 4장을 읽고 정리한 포스트입니다.<br />책의 모든 내용을 작성하는 것이 아닌 주관적인 기준에 따라 필요한 정보만 정리했습니다.<br />
{: .prompt-info}

# 🎈 객체 타입

## 0️⃣ 객체 타입 선언
```ts
let myValue: {
  name: string;
  age: number;
};

myValue = {
  name: "Aatrox",
  age: 26,
};
```

## 1️⃣ 객체 타입 별칭
```ts
type MyValue = {
  name: string;
  age: number;
};

let myValue: MyValue;

myValue = {
  name: "Aatrox",
  age: 26,
};
```

# 🦆 구조적 타이핑
타입을 충족하면 값이 추가적으로 있어도 사용할 수 있는 것을 의미합니다.<br />
( 조금 더 자세한 예시를 보고 싶다면 [여기](/posts/이펙티브-타입스크립트-1장/#-item-4--구조적-타이핑에-익숙해지기-){:target="_blank"}에서 확인해주세요! )

`(1)`과 `(2)`처럼 타입과 완벽하게 일치하지 않는 값을 대입해도 문제 없이 동작합니다.<br />

```ts
type FirstName = { firstName: string };
type LastName = { lastName: string };

const fullName = {
  firstName: "a",
  lastName: "b",
};

// (1)
let firstName: FirstName = fullName;
// (2)
let lastName: LastName = fullName;
```

+ 덕 타이핑: 런타임에서 사용될 때까지 객체 타입을 검사하지 않는 것을 의미 ( `JavaScript` )
+ 구조적 타이핑: 정적 시스템이 타입을 검사하는 것을 의미 ( `TypeScript` )

## 0️⃣ 사용 검사
객체 타입에 정의한 값을 필수적으로 선언해야 합니다. ( `(1)` )<br />
또한 타입의 정보와도 일치해야 합니다. ( `(2)` )<br />

```ts
type Person = {
  name: string;
  age: number;
};

// (1) Error: Property 'age' is missing in type '{ name: string; }' but required in type 'Person'
const person: Person = {
  name: "Aatrox",
};

// (2) Error: Type 'string' is not assignable to type 'number'
const person: Person = {
  name: "Aatrox",
  age: "10",
};
```

## 1️⃣ 초과 속성 검사
정해진 타입보다 더 많은 속성을 부여하면 타입 오류가 발생합니다. ( `(3)` )<br />
하지만 객체 리터럴이 아닌 경우에는 구조적 타이핑에 의해서 오류가 발생하지 않습니다. ( `(4)` )<br />

```ts
type Person = {
  name: string;
  age: number;
};

// (3)
const person1: Person = {
  name: "Aatrox",
  age: 10,

  // Error: Type '{ name: string; age: number; gender: boolean; }' is not assignable to type 'Person'.
  // Error: Object literal may only specify known properties, and 'gender' does not exist in type 'Person
  gender: true,
};

// (4)
const person = {
  name: "Aatrox",
  age: 26,
  gender: true,
};
// (4)
const person2: Person = person;

// Error: Property 'gender' does not exist on type 'Person'.
person2.gender;
```

## 2️⃣ 중첩된 객체 타입
객체의 속성으로 타입을 사용할 수 있기 때문에 `(5)`처럼 사용하는 것보다는 `(6)`처럼 사용하는 것이 더 좋습니다.<br />
가독성도 더 좋고, 오류 메시지도 더 명확해집니다.<br />

```ts
type Vision = {
  left: number;
  right: number;
};

// (5)
type Person1 = {
  name: string;
  age: number;
  vision: {
    left: number;
    right: number;
  };
};

// (6)
type Person2 = {
  name: string;
  age: number;
  vision: Vision;
};
```

## 3️⃣ 선택적 속성
`?:`을 이용해서 선택적 속성을 부여할 수 있습니다.<br />
선택적 속성은 존재하지 않아도 되고 `undefined`를 넣어줘도 됩니다. ( `(7)` )<br />
하지만 `undefined`를 유니언으로 선언했다면 속성 값을 반드시 선언해줘야합니다. ( `(8)` )<br />

```ts
type Person = {
  name: string;
  age?: number;
  gender: boolean | undefined;
};
/**
 * type Person = {
 *   name: string;
 *   age?: number | undefined;
 *   gender: boolean | undefined;
 * }
 */ 

// (7)
const person1: Person = {
  name: "Aatrox",
  age: 26,
  gender: false,
};

// (8) Error: Property 'gender' is missing in type '{ name: string; age: number; }' but required in type 'Person'
const person2: Person = {
  name: "Aatrox",
  age: 26,
};

// (7) 문제 없음
const person3: Person = {
  name: "Aatrox",
  gender: false,
};

// (8) "undefined"라도 반드시 선언해줘야 함
const person4: Person = {
  name: "Aatrox",
  age: 26,
  gender: undefined;
};
```

# 🌌 객체 타입 유니온
## 0️⃣ 유추된 객체 타입 유니온
여러 가지 객체가 될 수 있는 초깃값을 할당하면 객체들의 유니온 타입으로 추론합니다.<br />

```ts
const person =
  Math.random() > 0.5 ? { name: "AA", age: 2 } : { name: "BB", gender: true };
/**
  * const person: {
  *   name: string;
  *   age: number;
  *   gender?: undefined;
  * } | {
  *   name: string;
  *   gender: boolean;
  *   age?: undefined;
  * }
  */

person.name;
person.gender;
person.age;
```

## 1️⃣ 명시된 객체 타입 유니온과 객체 타입 내로잉
값의 타입이 객체 타입인 유니온이라면 일반적으로는 모든 유니온에 존재하는 값만 사용할 수 있습니다. (`(1)`)<br />

하지만 타입 내로잉을 사용하면 특정 객체 타입으로 좁혀서 정확하게 타입을 추론하고 사용할 수 있습니다.(`(2)`)<br />

```ts
type Person1 = {
  name: string;
  age: number;
};
type Person2 = {
  name: string;
  gender: boolean;
};

let person: Person1 | Person2;

person =
  Math.random() > 0.5 ? { name: "AA", age: 2 } : { name: "BB", gender: true };

// (1)
person.name;
// Error: Property 'age' does not exist on type 'Person1 | Person2'.
person.age;
// Error: Property 'gender' does not exist on type 'Person1 | Person2'.
person.gender;

// (2)
if ("age" in person) {
  person.age;
} else {
  person.gender;
}
```

## 2️⃣ 판별된 유니온
판별된 유니온을 이용해서 타입 내로잉을 하는 것이 좋습니다.<br />

판별된 유니온이란 각 객체마다 각자의 태그를 붙여서 해당 태그로 타입을 구분할 수 있도록 판별하는 것을 의미합니다.<br />

```ts
type Person1 = {
  name: string;
  age: number;
  type: "person1";
};
type Person2 = {
  name: string;
  gender: boolean;
  type: "person2";
};

let person: Person1 | Person2;

person =
  Math.random() > 0.5
    ? { name: "AA", age: 2, type: "person1" }
    : { name: "BB", gender: true, type: "person2" };

if (person.type === "person1") {
  person.age;
} else {
  person.gender;
}
```

# 🏁 교차 타입
인터섹션(`&`)을 이용하고 각 타입에 선언된 모든 속성의 합집합인 타입이 됩니다.<br />

```ts
type Person1 = {
  name: string;
  age: number;
};
type Person2 = {
  name: string;
  gender: boolean;
};

declare const person: Person1 & Person2;

person.name;
person.age;
person.gender;
```

교차 타입과 판별된 유니온을 같이 사용하면 아래와 같습니다.<br />

```ts
type Person1 = {
  age: number;
  type: "person1"
};
type Person2 = {
  gender: boolean;
  type: "person2"
};

declare const person: { name: string } & ( Person1 | Person2 );

// "name: string"을 가지면서 "Person1"인 경우
if(person.type === "person1")  {
  person.age;
  person.name;
}
// "name: string"을 가지면서 "Person2"인 경우
else {
  person.name;
  person.gender;
}
```

## 0️⃣ 교차 타입의 위험성
교차 타입을 잘못 사용하면 불가능한 타입인 `never`가 생성될 위험이 있습니다.<br />

```ts
// type MyType = never
type MyType = number & string;

type T1 = {
  name: string;
};
type T2 = {
  name: number;
};

// T: { name: never }
type T = T1 & T2;
```

# 📮 레퍼런스
1. « 러닝 타입스크립트 4장 » ( 조시 골드버그 지음, 고승원 옮김, 한빛미디어, 2023 )

1. [1- blue - 구조적 타이핑](/posts/이펙티브-타입스크립트-1장/#-item-4--구조적-타이핑에-익숙해지기-){:target="_blank"}