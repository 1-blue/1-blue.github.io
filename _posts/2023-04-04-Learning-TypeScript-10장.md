---
layout: post
title: 러닝 타입스크립트 10장 ( 제네릭 )
author: admin
date: 2023-04-04 23:01:00 +900
lastmod: 2023-04-12 08:39:00 +900
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

> 해당 포스트는 `러닝 타입스크립트` 10장을 읽고 정리한 포스트입니다.<br />책의 모든 내용을 작성하는 것이 아닌 주관적인 기준에 따라 필요한 정보만 정리했습니다.<br />
{: .prompt-info}


+ 제네릭: 타입간의 관계
+ 제네릭 타입 매개변수: 타입으로 받을 수 있는 변수들
+ 타입 인수: 각 인스턴스 내에서 일관성있게 유지되는 타입

# 🪔 제네릭 함수
제네릭이란 타입 자체를 변수로 사용하는 것을 의미하고 `<`와 `>`를 사용해서 정의합니다.<br />
함수에서는 괄호 앞에 제네릭을 선언합니다.<br />

```ts
function func1<T>(value: T): T{
  return value;
}
const func2 = <T>(value: T): T => value;

// JSX와 충돌나지 않기 위해서 extends 사용
const func3 = <T extends any>(value: T): T => value;
```

제네릭은 호출되는 방식에 따라서 타입 인수를 추론하기 때문에 필요한 경우가 아니라면 명시적으로 타입을 작성해주지 않아도 됩니다.<br />
단, 생각보다 구체적으로 추론하기 때문에 넓은 범위로 사용하기 위해서는 타입 인수를 작성하는 것이 좋습니다.<br />

```ts
// v1: "10"
const v1 = func2("10");

// v2: string
const v2 = func2<string>("10");
```

## 0️⃣ 명시적 제네릭 호출 타입
함수가 호출되는 방식에 의해서 타입을 추론해주지만, 추론하지 못하는 경우 `(1)`처럼 `unknown`으로 추론하게 됩니다.<br />

그런 경우를 방지하기 위해서는 `(2)`처럼 필요한 곳에 명시적으로 타입을 정의하거나, `(3)`처럼 **명시적 제네릭 타입 인수**를 사용해서 함수의 타입을 지정할 수 있습니다.<br />

```ts
const logWrapper =
  <T>(callback: (input: T) => void) =>
  (input: T) =>
    callback(input);

// (1)
const logUnknown = logWrapper((input) => {
  // input: unknown
  console.log(input);
});

// (2)
const logString1 = logWrapper((input: string) => {
  // input: string
  console.log(input);
});
// (3)
const logString2 = logWrapper<string>((input) => {
  // input: string
  console.log(input);
});
```

## 1️⃣ 다중 함수 타입 매개변수
> 타입 매개변수를 세 개 이상 사용하는 것은 권장하지 않습니다.<br />
{:.prompt-info}

제네릭은 `(4)`처럼 여러 개의 타입 매개변수를 받을 수 있습니다.<br />
사용하는 것은 동일하게 사용하면 되지만, `(5)`처럼 명시적으로 사용하는 경우 일부분의 타입만 작성하면 에러가 발생합니다.<br />
즉, 제네릭 타입은 모두 선언하지 않거나, 모두 선언해야 합니다.<br />

```ts
// (4)
const makeTuple = <T1, T2>(v1: T1, v2: T2) => [v1, v2] as const;

const tuple1 = makeTuple("x", 1);

const tuple2 = makeTuple<string, number>("x", 1);

// (5) Error: Expected 2 type arguments, but got 1.
const tuple3 = makeTuple<string>("x", 1);
```

# 🗿 제네릭 인터페이스
제네릭 함수처럼 인터페이스에서도 동일하게 사용할 수 있습니다.<br />

`(1)`처럼 제네릭 인터페이스는 알아서 타입을 추론해주지 않습니다.<br />
따라서 항상 명시적으로 작성해줘야 합니다.<br />

```ts
interface Box<T> {
  inside: T;
}

// (1) Error: Generic type 'Box<T>' requires 1 type argument(s).
const box0: Box = { inside: 10 };

const box1: Box<number> = { inside: 10 };
const box2: Box<string> = { inside: "10" };
```

