---
layout: post
title: 러닝 타입스크립트 8장 ( 클래스 )
author: admin
date: 2023-03-17 11:31:00 +900
lastmod: 2023-03-17 11:31:00 +900
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

> 해당 포스트는 `러닝 타입스크립트` 8장을 읽고 정리한 포스트입니다.<br />책의 모든 내용을 작성하는 것이 아닌 주관적인 기준에 따라 필요한 정보만 정리했습니다.<br />
{: .prompt-info}

> 클래스의 키워드 작성 순서는 접근성 / `static` / `readonly` 순서로 작성해야 합니다.<br />( `protected static readonly age: number = 20;` )
{:.prompt-tip}

# 🔱 클래스 메서드
클래스의 생성자는 클래스의 메서드처럼 취급됩니다.<br />
그리고 메서드는 일반 함수와 마찬가지로 매개변수와 반환 값에 대한 타입 체크를 수행합니다.<br />
( `noImplicitAny`를 해제한 경우에는 매개변수의 타입을 지정하지 않으면 `any`가 됩니다. )<br />

```ts
class Person {
  constructor(name: string) {}

  say(message: string){}
}

const person = new Person("");
person.say("");
```

# 💸 클래스 속성
클래스의 속성을 읽거나 쓰려면 클래스의 최상단에 명시적으로 선언해야 합니다.<br />
하지만 타입 애너테이션은 선택적으로 사용해도 됩니다.<br />
( 사용하지 않으면 생성자에서 할당되는 초깃값을 기준으로 결정되는 것 같습니다. )<br />

`(1)`과 같이 정의하지 않은 멤버에 접근하려고 하면 오류가 발생합니다.<br />

```ts
class Person {
  name;
  // name: string;

  constructor(name: string) {
    this.name = name;
  }

  say(message: string){}
}

const person = new Person("");
person.say("");

// (1) Error: Property 'age' does not exist on type 'Person'
person.age;
```

## 0️⃣ 함수 속성
+ 메서드: 클래스의 프로토타입으로 함수를 할당하는 방식<br />
+ 속성으로 선언한 메서드: 클래스의 속성으로 함수를 할당하는 방식<br />

메서드는 인스턴스끼리 공유하는 함수가 되고, 속성으로 선언한 메서드는 각 인스턴스가 갖는 독립적인 함수가 됩니다.<br />
( `this`로 각자의 멤버에 접근할 수 있기 때문에 굳이 속성으로 선언할 필요는 없다고 생각합니다. )<br />

```ts
class Person {
  move;

  constructor () {
    // 속성
    this.move = () => {};
  }

  // 메서드
  say() {}
}

const person1 = new Person();
const person2 = new Person();

console.log(person1.move === person2.move); // false
console.log(person1.say === person2.say); // true
```

## 1️⃣ 초기화 검사
`strictNullChecks`가 켜져있다면 아래와 같이 초기화를 하지 않은 경우 타입 에러를 보여줍니다.<br />

```ts
class Person {
  // Error: Property 'name' has no initializer and is not definitely assigned in the constructor
  name: string;

  // constructor () {}
}
```

만약 `strictNullChecks`를 켜두지 않았다면 아래와 같이 타입 체커에서는 통과하지만 런타임에 문제가 되는 동작을 허용할 수 있습니다.<br />
따라서 `strictNullChecks`는 켜두는 것이 좋습니다.<br />

```ts
class Person {
  name: string;

  // constructor () {}
}

// 문제 없이 동작 ( 하지만 실제로는 "name"이 존재하지 않음 )
new Person().name.length;
```

### 1. 확실하게 할당된 속성
만약 `strictNullChecks`를 켜뒀지만, 초기화 검사를 하고 싶지 않은 경우 `Not-null assertion`(`!`)을 사용하면 됩니다.<br />

```ts
class Person {
  name!: string;

  // constructor () {}
}
```

## 2️⃣ 선택적 속성
클래스는 선언된 속성 이름 뒤에 `?`를 사용해서 옵셔널로 선언합니다.<br />

```ts
class Person {
  name?: string;

  // constructor () {}
}

const person = new Person();

// myName: string | undefined
const { name: myName } = person;
```

## 3️⃣ 읽기 전용 속성
인터페이스처럼 클래스에서도 `readonly`를 추가해 속성을 읽기 전용으로 선언할 수 있습니다.<br />
`readonly`를 사용해서 선언한 멤버는 선언된 위치(`(1)`) 혹은 생성자(`(2)`)에서만 값을 할당할 수 있습니다.<br />

( `readonly`는 `TypeScript` 전용 문법이기 때문에 컴파일 후에 사라집니다. )<br />
( 혹시 보호의 목적이라면 `private`, `getter`를 사용하는 것이 맞습니다.  )<br />

```ts
class Person {
  // (1)
  readonly name: string = "Aatrox";

  constructor () {
    // (2)
    this.name = "Puppy";
  }
}

const person = new Person();
```

