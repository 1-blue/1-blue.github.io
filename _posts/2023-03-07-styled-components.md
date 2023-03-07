---
layout: post
title: styled-components + TypeScript 세팅 및 사용 예시
author: admin
date: 2023-03-07 23:53:00 +900
lastmod: 2023-03-07 23:53:00 +900
sitemap:
  changefreq: monthly
  priority: 0.5
categories: [FrontEnd, CSS]
tags: [styled-components, CSS In JS]
---

> 해당 포스트는 `styled-components` + `TypeScript`의 사용법을 정리하는 포스트입니다.<br />
{: .prompt-info }

# 🫠 Install
해당 포스트는 `TypeScript` 기준으로 작성했기 때문에 타입 정의도 설치해야 합니다.<br />

```bash
# "styled-components" 설치
npm i styled-components

# "styled-components" 타입 정의 설치
npm i -D @types/styled-components

# 제공해주는 "css reset"을 사용한다면 설치
npm i styled-reset
```

# 🦴 Global
`styled-components`에 대한 폴더 구조만 작성했습니다.<br />
( 추가로 아래의 예시에서 사용하는 경로에 `@src`는 절대 경로로 `src` 폴더를 의미합니다. )<br />

```
  ├── node_modules
  ├── public
  ├── src
  │   ├── components
  │   │   └── common
  │   │       └── Box
  │   │           │── index.tsx
  │   │           └── style.tsx
  │   ├── shared
  │   │   │── global.ts
  │   │   └── theme.ts
  │   └── index.tsx
  ├── tsconfig.json
  ├── package-lock.json
  └── package.json
```

## 0️⃣ Global Style
전역적인 스타일을 적용하는 방법입니다.<br />
`css reset`과 전역적으로 커스터마이징할 스타일에 대해 작성합니다.<br />

+ `@src/shared/global.ts`

```ts
import { createGlobalStyle } from "styled-components";
import reset from "styled-reset";

export const GlobalStyle = createGlobalStyle`
  ${reset}

  /* 아래에 추가적으로 적용할 전역 스타일 작성 */
`;
```

## 1️⃣ Theme
`styled-components`에서 전역적으로 사용할 변수를 지정하는 방법입니다.<br />

+ `@src/shared/theme.ts`

```ts
import { css } from "styled-components";

/** 자주 사용하는 색상들 */
const colors = {
  slate50: "#f8fafc",
  slate100: "#f1f5f9",
  slate200: "#e2e8f0",
  slate300: "#cbd5e1",
  slate400: "#94a3b8",
  slate500: "#64748b",
  slate600: "#475569",
  slate700: "#334155",
  slate800: "#1e293b",
  slate900: "#0f172a",
  /* 나머지 색상들 생략 ( https://tailwindcss.com/docs/customizing-colors ) */

  main50: "#eef2ff",
  main100: "#e0e7ff",
  main200: "#c7d2fe",
  main300: "#a5b4fc",
  main400: "#818cf8",
  main500: "#6366f1",
  main600: "#4f46e5",
  main700: "#4338ca",
  main800: "#3730a3",
  main900: "#312e81",

  /* 아래 부분을 비워둔 이유는 타입때문 ( "<ThemeProvider>"에서 조건에 따라 다르게 값을 채움 ) */
  color: "",
  bgColor: "",
  gray: "",
};

/** 검정 배경 */
export const darkTheme = {
  color: "#000000",
  bgColor: "#FFFFFF",
  gray: "#343434",
};
/** 흰색 배경 */
export const lightTheme = {
  color: "#FFFFFF",
  bgColor: "#000000",
  gray: "#D9D9D9",
};

/** 반응형 사이즈 */
const mediaSize = {
  xs: "screen and (max-width: '400px')",
  sm: "screen and (max-width: '640px')",
  md: "screen and (max-width: '768px')",
  lg: "screen and (max-width: '1024px')",
  xl: "screen and (max-width: '1280px')",
  "2xl": "screen and (max-width: '1536px')",
};

/** 폰트 크기 */
const fontSize = {
  xs: "0.75rem",
  sm: "0.875rem",
  md: "1rem",
  lg: "1.125rem",
  xl: "1.25rem",
  "2xl": "1.5rem",
};

/** 그 외의 크기 */
const size = {
  xs: "0.2em",
  sm: "0.4em",
  md: "0.6em",
  lg: "1em",
  xl: "1.4em",
  "2xl": "1.6em",
};

/** 유틸리티 */
const util = {
  truncate: () => css`
    overflow: hidden;
    text-overflow: ellipsis;
    white-space: nowrap;
  `,

  scroll: () => css`
    &::-webkit-scrollbar {
      /** 스크롤바의 너비 */
      width: 4px;
    }
    &::-webkit-scrollbar-thumb {
      /** 스크롤바 길이 */
      height: 25%;
      /** 스크롤바의 색상 */
      background: ${({ theme }) => theme.colors.indigo600};
      border-radius: 10px;
    }
    &::-webkit-scrollbar-track {
      /** 스크롤바 뒷 배경 색상 */
      background: ${({ theme }) => theme.colors.indigo300};
    }
  `,
};

const theme = {
  colors,
  mediaSize,
  fontSize,
  size,
  util,
};

export default theme;

/** 타입 재정의를 위함 ( "styled-components" 변수 타입 추론을 위함( 자동완성 ) ) */
export type Theme = typeof theme;
```

## 2️⃣ 타입 재정의
`styled-components`의 자동완성을 위해서 아래와 같이 타입 정의를 수정해줘야 합니다.<br />

+ `@src/@types/styled.d.ts`

