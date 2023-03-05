---
layout: post
title: 스토리북 기본 사용법 ( Storybook + webpack )
author: admin
date: 2023-03-04 21:47:00 +900
lastmod: 2023-03-04 21:47:00 +900
sitemap:
  changefreq: monthly
  priority: 0.5
categories: [FrontEnd, Storybook]
tags: [Storybook, 스토리북]
---

> `Storybook`은 컴포넌트만 분리해서 개발하는 도구입니다. ( `CDD` )<br />
{:.prompt-info}

> `Storybook` + `styled-components` 기반으로 작성하겠습니다. ( [결과물](https://www.chromatic.com/library?appId=64033003a278c2f727eaaeac&branch=master){:target="_blank"} )<br />
{:.prompt-info}

# 🫠 설치
빈 프로젝트에서는 시작할 수 없기 때문에 `React` 기준으로 `CRA`나 직접 `Webpack`을 설정한 파일에서 시작해야 합니다.<br />
( `Webpack`을 직접 설정하고 싶다면 [직접 설정한 Webpack](/posts/Webpack/){:target="_blank"}를 참고해주세요. )<br />

```bash
# 스토리북 설치
npx storybook init
npx sb init
```

# 🦴 세팅
위 명령어를 실행하면 `.storybook`이라는 폴더가 생성됩니다.<br />
그리고 폴더 내부에는 `main.js`와 `preview.js` 파일이 있습니다.<br />
각 파일들의 역할과 스토리북을 만드는 방법에 대해 설명하겠습니다.<br />

## 0️⃣ 프로젝트 구성 파일
> [TypeScript + Storybook](https://storybook.js.org/docs/react/configure/overview#configure-your-project-with-typescript){:target="_blank"} / [Webpack + Storybook](https://storybook.js.org/docs/react/builders/webpack){:target="_blank"}
{:.prompt-tip}

기본적으로 `main.js`로 생성되며 수정할 경우 프로세스를 다시 시작해야합니다.<br />
`TypeScript` 기반으로 작성하기 때문에 `main.ts`로 변경하면 됩니다.<br />

저는 아래와 같이 세팅했고 원하는 부분을 원하는 대로 추가 및 수정하면 됩니다.<br />

`npm start`와 같이 기본적으로 세팅된 실행으로 설정한 `Webpack` 설정을 먹히지 않습니다.<br />
스토리북 자체적으로 `Webpack`을 돌리기 때문에 해당 설정도 같게 바꿔줘야 서로의 설정이 동일해집니다.<br />

+ `main.ts`

```ts
import path from "path";

// type
import type { StorybookConfig } from "@storybook/core-common";

const config: StorybookConfig = {
  /** 사용할 스토리 파일 위치 및 확장자 */
  stories: ["../src/components/**/*.stories.@(js|jsx|ts|tsx)"],

  /** 애드온(추가 기능) 설정 */
  addons: [
    "@storybook/addon-links",
    "@storybook/addon-essentials",
    "@storybook/addon-interactions",
  ],

  /** 사용하는 프레임워크 */
  framework: "@storybook/react",

  /** 스토리북 내부 기능 설정 */
  core: {
    /** Webpack 5 사용 */
    builder: "@storybook/builder-webpack5",
  },

  /** 웹팩 세팅 커스터마이징 */
  webpackFinal: async (config) => {
    return {
      ...config,
      resolve: {
        ...config.resolve,
        alias: {
          ...config.resolve?.alias,
          /** 절대 경로 세팅 */
          "@src": path.resolve(__dirname, "..", "src"),
        },
      },
    };
  },

  /** "typescript" 세팅 ( 공식문서에서 제공하는 세팅 ) */
  typescript: {
    /** "fork-ts-checker-webpack-plugin" 사용 여부 ( 별도의 타입 검사 플러그인 ) */
    check: false,

    /** "check" 활성화 시 "fork-ts-checker-webpack-plugin" 옵션 설정 */
    checkOptions: {},

    /** "react-docgen-typescript-plugin" 추가 ( 컴포넌트에서 사용한 타입을 추출해 문서로 만들어주는 도구 ) */
    reactDocgen: "react-docgen-typescript",

    /** "react-docgen-typescript"가 활성화된 경우 옵션 설정 */
    reactDocgenTypescriptOptions: {
      shouldExtractLiteralValuesFromEnum: true,
      propFilter: (prop) =>
        prop.parent ? !/node_modules/.test(prop.parent.fileName) : true,
    },
  },

  /** 스토리북 추가 기능 설정 */
  features: {
    storyStoreV7: true,
  },

  /** 로드할 정적 파일 설정 */
  staticDirs: [
    /** 파일 경로 */
  ],
};

module.exports = config;
```

## 1️⃣ 전역 설정
생성해준 `preview.js`를 이용해서 전역 설정을 해줍니다.<br />
`TypeScript` 기반으로 작성하기 때문에 `preview.tsx`로 변경해도 됩니다.<br />
( 정해진 이름으로 내보내기(`export`)를 하면 자동으로 전역 설정으로 등록됩니다. )<br />

기본으로 제공해주는 세팅과 `styled-components`를 이용한 전역 스타일 세팅을 어떻게 하는지에 대해 알아보겠습니다.<br />
( 사용하는 `css`에 따라서 각자의 방식을 `styled-components`처럼 적용하면 됩니다. )<br />

+ `preview.tsx`

```js
import React from "react";
import { ThemeProvider } from "styled-components";

// "webpack"의 설정 외부에 존재하는 파일이라 절대 경로 사용 불가능
import { GlobalStyle } from "../src/shared/global";
import theme from "../src/shared/theme";

/** global argTypes */
export const argTypes = {
  /**
   * 사용하는 모든 컴포넌트에 "props"로 "{ theme: 'light' | 'dark' }"를 부여할 수 있는 컨트롤러 설정
   * 추가적으로 컨트롤러는 "select"이고 옵션들 지정 ( "control"의 값들은 정해져 있고, 컨트롤러에 따라서 옵션을 설정할 수 있음 )
   * */
  theme: { control: "select", options: ["light", "dark"] },
};
/** global args */
export const args = {
  /** 모든 컴포넌트에 "props"로 "{ theme: 'dark' }" 부여 */
  theme: "dark",
};
/** global decorator */
export const decorators = [
  (Story) => (
    <ThemeProvider theme={theme}>
      <GlobalStyle />
      <Story />
    </ThemeProvider>
  ),
];

/** global parameters */
/** 스토리북의 기능과 애드온의 동작을 제어하기 위해 사용 */
export const parameters = {
  /** 클릭이 되었을 때 스토리북 UI의 Actions 패널에 나타날 콜백을 생성할 수 있도록 도와줌 */
  actions: { argTypesRegex: "^on[A-Z].*" },
  controls: {
    matchers: {
      color: /(background|color)$/i,
      date: /Date$/,
    },
  },
};
```

# 📚 컴포넌트 및 스토리 생성
개인적으로 같은 목적을 가진 파일들을 그룹화하는 것을 좋아해서 하나의 폴더로 묶어서 사용했습니다.<br />
( 직접 만든 `Modal` 컴포넌트를 예시로 사용하겠습니다. )<br />
( 부족함이 많아 스토리북의 설정을 어떻게 사용하는지 위주로 읽어주세요!! ... 🥲 )<br />

+ 파일 경로
  1. 컴포넌트: `src/components/stories/common/Modal/index.tsx`
  1. 스토리: `src/components/stories/common/Modal/index.stories.tsx`
  1. 스타일: `src/components/stories/common/Modal/style.tsx` ( `styled-components` 사용 )

## 0️⃣ 컴포넌트
일반 컴포넌트와 동일하게 작성하면 됩니다.<br />
( 필요한 `Props`의 타입을 정의 )<br />

스토리북에서 `Modal`의 `Props`를 보고 어느정도 추론해서 스토리를 생성합니다.<br />

```tsx
import { useEffect, useRef, useState } from "react";

// style
import StyledModal from "./style";

// type
import type { Shape, Size } from "@src/components/stories/@types";
export type Props = {
  title: string;
  contents?: string;
  shape?: Shape;
  size?: Size;

  onCancel?: () => void;
  onConfirm?: () => void;
};

const Modal = ({
  title,
  contents,
  shape = "primary",
  size = "medium",
  onCancel,
  onConfirm,
}: Props) => {
  /** 2023/03/04 - 모달 보여줄지 여부 - by 1-blue */
  const [show, setShow] = useState(true);
  /** 2023/03/04 - 모달 ref - by 1-blue */
  const modalRef = useRef<null | HTMLUListElement>(null);

  /** 2023/03/04 - ( 버튼이 없다면 ) 모달 외부 클릭 시 닫기 이벤트 등록 - by 1-blue */
  useEffect(() => {
    if (onCancel && onConfirm) return;

    const handleOutsideClose = (e: MouseEvent) => {
      // 클릭 이벤트 발생지가 "element"인지 확인
      if (!(e.target instanceof HTMLElement)) return;

      /**
       * 현재 열려있고, 모달이 포함한 엘리먼트가 아니라면
       * 즉, 모달 외부를 클릭한다면 닫기
       */
      if (show && (!modalRef.current || !modalRef.current.contains(e.target))) {
        setShow(false);
      }
    };

    // 모달 닫기 이벤트 등록
    document.addEventListener("click", handleOutsideClose);

    // 모달 닫기 이벤트 해제
    return () => document.removeEventListener("click", handleOutsideClose);
  }, [show]);

  return (
    <>
      {show && (
        <StyledModal shape={shape} size={size}>
          <section ref={modalRef}>
            <h4>{title}</h4>
            {contents && <p>{contents}</p>}

            {(onCancel || onConfirm) && (
              <div>
                {onCancel && (
                  <button type="button" onClick={onCancel}>
                    취소
                  </button>
                )}
                {onConfirm && (
                  <button type="button" onClick={onConfirm}>
                    확인
                  </button>
                )}
              </div>
            )}
          </section>
        </StyledModal>
      )}
    </>
  );
};

export default Modal;
```

## 1️⃣ 스타일
일반적인 `styled-components`을 사용하는 것처럼 동일하게 사용하면 됩니다.<br />
스타일을 부여한 컴포넌트에서 받는 `props`를 이용해서 스타일을 지정합니다.<br />
( + `Props` 타입과 `utility`를 활용해서 타입을 재구성합니다. 즉, 기존 타입을 기반으로 새로운 타입을 생성합니다. )<br />
( 기존 타입에 의존하기 때문에 기존 타입만 변경하면 자동으로 새로운 타입도 수정됩니다. )<br />

```tsx
import styled, { css } from "styled-components";

// type
import type { Props } from ".";
type StyledProps = Pick<Props, "shape" | "size">;

const StyledModal = styled.article<StyledProps>`
  position: fixed;
  inset: 0;

  display: flex;
  justify-content: center;
  align-items: center;

  background-color: rgba(0, 0, 0, 0.4);

  & > section {
    width: 60vw;
    height: 60vh;

    display: flex;
    flex-flow: column nowrap;

    border-radius: 0.3em;
    box-shadow: 2px 2px 10px #666;

    overflow-y: auto;

    /** 스크롤바 디자인 수정 */
    &::-webkit-scrollbar {
      /** 스크롤바의 너비 */
      width: 4px;
    }
    &::-webkit-scrollbar-thumb {
      /** 스크롤바 길이 */
      height: 25%;
      /** 스크롤바의 색상 */
      background: ${({ theme }) => theme.colors.blue500};
      border-radius: 10px;
    }
    &::-webkit-scrollbar-track {
      /** 스크롤바 뒷 배경 색상 */
      background: ${({ theme }) => theme.colors.blue100};
    }

    /** 모달 제목 */
    h4 {
      text-align: center;
    }
    /** 모달 내용 */
    p {
      padding: 0 1.4em;

      font-weight: 500;
      line-height: 1.4;
    }
    /** 모달 버튼 */
    div {
      flex: 1;

      padding: 0 1em;

      display: flex;
      justify-content: flex-end;
      align-items: flex-end;

      & > button {
        padding: 0.3em 0.6em;
        margin-right: 1em;

        border-radius: 0.2em;

        font-weight: bold;

        transition: all 0.3s;

        & + button {
          margin-right: 0;
        }
      }
    }

    & > * {
      margin: 0.8em 0;
      margin-bottom: 0;

      &:last-child {
        margin-bottom: 0.8em;
      }
    }

    /** 색상 */
    ${({ shape }) => {
      switch (shape) {
        case "primary":
          return css`
            background-color: #fff;

            h4 {
              color: #000;
            }
            p {
              color: ${({ theme }) => theme.colors.gray800};
            }
            & > div > button {
              background-color: ${({ theme }) => theme.colors.gray400};
              color: #fff;

              &:hover {
                background-color: ${({ theme }) => theme.colors.gray500};
              }
              &:active {
                background-color: ${({ theme }) => theme.colors.gray600};
              }
            }
          `;
        case "secondary":
          return css`
            background-color: ${({ theme }) => theme.colors.indigo500};

            h4 {
              color: #fff;
            }
            p {
              color: ${({ theme }) => theme.colors.gray100};
            }
            & > div > button {
              background-color: ${({ theme }) => theme.colors.gray100};
              color: ${({ theme }) => theme.colors.gray900};

              &:hover {
                background-color: ${({ theme }) => theme.colors.indigo100};
              }
              &:active {
                background-color: ${({ theme }) => theme.colors.indigo200};
              }
            }

            &::-webkit-scrollbar-thumb {
              background: ${({ theme }) => theme.colors.gray700};
            }
            &::-webkit-scrollbar-track {
              background: ${({ theme }) => theme.colors.gray100};
            }
          `;
        case "tertiary":
          return css`
            background-color: ${({ theme }) => theme.colors.teal500};

            h4 {
              color: #fff;
            }
            p {
              color: ${({ theme }) => theme.colors.gray100};
            }
            & > div > button {
              background-color: ${({ theme }) => theme.colors.gray100};
              color: ${({ theme }) => theme.colors.gray900};

              &:hover {
                background-color: ${({ theme }) => theme.colors.teal100};
              }
              &:active {
                background-color: ${({ theme }) => theme.colors.teal200};
              }
            }

            &::-webkit-scrollbar-thumb {
              background: ${({ theme }) => theme.colors.gray700};
            }
            &::-webkit-scrollbar-track {
              background: ${({ theme }) => theme.colors.gray100};
            }
          `;
      }
    }}

    /** 크기 */
    ${({ size }) => {
      switch (size) {
        case "tiny":
          return css`
            min-width: 200px;
            max-width: 300px;
            min-height: 100px;
            max-height: 200px;

            h4 {
              font-size: 1rem;
              font-weight: 700;
            }
            p {
              font-size: 0.8rem;
            }
            & > div > button {
              font-size: 0.6rem;
            }
          `;
        case "small":
          return css`
            min-width: 300px;
            max-width: 400px;
            min-height: 200px;
            max-height: 300px;

            h4 {
              font-size: 1.2rem;
              font-weight: 700;
            }
            p {
              font-size: 1rem;
            }
            & > div > button {
              font-size: 0.8rem;
            }
          `;
        case "medium":
          return css`
            min-width: 400px;
            max-width: 500px;
            min-height: 300px;
            max-height: 400px;

            h4 {
              font-size: 1.4rem;
              font-weight: 900;
            }
            p {
              font-size: 1.1rem;
            }
            & > div > button {
              font-size: 1rem;
            }
          `;
        case "large":
          return css`
            min-width: 600px;
            max-width: 800px;
            min-height: 400px;
            max-height: 600px;

            h4 {
              font-size: 1.6rem;
              font-weight: 900;
            }
            p {
              font-size: 1.2rem;
            }
            & > div > button {
              font-size: 1.1rem;
            }
          `;
      }
    }}
  }
`;

export default StyledModal;
```

## 2️⃣ 스토리
스토리북에서 사용할 여러 컴포넌트와 설정에 대한 설정을 하는 파일입니다.<br />

`args`에서 설정한 값들은 컴포넌트의 `props`로 들어가게 됩니다.<br />
또한 `argTypes`에 설정한 세팅도 사용자의 특정 액션에 의해서 컴포넌트의 `props`로 들어갑니다.<br />

`export default { /** ... */ }`(`(1)`)로 전역적으로 설정할 수도 있지만, 속성들은 해당 컴포넌트마다 다르게 구체적으로 설정(`(3)`)할 수 있습니다.<br />
또한 기본 내보낸 컴포넌트들(`(2)`)은 스토리북에 등록되게 됩니다.<br />

```tsx
// component
import Modal from "./index";

// type
import type { ComponentStory, ComponentMeta } from "@storybook/react";

/** (1) 해당 컴포넌트의 전역 설정 */
export default {
  /** 스토리북 앱의 사이드바에서 컴포너트를 참조하는 이름 ( "/"로 그룹화 ) */
  title: "Common/Modal",

  /** 사용할 컴포넌트 */
  component: Modal,

  /** 기본 "props" 지정 */
  args: {
    title: "모달 제목",
    contents: "Lorem ipsum dolor sit amet consectetur adipisicing elit.",
    shape: "tertiary",
    size: "medium",
    onCancel: undefined,
    onConfirm: undefined,
  },


  /** 특정 "props"들을 스토리북에서 커스터마이징하는 방법을 정의하는 속성 */
  argTypes: {
    /** "props"명 */
    title: {
      /** 스토리북 도구에서 명시할 이름 */
      name: "제목",

      /** 어떤 컨트롤러 사용 ( 컨트롤러에 따라 "options"가 필요할 수 있음 ) */
      control: { type: "text" },
    },
    contents: {
      control: { type: "text" },
    },
    shape: {
      control: { type: "radio" },
      options: ["primary", "secondary", "tertiary"],
    },
    size: {
      control: { type: "select" },
      options: ["tiny", "small", "medium", "large"],
    },
    onCancel: {
      control: { type: null },
    },
    onConfirm: {
      control: { type: null },
    },
  },

  /**
   * 컴포넌트를 감싸고 싶을 때 사용
   * 첫 번째 인자("Story")에는 해당 컴포넌트
   * 두 번째 인자("v")에는 컴포넌트의 정보에 대한 데이터가 들어옴
   * */
  decorators: [
    (Story, v) => {
      return (
        {%raw%}<div style={{ padding: "3rem", backgroundColor: "darkslategray" }}>{%endraw%}
          {Story()}
        </div>
      );
    },
  ],

  /** 애드온의 동작을 제어하는 데 사용되는 속성 */
  parameters: {
    /** 상단 nav에서 백그라운드 색상 선택에 제공할 옵션 */
    backgrounds: {
      values: [
        { name: "red", value: "#f00" },
        { name: "green", value: "#0f0" },
        { name: "blue", value: "#00f" },
      ],
    },

    /** 특정 액션에 대한 정의 */
    actions: {
      /** ".my"를 "click"한 경우 스토리북 도구의 "Actions"에서 감지 */
      handles: ["click .my"],
    },
  },
} as ComponentMeta<typeof Modal>;

/** 컴포넌트 템플릿 생성 */
const Template: ComponentStory<typeof Modal> = (args) => <Modal {...args} />;

/** (2) 함수 복사본 생성 */
export const Normal = Template.bind({});
/** (3) 컴포넌트에 전달할 인수 설정 ( props ) */
Normal.args = {};
Normal.argTypes = {};

/** (2) 함수 복사본 생성 */
export const ModalWithButton = Template.bind({});
/** (3) 컴포넌트에 전달할 인수 설정 ( props ) */
ModalWithButton.args = {
  onCancel() {
    console.log("취소버튼 클릭");
  },
  onConfirm() {
    console.log("확인버튼 클릭");
  },
};
```

## 3️⃣ 타입
크게 상관은 없는 부분이지만 없으면 코드 해석이 안되기 때문에 작성했습니다.<br />
`Modal` 컴포넌트에서 `import` 했던 타입입니다.<br />

```ts
/** 2023/03/03 -  크기 - by 1-blue */
export type Size = "tiny" | "small" | "medium" | "large";

/** 2023/03/04 - 형태 - by 1-blue */
export type Shape = "primary" | "secondary" | "tertiary";
```

## 4️⃣ StoryBook의 타입 추론
스토리북에서는 컴포넌트의 `props`를 보고 어느정도 추론해서 스토리북의 `argTypes`의 생성을 도와줍니다.<br />

아래는 여태까지의 예시와 상관없는 작은 예시입니다.<br />
아래 코드를 실행해보면 `argTypes`의 `radio` 설정이 없어도 `props`의 타입을 보고 추론해서 만들어주는 것 같습니다.<br />

`(4)`에서 리터럴 타입의 유니온으로 사용했기 때문에 `props`로 들어갈 수 있는 값은 세 가지밖에 없습니다.<br />
따라서 `(5)`처럼 설정을 해주지 않아도 자동으로 추론해서 정해진 타입을 설정해서 스토리북에서 적용해줍니다.<br />

```tsx
// ==================== MyComponent/index.tsx ====================
type Props = {
  /** (4) "props" 타입 설정 */
  backgroundColor: "#F00" | "#0F0" | "#00F";
};

const MyComponent = ({ backgroundColor }: Props) => {
  return (
    {%raw%}<div style={{ backgroundColor, width: "400px", height: "400px" }}></div>{%endraw%}
  );
};

export default MyComponent;

// ==================== MyComponent/index.stories.tsx ====================
import MyComponent from "./index";

// type
import type { ComponentStory, ComponentMeta } from "@storybook/react";

/** 해당 컴포넌트의 전역 설정 */
export default {
  name: "MyComponent",
  component: MyComponent,
  args: {
    backgroundColor: "#00F",
  },
  /** (5) 아래 구문이 없어도 타입을 보고 추론해서 라디오 버튼을 만들어줍니다. */
  // argTypes: {
  //   backgroundColor: {
  //     control: { type: "radio" },
  //     options: ["#F00", "#0F0", "#00F"],
  //   },
  // },
} as ComponentMeta<typeof MyComponent>;

/** 컴포넌트 템플릿 생성 */
const Template: ComponentStory<typeof MyComponent> = (args) => (
  <MyComponent {...args} />
);

/** 함수 복사본 생성 */
export const Normal = Template.bind({});

/** 컴포넌트에 전달할 인수 설정 ( props ) */
Normal.args = {};
```

# 🚀 배포
> 해당 내용을 읽기보다는 [스토리북 - Deploy](https://storybook.js.org/tutorials/intro-to-storybook/react/ko/deploy/){:target="_blank"}를 읽으시는 것을 추천드립니다.<br />
{:.prompt-info}

지금까지 작성한 스토리북을 배포하는 방법은 매우 간단합니다.<br />
스토리북 관리자가 만든 [Chromatic](https://www.chromatic.com){:target="_blank"}를 이용하면 됩니다.<br />

`CLI`를 통해서 현재 작성한 것을 바로 배포할 수 있지만, 저희는 `GitHub Actions`를 통한 배포 자동화 방법에 대해 설명하겠습니다.<br />

먼저 [크로마틱](https://www.chromatic.com/start?startWithSignup=true){:target="_blank"}에 `GitHub` 계정으로 로그인해서 인증 토큰을 얻습니다.<br />
해당 토큰을 복사해놓고 아래 `.yml` 파일을 작성합니다.<br />
( `.yml` 파일의 경로를 제대로 작성하지 않으면 `GitHub Actions`에서 인식하지 못합니다. )<br />

+ `.github/workflows/chromatic.yml`

```yml
name: 'Chromatic Deployment'

on: push

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v1
      - run: yarn
      - uses: chromaui/action@v1
        with:
          # (1) "chromatic"에서 제공한 토큰 ( "GitHub"에 직접 등록 )
          {%raw%}projectToken: ${{ secrets.CHROMATIC_PROJECT_TOKEN }}{%endraw%}
          # "GitHub Actions"에서 제공해주는 토큰
          {%raw%}token: ${{ secrets.GITHUB_TOKEN }}{%endraw%}
```

위에서 `(1)`에 해당하는 값은 `GitHub`에 직접 등록해줘야 합니다.<br />
물론 `.yml`에 직접 작성해도 동작하지만 중요한 키가 노출되기 때문에 숨겨주는 것이 좋습니다.<br />

+ `GitHub Actions`에 사용할 비밀 키 등록 방법
  1. 스토리북을 작성하는 레포지토리
  2. 상단 네비게이션의 `Settings`
  3. 좌측 네비게이션의 `Secrets ant variables`
  4. 좌측 네비게이션의 `Actions`
  5. `New repository secret`
  6. `Name`은 `CHROMATIC_PROJECT_TOKEN`, `Secret`은 크로마틱에서 얻은 토큰

위 순서대로 작성하면 비밀 키를 숨길 수 있습니다.<br />
`secrets.GITHUB_TOKEN`는 `GitHub Actions`가 돌아가는 시점에 자동으로 생성해주기 때문에 직접 작성해줄 필요 없습니다.<br />

# 📮 레퍼런스
1. [TypeScript + Storybook](https://storybook.js.org/docs/react/configure/overview#configure-your-project-with-typescript){:target="_blank"}
1. [Webpack + Storybook](https://storybook.js.org/docs/react/builders/webpack){:target="_blank"}
1. [Storybook - Deploy](https://storybook.js.org/tutorials/intro-to-storybook/react/ko/deploy/){:target="_blank"}
1. [Chromatic](https://www.chromatic.com){:target="_blank"}

1. [1-blue - Webpack](/posts/Webpack/){:target="_blank"}
1. [1-blue - Storybook 배포 결과물](https://www.chromatic.com/library?appId=64033003a278c2f727eaaeac&branch=master){:target="_blank"}