## 0️⃣ 유추된 제네릭 인터페이스 타입
제네릭 인터페이스는 명시적으로 타입을 작성해줘야 하지만 다른 제네릭을 이용하면 유추된 타입을 사용할 수 있습니다.<br />

제네릭 자체의 타입은 추론이 안되지만, `(2)`의 함수의 제네릭을 타입으로 넘겨줬기 때문에 함수의 타입을 보고 인터페이스의 타입을 결정하게 됩니다.<br />
즉, `(2)`의 타입 인수의 타입은 `(3)`의 `thirdNode`의 타입에 의해 `<string>`으로 추론되게 됩니다.<br />

```ts
// (1)
interface LinkedNode<T> {
  next?: LinkedNode<T> | null;
  value: T;
}

// (2)
const getLastNode = <U>(node: LinkedNode<U>): U =>
  node.next ? getLastNode(node.next) : node.value;

const firstNode: LinkedNode<string> = {
  value: "first",
  next: null,
};
const secondNode: LinkedNode<string> = {
  value: "second",
  next: firstNode,
};
const thirdNode: LinkedNode<string> = {
  value: "third",
  next: secondNode,
};

// (3) lastNode === firstNode; // true
const lastNode = getLastNode(thirdNode);

// 굳이 명시적으로 <string>을 작성해주지 않아도 됨
// const lastNode = getLastNode<string>(thirdNode);
```

# ⚔️ 제네릭 클래스
제네릭 함수나 인터페이스처럼 클래스에도 제네릭을 사용할 수 있습니다.<br />
제네릭 함수처럼 명시적으로 작성해주지 않아도 사용하는 시점에 생성자의 매개변수를 보고 타입 인수를 추론합니다.<br />

```ts
class MyObject<K, V> {
  key: K;
  value: V;

  constructor(key: K, value: V) {
    this.key = key;
    this.value = value;
  }

  getValue(key: K) {
    return this.key === key ? this.value : null;
  }
}

const myObject1 = new MyObject("key", 10);
const myObject2 = new MyObject<string, number>("key", 10);
```