```ts
import type { CSSProp } from "styled-components";
import type { Theme } from "@src/shared/theme";

declare module "styled-components" {
  export interface DefaultTheme extends Theme {
    /*
     * 필요하다면 여기서 타입을 정의해줘도 됨
     * 하지만 "Theme"가 수정될 때마다 수정사항을 반영해줘야 하기 때문에 "extends Theme"형태로 적는 것이 좋음
     */
  }
}
```

## 3️⃣ Global Style과 Theme 적용 방법
`styled-components`를 전역적으로 적용할 것이기 때문에 컴포넌트를 라우팅하는 `<BrowserRouter>`를 감싸주도록 작성하면 됩니다.<br />

`(1)`처럼 `<ThemeProvider theme={theme}>`를 감싸주면 [이전에 만들었던 `theme`](/posts/styled-components/#1%EF%B8%8F⃣-theme)를 전역적으로 사용할 수 있습니다.<br />
하지만 `TypeScript`에서 어떤 타입이 들어갔는지 추론을 못하기 때문에 [타입 재정의](/posts/styled-components/#2%EF%B8%8F⃣-타입-재정의)에서 `theme`에 어떤 타입이 들어가는지 구체적인 정의를 작성했습니다.<br />
따라서 실제로 사용할 때는 타입 체커의 도움을 받을 수 있습니다.<br />

`(2)`를 작성한 이유는 [GlobalStyle](/posts/styled-components/#0%EF%B8%8F⃣-global-style)을 적용하기 위해서입니다.<br />

```tsx
import { createRoot } from "react-dom/client";
import { BrowserRouter, Route, Routes } from "react-router-dom";
import { ThemeProvider } from "styled-components";

// global-setting
import { GlobalStyle } from "@src/shared/global";
import theme from "@src/shared/theme";

// page component
import App from "@src/components/App";

const container = document.getElementById("root");
const root = createRoot(container!);

root.render(
  /* (1) */
  <ThemeProvider theme={theme}>
    {/* (2) */}
    <GlobalStyle />

    <BrowserRouter>
      <Routes>
        <Route path="/" element={<App />} />
      </Routes>
    </BrowserRouter>
  </ThemeProvider>
);
```

# 📚 Component
이후에 설명하는 부분은 `common`의 `Box` 컴포넌트에 해당합니다.<br />
의미있는 컴포넌트는 아니고 단순하게 `props`에 따라서 크기와 색상이 변경되는 컴포넌트입니다.<br />

크기와 색상의 변경을 `styled-components`를 이용합니다.<br />
그리고 개인적으로 목적이 다르다면 파일을 분리하는 것을 좋아해서 컴포넌트와 스타일의 파일을 분리해서 작성했습니다.<br />

```
  ├── node_modules
  ├── public
  ├── src
  │   ├── components
  │   │   └── common
  │   │       └── Box
  │   │           │── index.tsx
  │   │           └── style.tsx
  │   ├── shared
  │   │   │── global.ts
  │   │   └── theme.ts
  │   └── index.tsx
  ├── tsconfig.json
  ├── package-lock.json
  └── package.json
```

## 0️⃣ Component 작성
컴포넌트 부분에 대한 설명입니다.<br />
일반적인 컴포넌트와 같지만 `styled-components`를 가져와서 사용하고 있고, `Props` 타입을 `styled-components`에서 사용하기 위해서 `export` 해서 사용하고 있습니다.<br />

+ `@src/components/common/Box/index.tsx`

```tsx
// style
import StyledBox from "./style";

// type
type Size = "xs" | "sm" | "md" | "lg" | "xl" | "2xl";
export type Props = {
  size?: Size;
  bgColor: string;
};

const Box = ({ size, bgColor }: Props) => {
  return (
    <StyledBox size={size} bgColor={bgColor}>
      <span>아무 텍스트</span>
    </StyledBox>
  );
};

export default Box;
```

## 1️⃣ Styled Component 작성
스타일이 적용된 컴포넌트를 만드는 부분입니다.<br />
`(1)`에서는 컴포넌트에서 정의한 `Props`에서 필요한 것만 `Pick<T, K>`을 이용해서 필터링해서 사용합니다.<br />

`(2)`에서는 이전에 [전역적으로 정의한 `theme`](/posts/styled-components/#1%EF%B8%8F⃣-theme)과 `props`로 받은 인수를 이용해서 스타일을 적용합니다.<br />

+ `@src/components/common/Box/index.tsx`

```tsx
import styled, { css } from "styled-components";

// type
import type { Props } from ".";
// (1) Pick<T, K>
type StyledProps = Pick<Props, "size" | "bgColor">;

// (1) <StyledProps>
const StyledBox = styled.div<StyledProps>`
  display: flex;
  justify-content: center;
  align-items: center;

  background-color: ${({ bgColor }) => bgColor};

  /* (2) theme */
  font-size: ${({ theme, size }) => theme.fontSize[size || "md"]};
  font-weight: bold;

  border-radius: 0.4em;

  ${({ size }) => {
    switch (size) {
      case "xs":
        return css`
          width: 200px;
          height: 200px;
        `;
      case "sm":
        return css`
          width: 240px;
          height: 240px;
        `;
      case "md":
        return css`
          width: 280px;
          height: 280px;
        `;
      case "lg":
        return css`
          width: 320px;
          height: 320px;
        `;
      case "xl":
        return css`
          width: 360px;
          height: 360px;
        `;
      case "2xl":
        return css`
          width: 400px;
          height: 400px;
        `;
    }
  }}
`;

export default StyledBox;
```

# 📮 레퍼런스
1. [`styled-components`](https://styled-components.com/){:target="_blank"}