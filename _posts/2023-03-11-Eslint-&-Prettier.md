---
layout: post
title: Eslint & Prettier ( + TypeScript )
author: admin
date: 2023-03-11 19:26:00 +900
lastmod: 2023-03-11 19:26:00 +900
sitemap:
  changefreq: monthly
  priority: 0.5
categories: [Common, Eslint/Prettier]
tags: [Eslint, Prettier]
---

> 해당 포스트는 [직접 설정한 `Webpack`](/posts/Webpack){:target="_blank"}에 `Eslint`와 `Prettier`를 설정하는 방법을 정리한 포스트입니다.<br />
{: .prompt-info}

# 🚨 Eslint
소스 코드의 문법적인 오류를 찾아내는 도구입니다.<br />

## 0️⃣ 설치
개발 모드로 설치하고 `-g`를 이용해서 글로벌로 설치해도 됩니다.<br />
( 저는 컴퓨터에 최소한으로 설치하는 것을 좋아해서 전역적으로 설치하지는 않았습니다. )<br />

```bash
npm i -D eslint
```

## 1️⃣ 설정 파일 생성
`.eslintrc.json` 혹은 `.eslintrc.js`의 파일이 존재하면 `VSCode`에서 인식하고 해당 설정에 맞게 처리해줍니다.<br />

직접 생성해도 되지만 아래 명령어를 사용하면 여러 설정에 대해 물어보고 파일을 생성하고 패키지를 설치해줍니다.<br />

```bash
npx eslint --init
```

- 응답
  1. 구문확인/문제체크/코드스타일
  1. modules
  1. React
  1. TypeScript
  1. Browser
  1. 나만의 스타일
  1. JavaScript
  1. Spaces
  1. Double
  1. windows
  1. semocolons

위와 같이 응답하면 알아서 `eslint-plugin-react@latest`, `@typescript-eslint/eslint-plugin@latest`, `@typescript-eslint/parser@latest`를 설치해줍니다.<br />
( `React`, `TypeScript`에 적용할 코드 검사 설정에 대한 패키지 )<br />

`eslint`와 `prettier`의 호환성을 위해 아래의 패키지도 설치합니다.<br />

+ [`eslint-config-prettier`](https://github.com/prettier/eslint-config-prettier){:target="_blank"}: `eslint`와 `prettier`가 중복되는 규칙이 있다면 `eslint`를 적용
+ [`eslint-plugin-prettier`](https://github.com/prettier/eslint-plugin-prettier){:target="_blank"}: `eslint`의 규칙으로 `prettier`를 실행하고 `eslint`에서 문제 발생시킴

```bash
npm i -D eslint-config-prettier eslint-plugin-prettier
```

모든 처리를 하고 나면 아래와 유사한 파일이 생성됩니다.<br />
저는 필요에 의해서 몇 가지 설정을 추가해서 조금 다를 수 있습니다.<br />

+ `.eslintrc.js`

```js
/** @type { import("eslint").Linter.Config } */
module.exports = {
  env: {
    browser: true,
    es2021: true,
    node: true,
  },
  extends: [
    "eslint:recommended",
    "plugin:react/recommended",
    "plugin:prettier/recommended",
    "plugin:@typescript-eslint/recommended",
  ],
  overrides: [],
  parser: "@typescript-eslint/parser",
  parserOptions: {
    ecmaVersion: "latest",
    sourceType: "module",
  },
  plugins: ["react", "@typescript-eslint"],
  rules: {
    "prettier/prettier": [
      "error",
      {
        // 윈도우에서 "Delete `␍`" 에러를 발생시켜서 추가
        endOfLine: "auto",
      },
    ],

    // webpack에서 `import React from "react"`처리를 해주기 때문에 검사 끄기
    "react/react-in-jsx-scope": "off",
  },
};
```

# 🖌️ Prettier
소스 코드의 스타일을 관리해주는 도구입니다.<br />

## 0️⃣ 설치

```bash
npm i -D prettier
```

## 1️⃣ 설정 파일 생성
`.prettierrc.json` 혹은 `.prettierrc.js`의 파일이 존재하면 `VSCode`에서 인식하고 해당 설정에 맞게 처리해줍니다.<br />

많은 설정들이 있지만 저의 스타일대로 아래처럼 설정했습니다.<br />
`VSCode` 자체에 `prettier`를 세팅했어도 아래 파일의 세팅이 우선적으로 설정됩니다.<br />

+ `.prettierrc.js`

```js
/** @type { import("prettier").Options } */
module.exports = {
  singleQuote: false,
  semi: true,
  useTabs: false,
  tabWidth: 2,
  trailingComma: "all",
  printWidth: 100,
  parser: "babel",
  bracketSpacing: true,
  arrowParens: "always",
};
```

# ⚙️ VSCode 세팅
아래의 세팅을 추가해주면 기본적으로 `VSCode`에서 파일을 저장할 때 `prettier` 세팅에 의해 코드 스타일을 변경시켜줍니다.<br />
또한 아래에 작성한 파일들 `.js`, `.jsx`, `.ts`, `.tsx`에만 적용됩니다.<br />

그리고 `.prettierrc.*` 같은 파일이 있다면 해당 파일의 설정이 우선적으로 적용됩니다.<br />
만약 파일이 없다면 `VSCode`의 `setting.json`에 등록한 스타일 설정으로 적용됩니다.<br />

+ `setting.json`

```json
{
  // ... 다른 설정들 생략

  "editor.defaultFormatter": "esbenp.prettier-vscode",
  "[javascript]": { "editor.formatOnSave": true },
  "[javascriptreact]": { "editor.formatOnSave": true },
  "[typescript]": { "editor.formatOnSave": true },
  "[typescriptreact]": { "editor.formatOnSave": true }
}
```

# ☄️ 문제
## 0️⃣ 충돌 문제
`Not-Null Assertion`(`!`)이나 타입 단언(`as`)를 사용하면 `prettier`에서 에러를 발생시킵니다.<br />

근데 이게 `TypeScript`에서는 맞는 구문이고 `eslint`에서 `@typescript-eslint/parser`를 통해 처리를 해줬는데 왜 발생하는지 어떻게 해결하는지 전혀 모르겠습니다.<br />
`eslint-config-prettier`를 적용하면 `eslint`와 `prettier`가 충돌나면 `eslint`를 우선적으로 적용하는 것으로 알고 있는데 잘못 알고 있는 건지 모르겠네요... 😥<br />

```tsx
const root = createRoot(container!); // Parsing error: Unexpected token eslint(prettier/prettier)
const str = "str" as string; // Parsing error: Missing semicolon eslint(prettier/prettier)
```

## 1️⃣ 적용 문제
어떤 설정을 적용했는데도 불구하고 계속 적용이 안 되는 경우가 있습니다.<br />
저는 무슨 다른 문제가 있는줄 알고 계속 찾아보고 수정했는데 알고 보니 `VSCode`를 껐다가 키면 적용이 됩니다... 🥲<br />
( 내 잃어버린 2시간... 😥 )<br />

# 📮 레퍼런스
1. [1-blue - 직접 설정한 `Webpack`](/posts/Webpack){:target="_blank"}