## 0️⃣ 명시적 제네릭 클래스 타입
직접적으로 타입 매개변수를 작성해줘도 되고 제네릭 함수처럼 추론이 가능하다면 추론을 하도록 놔둬도 됩니다.<br />
[명시적 제네릭 함수 호출 타입](/posts/Learning-TypeScript-10장/#0%EF%B8%8F⃣-명시적-제네릭-호출-타입)처럼 추론이 불가능하고 명시적으로 작성하지 않으면 `unknown`으로 추론하게 됩니다.<br />

```ts
class MyClass<T> {
  private callback: (input: T) => void;

  constructor(callback: (input: T) => void) {
    this.callback = callback;
  }
}

// "unknown"으로 추론
new MyClass((v) => {});

// "string"으로 추론
new MyClass((v: string) => {});

// "string"으로 지정
new MyClass<string>((v) => {});
```

## 1️⃣ 제네릭 클래스 확장
제네릭 클래스를 확장하는 경우에는 `(1)`처럼 명시적으로 타입을 적어줘야 합니다.<br />
( 제네릭 클래스를 확장할 때는 제네릭 타입을 추론해주지 않습니다. )<br />

```ts
class MyClass<T> {
  private value: T;

  constructor(value: T) {
    this.value = value;
  }
}

// (1)
class SubMyClass<U> extends MyClass<U> {
  private key: U;

  constructor(value: U, key: U) {
    super(value);
    this.key = key;
  }
}
```

## 2️⃣ 제네릭 인터페이스 구현
클래스에 제네릭 인터페이스를 구현하도록 하는 것도 유사한 형태로 사용하면 됩니다.<br />

```ts
interface IMyClass<T> {
  value: T;
}

class MyClass1<T> implements IMyClass<T> {
  value: T;

  constructor(value: T) {
    this.value = value;
  }
}

class MyClass2 implements IMyClass<string> {
  value: string;

  constructor(value: string) {
    this.value = value;
  }
}

const myClass1 = new MyClass1(10);

// Error: Argument of type 'number' is not assignable to parameter of type 'string'.
const myClass2 = new MyClass2(10);
```

## 3️⃣ 메서드 제네릭
제네릭 함수와 동일한 형태로 사용하면 됩니다.<br />

```ts
class MyClass<T> {
  value: T;

  constructor(value: T) {
    this.value = value;
  }

  getValue<T>(v: T) {
    return v;
  }
}

const myClass = new MyClass(10);

// v1: 10
const v1 = myClass.getValue(10);
// v2: number
const v2 = myClass.getValue<number>(10);
```

## 4️⃣ 정적 클래스 제네릭
정적 멤버나 정적 메서드는 클래스의 특정 인스턴스와 연관되어있지 않기 때문에 클래스의 제네릭 즉, 인스턴스의 제네릭에 접근할 수 없습니다.<br />

아래의 타입 인수인 `T`는 생성될 인스턴스에 사용할 타입 인수이기 때문에 `static`한 변수, 메서드에서 사용할 수 없습니다.<br />

```ts
class MyClass<T> {
  // Error: Static members cannot reference class type parameters
  static key: T;

  static staticLog<U>(value: U) {
    // Error: Static members cannot reference class type parameters
    let v: T;

    console.log(value);
  }
}
```

# 🪜 제네릭 타입 별칭
타입 별칭에도 제네릭을 사용할 수 있습니다.<br />

```ts
type Value<T> = T;

const value: Value<string> = "10";
```

하지만 위처럼 사용하면 단순히 타입을 적용하는 것보다 더 번거롭기 때문에 일반적으로 함수와 함께 사용합니다.<br />

```ts
type Func<T> = (v: T) => T;

const func: Func<number> = (n) => n;
```

## 0️⃣ 제네릭 판별된 유니언
제네릭과 판별된 유니언을 사용하면 원하는 타입에만 제네릭을 적용해서 타입을 적용할 수 있습니다.<br />

이 코드의 장점은 기본 형태는 그대로 유지하되 서로 다른 타입을 적용한 타입을 원하는 대로 사용할 수 있습니다.<br />

```ts
interface Failure {
  succeeded: false;
  error: Error;
}
interface Successful<T> {
  succeeded: true;
  data: T;
}

type Result<T> = Failure | Successful<T>;

const handleResult = (result: Result<string>) => {
  if (result.succeeded) {
    result.data; // string
  } else {
    result.error; // Error
  }
};
```

# 🩻 제네릭 제한자
## 0️⃣ 제네릭 기본값
제네릭에도 기본 매개변수처럼 기본 타입을 정할 수 있습니다.<br />
원래 인터페이스는 타입을 명시적으로 작성해야 하지만 기본 타입을 지정함으로써 명시적으로 작성하지 않아도 `string`으로 정의됩니다.<br />

```ts
interface Person<T = string> {
  name: T;
}

const person1: Person = { name: "" };
const person2: Person<string> = { name: "" };
```

만약 여러 타입 매개변수를 사용한다면 기본 타입이 제일 마지막에 위치해야 합니다.<br />
( 함수 매개변수의 기본 타입과 동일 )<br />

또한 `(1)`처럼 기본 값이 있다면 일부분의 타입만 명시적으로 작성해도 됩니다.<br />

```ts
// Error: Required type parameters may not follow optional type parameters
interface Person1<T = string, U> {
  name: T;
  age: U;
}

interface Person2<T, U = number> {
  name: T;
  age: U;
}

// (1)
const person: Person2<string> = {
  name: "",
  age: 26,
};
```

# 🔩 제한된 제네릭 타입
`extends` 키워드를 제네릭에 사용하면 입력 받을 수 있는 타입을 제한할 수 있습니다.<br />

아래 예시는 `string`이 포함하는 타입으로 제한을 걸었습니다.<br />

```ts
interface Person<T extends string> {
  name: T;
}

const person1: Person<string> = {
  name: "",
};
const person2: Person<"Aatrox"> = {
  name: "Aatrox",
};
```

## 0️⃣ keyof와 제한된 타입 매개변수
> [`keyof`](/posts/Learning-TypeScript-9장/#0%EF%B8%8F⃣-keyof){:target="_blank"}에 대해서는 링크를 참고해주세요!<br />
{:.prompt-tip}

`keyof`와 같이 사용하면 타입을 더 정확하게 추론해주는 제네릭을 사용할 수 있습니다.<br />

```ts
const getValue = <T, K extends keyof T>(obj: T, k: K) => obj[k];

const person = {
  name: "Aatrox",
  age: 26,
};

// Error: Argument of type '"name1"' is not assignable to parameter of type '"name" | "age"'.
getValue(person, "name1");

// name1: string
const name1 = getValue(person, "name");
// age1: number
const age1 = getValue(person, "age");
```

만약 제네릭을 사용하지 않는다면 객체가 갖는 모든 값들의 타입의 유니언으로 추론됩니다.<br />

```ts
const getValue = <T>(obj: T, k: keyof T) => obj[k];

const person = {
  name: "Aatrox",
  age: 26,
};

// Error: Argument of type '"name1"' is not assignable to parameter of type '"name" | "age"'.
getValue(person, "name1");

// name1: string | number
const name1 = getValue(person, "name");
// age1: string | number
const age1 = getValue(person, "age");
```

# 🎚️ Promise
`Promise`는 제네릭으로 명시해주지 않으면 `unknown`으로 추론하게 됩니다.<br />
하지만 에러에 대한 타입은 무조건 `unknown`입니다.<br />
( 사용할 때 타입을 좁혀서 사용해야 합니다. )<br />

## 0️⃣ Promise 생성
반환되는 값의 타입을 추론해서 결정하는 것이 아니기 때문에 `(1)`처럼 직접적으로 작성해야 합니다.<br />
`fetch()`같은 비동기 요청을 통한 응답 타입을 잘못 작성해도 타입 체커가 인지하지 못하기 때문에 주의해야 합니다.<br />

```ts
(async () => {
  // result1: number
  const result1 = await new Promise((resolve) => resolve(10));

  // (1) result2: string
  const result2 = await new Promise<string>((resolve) => resolve("10"));

  // Error: Argument of type 'number' is not assignable to parameter of type 'string | PromiseLike<string>'
  const result3 = await new Promise<string>((resolve) => resolve(10));
})();
```

## 1️⃣ async 함수
`async`를 사용한 함수는 모두 반환 값이 `Promise`가 되게 됩니다.<br />

반환 값을 `Promise<>` 형태로 감싸주기 때문에 명시적으로 작성하지 않아도 타입을 알아서 추론해줍니다.<br />

```ts
const func1 = async () => 1;
const func2 = () => Promise.resolve(1);

(async () => {
  // v1: number
  const v1 = await func1();

  // v2: number
  const v2 = await func1();
})();
```

비동기 함수의 경우에는 명시적으로 `async`를 적는 것이 더 좋습니다. ( [참고](/posts/이펙티브-타입스크립트-3장/#-item-25--비동기-코드에는-콜백-대신-async-함수-사용하기-){:target="_blank"} )<br />
잘못하면 동기와 비동기가 합쳐진 함수가 만들어질 수 있고, 함수의 시그니처만 보고 함수의 반환 타입을 확실하게 추론할 수 있기 때문입니다.<br />

```ts
// fetch나 axios라고 가정
const getUser = async (id: number) => ({ id, name: "Aatrox", age: 26 });

const func = (id: number) => {
  if (id < 10) return;

  return getUser(id);
};

// Error: Object is possibly 'undefined'.
func(1).then(() => {}).catch(console.error);
```

# 🎲 제네릭 올바르게 사용하기
## 0️⃣ 제네릭 황금률
제네릭은 반드시 필요한 곳에 사용하는 것이 좋습니다.<br />

`(1)`처럼 사용하지 않아도 되는 부분 혹은 반복된 로직이 없는 곳에 사용하는 것보다는 `(2)`처럼 그냥 사용하지 않는 것이 좋습니다.<br />

```ts
// (1)
const logInput = <T extends string>(value: T) => console.log(value);

// (2)
const logInput = (value: string) => console.log(value);
```

## 1️⃣ 제네릭 명명 규칙
제네릭을 사용하는 경우 이름을 명시적으로 작성하는 것이 좋습니다.<br />
`T`, `U`같은 형태를 주로 사용하지만 굳이 한 글자로 작성하지 않아도 됩니다.<br />

```ts
const logInput = <TEXT extends string>(value: TEXT): TEXT => value;
```

# 📮 레퍼런스
1. « 러닝 타입스크립트 10장 » ( 조시 골드버그 지음, 고승원 옮김, 한빛미디어, 2023 )

1. [1-blue - keyof](/posts/Learning-TypeScript-9장/#0%EF%B8%8F⃣-keyof){:target="_blank"}
1. [1-blue - typescript와 async](/posts/이펙티브-타입스크립트-3장/#-item-25--비동기-코드에는-콜백-대신-async-함수-사용하기-){:target="_blank"}