원시 타입을 갖는 `readonly`의 초깃값은 타입 추론이 구체적으로 됩니다.<br />
`let`과 `const`에서 추론하듯이 리터럴 타입으로 추론됩니다.<br />

```ts
class Person {
  name1 = "Aatrox";
  readonly name2 = "Aatrox";
  readonly name3: string = "Aatrox";

  constructor() {
    // 정상 동작
    this.name1 = "Puppy";

    // Error: Type '"Puppy"' is not assignable to type '"Aatrox"'
    this.name2 = "Puppy";

    // 정상 동작
    this.name3 = "Puppy";
  }
}

// person.name1: string
// person.name2: "Aatrox"
// person.name3: string
const person = new Person();
```

# 🧧 타입으로서의 클래스
클래스는 `JavaScript`의 값으로 사용되지만, `TypeScript`의 타입으로 사용할 수 있습니다.<br />

흥미로운 점은 해당 클래스의 인스턴스가 아니더라도 모든 멤버와 메서드를 갖고 있다면 할당할 수 있습니다.<br />

```ts
class Person {
  name = "Aatrox";

  constructor() {}

  say() {}
}

// 타입으로 사용한 클래스
const person: Person = {
  // 인스턴스가 아니더라도 "Person"에 정의된 멤버와 메서드를 갖고 있는 객체라면 할당 가능
  name: "",
  say() {},
};
```

# 💎 클래스와 인터페이스
`implements`를 사용하면 해당 클래스가 해당 인터페이스를 준수한다고 선언할 수 있습니다.<br />

```ts
interface Person {
  name: string;
  say(): void;
}

// Class 'Student' incorrectly implements interface 'Person'.
// Property 'say' is missing in type 'Student' but required in type 'Person'.
class Student implements Person {
  name = "Aatrox";

  constructor() {}

  // say() {}
}
```

해당 인터페이스를 준수하지 않으면 타입 에러가 발생하게 됩니다.<br />

```ts
interface Person {
  name: string;
  say(message: string): void;
}

class Student implements Person {
  name = "Aatrox";

  constructor() {}

  /**
   * Property 'say' in type 'Student' is not assignable to the same property in base type 'Person'.
   * Type '(message: number) => void' is not assignable to type '(message: string) => void'.
   *   Types of parameters 'message' and 'message' are incompatible.
   *     Type 'string' is not assignable to type 'number'.
   */
  say(message: number) {}
}
```

## 0️⃣ 다중 인터페이스 구현
`implements` 뒤에 `,`를 이용해서 개수 제한 없이 인터페이스를 등록할 수 있습니다.<br />
해당 클래스는 등록된 모든 인터페이스를 준수하도록 정의되어야 합니다.<br />

```ts
interface Person1 {
  name: string;
}
interface Person2 {
  age: number;
}

class Student implements Person1, Person2 {
  name = "Aatrox";
  age = 26;

  constructor() {}
}
```

만약 인터페이스끼리 중복된 부분이 있고 그 타입이 겹칠 수 없다면 해당 클래스는 정의할 수 없습니다.<br />

```ts
interface Person1 {
  name: string;
}
interface Person2 {
  name: string | number;
  age: number;
}

class Student implements Person1, Person2 {
  name = "Aatrox";

  // Property 'name' in type 'Student' is not assignable to the same property in base type 'Person1'.
  // Type 'number' is not assignable to type 'string'.
  // name = 0;
  
  age = 26;

  constructor() {}
}
```

# ⛳ 클래스 확장
`extends`를 이용하면 클래스의 관계가 생기고 확장할 수 있습니다.<br />

```ts
class Person {
  name: string;

  constructor(name: string) {
    this.name = name;
  }
}

class Student extends Person {
  age: number;

  constructor(name: string, age: number) {
    super(name);
    this.age = age;
  }
}
```

## 0️⃣ 할당 가능성 확장
하위 클래스에서 기보 클래스의 멤버나 메서드를 사용할 수 있습니다.<br />

구조적 타이핑을 따르기 때문에 기보 클래스의 타입에 하위 클래스의 인스턴스를 할당할 수 있습니다.<br />
`(1)`에서는 실제로 값은 들어가 있지만 타입 체커는 값이 없다고 판단하기 때문에 타입 에러가 발생합니다.<br />

```ts
class Person {
  name: string;

  constructor(name: string) {
    this.name = name;
  }
}

class Student extends Person {
  age: number;

  constructor(name: string, age: number) {
    super(name);
    this.age = age;
  }
}

const student = new Student("Aatrox", 26);
const person: Person = new Student("Aatrox", 26);

student.name;
// (1) Error: Property 'age' does not exist on type 'Person'.
person.age;
```

## 1️⃣ 재정의된 생성자
하위 클래스의 생성자에서는 반드시 최상위에서 기본 클래스의 생성자를 호출(`(2)`)해야 합니다.<br />

기본 클래스나 하위 클래스가 아무것도 받지 않으면 생략해도 됩니다.<br />
( 생성자를 생략하면 기본적인 형태의 생성자가 들어간 것처럼 동작합니다. )<br />

