---
layout: post
title: ESLint & Prettier
author: admin
date: 2023-02-25 16:49:00 +900
lastmod: 2023-02-25 16:49:00 +900
sitemap:
  changefreq: monthly
  priority: 0.5
categories: [Common, ESLint/Prettier]
tags: [ESLint, Prettier]
---

> 해당 포스트는 `Eslint`와 `Prettier`에 대한 포스트입니다.<br />
{: .prompt-info}

# 🧱 ESLint
`Lint`란 소스 코드에 문법적인 문제가 있는지 탐색하는 도구입니다.<br />
그 중에서 `ESLint`라는 도구를 사용해서 런타임 전에 미리 코드의 문제를 확인할 수 있습니다.<br />

## 0️⃣ 설치

```bash
npm i -D eslint

# extends ( 예시용 )
npm i -D eslint-config-airbnb

# plugin ( 예시용 )
npm i -D eslint-plugin-import
```

+ [`eslint-config-airbnb`](https://github.com/airbnb/javascript){:target="_blank"}: `airbnb`의 코드 스타일 가이드
+ [`eslint-plugin-import`](https://github.com/import-js/eslint-plugin-import){:target="_blank"}: [`ES6` 모듈 시스템](/posts/자바스크립트-완벽-가이드-10장/#-es6의-모듈-시스템){:target="_blank"} 구문 규칙 지원 플러그인

## 1️⃣ ESLint 설정 파일
`.eslintrc.js`말고 `.eslintrc.json`으로 생성하면 자동완성을 지원해줍니다.<br />

+ `.eslintrc.json`

```json
{
  "env": {
    "browser": true,
    "es2021": true,
    "node": true
  },
  "overrides": [],
  "parserOptions": {
    "ecmaVersion": "latest",
    "sourceType": "module"
  },
  "plugins": ["import"],
  "extends": ["airbnb", "plugin:import/recommended"],
  "rules": {}
}
```

+ `airbnb`: `airbnb`의 규칙 적용
+ `plugin:import/recommended`: `ES6` 모듈 시스템 규칙 적용


### 1. env
코드를 실행하는 환경마다 갖고 있는 전역변수들이 다릅니다.<br />
특정 환경에 맞는 전역변수들이나 환경에 대한 린팅 문제를 해결하기 위한 설정 값입니다.<br />

```json
{
  "env": {
    "browser": true,
    "es2021": true,
    "node": true
  },
  // ... 나머지 생략
}
```

### 2. plugin
누군가에 의해 이미 만들어진 `rule`을 가져오는 옵션입니다.<br />
가져온다고 적용되는 것은 아니고 그냥 설정 가능한 상태로 만드는 옵션입니다.<br />
`extends`에서 추가 설정을 해줘야 적용됩니다.<br />

```json
{
  "plugins": ["import"],
  // ... 나머지 생략
}
```

### 3. extends
이미 만들어진 린팅 규칙 혹은 `plugin`으로 등록한 규칙을 적용하는 옵션입니다.<br />
`airbnb`, `facebook`같은 곳에서 만들어둔 린팅 규칙을 바로 적용할 수 있습니다.<br />
( `eslint-config-airbnb-base` 등 )<br />
( `eslint-config-` 생략 가능합니다. )<br />

```json
{
  "extends": ["airbnb", "plugin:import/recommended"],
  // ... 나머지 생략
}
```

### 4. rules
> 자세한 규칙들은 [eslint - rules](https://eslint.org/docs/latest/rules/){:target="_blank"}에서 확인할 수 있습니다.<br />
{:.prompt-tip}

`extends`로 설정된 `rule`을 커스터마이징하는 옵션입니다.<br />
즉, 기존 규칙에 필요한 부분만 커스터마이징하는 옵션입니다. ( 덮어쓰기 )<br />

```json
{
  "rules": {
    // 불필요한 세미콜론 사용 시 에러
    "no-extra-semi": "error"
  },
  // ... 나머지 생략
}
```

# 🖌️ Prettier
코드 스타일을 관리해주는 도구입니다.<br />
줄바꿈, 들여쓰기, 쌍따옴표, 화살표 함수등의 문법적으로 문제는 없지만 사람마다 다르게 사용하는 부분을 관리해주는 도구입니다.<br />

## 0️⃣ Prettier 설정 파일
> 자세한 `prettier` 설정은 [Prettier - Options](https://prettier.io/docs/en/options.html){:target="_blank"}를 참고해주세요.<br />
{:.prompt-tip}

+ `.prettierrc.json`

```json
{
  "singleQuote": false,
  "tabWidth": 2,
  // ...
}
```

> 만약 `.prettierrc.json`와 `VSCode setting.json`에 `prettier`가 세팅되어 있다면 `.prettierrc.json`가 우선적으로 적용됩니다.<br />
{:.prompt-info}

# 🎨 ESLint + Prettier
`ESLint`와 `Prettier`의 역할을 다르지만 중복으로 처리하는 부분이 있을 수 있습니다.<br />
그런 충돌을 해결하기 위해서 아래와 같이 세팅해야합니다.<br />

## 0️⃣ 설치
```bash
# eslint와 prettier의 호환성을 위함
npm i -D eslint-config-prettier eslint-plugin-prettier
```
+ [`eslint-config-prettier`](https://github.com/prettier/eslint-config-prettier){:target="_blank"}: `eslint`와 `prettier`가 중복되는 규칙이 있다면 `eslint`를 적용
+ [`eslint-plugin-prettier`](https://github.com/prettier/eslint-plugin-prettier){:target="_blank"}: `eslint`의 규칙으로 `prettier`를 실행하고 `eslint`에서 문제 발생시킴

## 1️⃣ 설정 파일

+ `.eslintrc.json`

```json
{
  "env": {
    "browser": true,
    "es2021": true,
    "node": true
  },
  "overrides": [],
  "parserOptions": {
    "ecmaVersion": "latest",
    "sourceType": "module"
  },
  "plugins": ["import", "prettier"],
  "extends": ["airbnb", "plugin:import/recommended", "plugin:prettier/recommended"],
  "rules": {
    "prettier/prettier": "error"
  }
}
```

+ `plugin:prettier/recommended`: `eslint-plugin-prettier`의 설정 적용

# 마무리
여러 문서와 포스트를 찾아보면서 설정을 했는데 이게 제대로 동작하는 건지 아닌 건지 확신이 안 들어서 제대로 정리한 것인지도 잘 모르겠네요... 🥲<br />

일단 얼마후에 팀 프로젝트를 할 예정이라 미리 기본 세팅 방법들에 대해 공부해보고 있어서 아직 적용을 해보지 못한 상태입니다.<br />
나중에 프로젝트를 진행하고 세팅에 문제가 있었거나 새롭게 이해한 부분이 생긴다면 다시 수정하겠습니다.<br />

# 📮 레퍼런스
1. [helloinyong - (ESLint & Prettier) Setting](https://helloinyong.tistory.com/325){:target="_blank"}
2. [daleseo - eslint 사용](https://www.daleseo.com/js-eslint/){:target="_blank"}
3. [daleseo - eslint 상세 설정 가이드](https://www.daleseo.com/eslint-config/){:target="_blank"}

1. [GitHub - `eslint-config-airbnb`](https://github.com/airbnb/javascript){:target="_blank"}
1. [GitHub - `eslint-plugin-import`](https://github.com/import-js/eslint-plugin-import){:target="_blank"}
1. [1-blue - `ES6` 모듈 시스템](/posts/자바스크립트-완벽-가이드-10장/#-es6의-모듈-시스템){:target="_blank"}
1. [GitHub - `eslint-config-prettier`](https://github.com/prettier/eslint-config-prettier){:target="_blank"}
1. [GitHub - `eslint-plugin-prettier`](https://github.com/prettier/eslint-plugin-prettier){:target="_blank"}