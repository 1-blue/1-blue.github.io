---
layout: post
title: 러닝 타입스크립트 11장 ( 선언 파일 )
author: admin
date: 2023-04-05 14:05:00 +900
lastmod: 2023-04-05 14:05:00 +900
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

> 해당 포스트는 `러닝 타입스크립트` 11장을 읽고 정리한 포스트입니다.<br />책의 모든 내용을 작성하는 것이 아닌 주관적인 기준에 따라 필요한 정보만 정리했습니다.<br />
{: .prompt-info}

# 🧤 선언 파일
> 선언 파일은 **앰비언트 컨텍스트**라는 코드 영역에 생성됩니다.<br />
{:.prompt-tip}

`.d.ts` 파일은 런타임 코드를 포함할 수 없는 선언 파일입니다.<br />

```ts
// person.d.ts

export interface Person {
  name: string;
}
// Error: .d.ts 파일의 최상위 수준 선언은 'declare' 또는 'export' 한정자로 시작해야 합니다
const person: Person = {
  name: string,
};
```

# 🎩 런타임 값 선언
`declare` 키워드를 이용하면 값이 선언되어 있다고 가정할 수 있습니다.<br />

실존하는 값이 아니기 때문에 값을 부여하면 안됩니다.<br />

```ts
interface Person {
  name: string;
}

// Error: 'string' only refers to a type, but is being used as a value here.
declare const person: Person = {
  name: string,
};
```

## 0️⃣ 전역 변수
스크립트인 파일에서 선언한 타입들은 다른 파일에서 전역 변수처럼 사용할 수 있습니다.<br />
( `a.d.ts`, `index.d.ts`처럼 파일명을 작성하면 오류가 발생합니다... ( 원인 모름 😥 ) )<br />

```ts
// aa.d.ts
declare const def: number;

// *.ts
def; // def: number;
```

## 1️⃣ 전역 인터페이스 병합
`interface`는 같은 이름으로 선언하면 자동으로 확장되게 됩니다.<br />
이 기능을 이용해서 스크립트로 인터페이스를 선언하면 `Window`같은 전역 객체를 확장할 수 있습니다.<br />

```ts
// aa.d.ts
interface Window {
  myValue: string;
}

// index.ts
// window.myValue: string
window.myValue;
```

## 2️⃣ 전역 확장
모듈인 파일이라도 `declare global` 코드 블록 구문을 사용하면 전역 컨텍스트에 정의할 수 있습니다.<br />

```ts
// aa.d.ts
declare const def: number;

declare global {
  declare const abc: string;
}

export {};

// index.ts
// Error: 'def' 이름을 찾을 수 없습니다
def;

// abc: string
abc;
```

# 💰 내장된 선언
## 0️⃣ 라이브러리 선언
내장된 전역 객체는 `lib.[target].d.ts`로 선언됩니다.<br />
각 `target`에 해당하는 타입이 각 선언 파일에 정의되어 있고 `ts.config.json`의 `lib`에 의해서 사용할 선언 파일을 결정합니다.<br />

## 1️⃣ DOM 선언
`HTMLElement`, `localStorage`의 타입 같은 경우에는 `lib.dom.d.ts`, `lib.*.d.ts`과 같은 선언 파일에 저장됩니다.<br />
기본적으로 `DOM`에 대한 타입을 포함되기 때문에 `node.js` 환경에서 개발한다면 `ts.config.json`의 `lib`를 수정해야 합니다.<br />

# 📇 모듈 선언
모듈의 내용을 타입 시스템에게 알리는 방법입니다.<br />

```ts
// "aa"라는 모듈에 대한 타입 선언
declare module "aa" {
  export const value: string;
}

// "aa"라는 모듈이 실제로 정의되어있지 않아서 오류가 나지만 타입은 존재함
import { aa } from "aa";

aa;
```

## 0️⃣ 와일드카드 모듈 선언
> 와일드카드는 타입의 안정성을 보장해주지 않기 때문에 주의해서 사용해야 합니다.<br />
{:.prompt-tip}

`react`의 `css` 모듈을 사용하는 경우 타입 시스템에게 아래와 같이 타입을 알려줄 수 있습니다.<br />

```ts
// styles.d.ts
declare module "*.module.css" {
  const styles: { [key: string]: string };
  export default styles;
}
```

# 🏐 패키지 타입
TODO:

## 0️⃣ 선언
TODO:

## 1️⃣ 패키지 타입 의존성
TODO:

## 2️⃣ 패키지 타입 노출
TODO:

# 🧩 Definitely Typed
`DT`에는 `.d.ts` 정의 패키지들이 포함되어 있습니다.<br />
( `@type/*` )<br />

## 0️⃣ 타입 사용 가능성
타입이 정의되지 않은 패키지인 경우 세 가지 방법으로 해결할 수 있습니다.<br />

1. `@types/`를 생성하기 위한 `PR` 보내기
2. `declare module`을 사용해 프로젝트 내에서 타입을 작성
3. `noInplicitAny` 옵션 비활성화

# 📮 레퍼런스
1. « 러닝 타입스크립트 11장 » ( 조시 골드버그 지음, 고승원 옮김, 한빛미디어, 2023 )