```ts
class Person {
  name: string;

  constructor(name: string) {
    this.name = name;
  }
}

class Student extends Person {
  age: number;

  constructor(name: string, age: number) {
    // (2)
    super(name);
    this.age = age;
  }
}
```

## 2️⃣ 재정의된 메서드 ( Overriding )
하위 클래스에서는 기본 클래스의 메서드를 재정의할 수 있습니다.<br />

하위 클래스에서 재정의한 메서드는 기본 클래스에서도 사용할 수 있어야 하기 때문에 매개변수와 반환 타입이 같거나 더 구체적이어야 합니다.<br />
`(3)`의 경우는 `volume`이 추가되긴 했지만, 옵셔널이기 때문에 기본 클래스에서도 문제 없이 사용할 수 있습니다.<br />

```ts
class Person {
  name: string;

  constructor(name: string) {
    this.name = name;
  }

  say(message: string) {
    console.log(message)
  }
}

class Student extends Person {
  age: number;

  constructor(name: string, age: number) {
    super(name);
    this.age = age;
  }

  // (3)
  say(message: string, volume?: number) {
    console.log(message)
  }
}
```

## 3️⃣ 재정의된 속성
속성도 마찬가지로 기본 클래스에서도 사용할 수 있어야 하기 때문에 구조적으로 일치해야 합니다.<br />

`(4)`처럼 다른 타입을 지정하면 타입 에러가 발생하게 됩니다.<br />
또한 하위 클래스에서 재정의하는 것이기 때문에 하위 클래스에서 초기화해줘야 합니다.<br />

```ts
class Person {
  name?: string;

  constructor(name?: string) {
    this.name = name;
  }
}

class Student extends Person {
  name: string;

  // (4) Property 'name' in type 'Student' is not assignable to the same property in base type 'Person'.
  // Type 'number' is not assignable to type 'string'
  // name: number;

  constructor(name: string) {
    super();
    this.name = name;
  }
}
```

# 🤿 추상 클래스
클래스, 메서드, 멤버를 직접 구현하고 싶지 않고 형태만 정의하고 싶을 수 있습니다.<br />
그런 경우에 추상화된 기본 클래스만 만드는 목적으로 사용하면 좋은 기능입니다.<br />

추상 클래스로 인스턴스를 만들 수 없고, 추상 메서드와 추상 멤버는 반드시 하위 클래스에서 구현해야 합니다.<br />

```ts
// 추상 클래스
abstract class Person {
  // 추상 멤버
  abstract name: string;
  // 추상 메서드
  abstract say(): void;

  // constructor() {};
}

class Student extends Person {
  name: string;

  constructor(name: string) {
    super();
    this.name = name;
  }

  say() {}
}

// Error: Cannot create an instance of an abstract class
const person = new Person("Aatrox");

const student = new Student("Aatrox");
```

# 🥯 멤버 접근성
`TypeScript`에서는 접근 제한자가 존재합니다.<br />
`JavaScript`에서도 `#`이라는 `private`한 접근 제한자가 존재하는데 `private`와 `#`은 같은 역할을 하지만 다릅니다.<br />

`#`은 컴파일 후에도 사라지지 않는 `JavaScript` 고유의 문법이고, `private`은 컴파일 후에 사라지는 `TypeScript` 고유의 문법입니다.<br />

+ 접근 제한자
  1. `public`: 어디서나 접근 가능
  1. `protected`: 클래스 내부 또는 하위 클래스에서만 접근 가능
  1. `private`: 클래스 내부에서만 접근 가능

```ts
abstract class Person {
  public name = "Aatrox";
  protected age = 26;
  private gender = true;

  constructor(name: string, age: number, gender: boolean) {
    this.name = name;
    this.age = age;
    this.gender = gender;
  };

  abstract say(): void
}

class Student extends Person {
  constructor(name: string, age: number, gender: boolean) {
    super(name, age, gender);
  }

  say(){
    console.log(this.name);
    console.log(this.age);
    // // Error: Property 'gender' is private and only accessible within class 'Person'.
    // console.log(this.gender);
  }
}

const student = new Student("Puppy", 13, false);

student.name;
// Error: Property 'age' is protected and only accessible within class 'Person' and its subclasses
student.age;
// Error: Property 'gender' is private and only accessible within class 'Person'
student.gender;

student.say(); // "Puppy" 13
```

## 0️⃣ 정적 필드 제한자
클래스 자체의 멤버로 선언하기 위해 사용하는 키워드입니다.<br />

```ts
class Person {
  public static age = 26;
  private static gender = true;

  public static say() {
    console.log(Person.age, Person.gender);
  }
}

const person = new Person();

// Error: Property 'age' does not exist on type 'Person'.
person.age;

Person.age;
Person.age = 20;

// Error: Property 'gender' is private and only accessible within class 'Person'.
Person.gender;

Person.say(); // 20, true
```

# 📮 레퍼런스
1. « 러닝 타입스크립트 8장 » ( 조시 골드버그 지음, 고승원 옮김, 한빛미디어, 2023 )