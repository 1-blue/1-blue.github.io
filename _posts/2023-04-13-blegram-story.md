---
layout: post
title: blegram - 간단한 구현
author: admin
date: 2023-04-13 21:16:00 +900
lastmod: 2023-04-13 21:16:00 +900
sitemap:
  changefreq: monthly
  priority: 0.5
categories: [Project, blegram]
tags: [blegram, Next.js, TypeScript, react-slick]
---

> 해당 프로젝트는 `Next.js` + `TypeScript`를 기반으로 만드는 인스타그램 클론 개인 프로젝트입니다.<br />프로젝트를 진행하면서 겪은 간단하지만 기록하고 싶은 부분을 작성하는 포스트입니다.<br />
{:.prompt-info}

## 🎠 Carousel 컴포넌트
[`react-slick`](https://react-slick.neostack.com){:target="_blank"}을 이용해서 만든 이미지 캐루셀입니다.<br />

`styled-components`와 같이 사용해서 커스터마이징했으며, 외부에서 `<Carousel />`을 감싸는 컴포넌트의 높이에 맞게 이미지의 높이가 결정됩니다.<br />
또한 `dot`을 커스터마이징해서 배경색과 대비되도록 만들었습니다.<br />

+ [`/src/components/common/Carusel/style.ts`](TODO:){:target="_blank"}

```ts
import styled from "styled-components";
import Slider from "react-slick";

/** 2023/04/13 - "react-slick"의 "Slider" 타입 - by 1-blue */
const StyledSlider = styled(Slider)`
  height: 100%;

  /** 이미지 높이 결정을 위함 */
  div {
    height: inherit;
  }
  & figure {
    height: inherit;

    position: relative;
  }

  /* dots customizing */
  .custom-dots {
    display: flex;
    text-align: center;
    margin-top: 1em;

    & > li {
      display: inline-block;
      margin: 0 6px;
      padding: 0;

      cursor: pointer;

      & > button {
        display: block;
        height: 8px;
        width: 8px;
        padding: 0;

        border: none;
        color: transparent;
        background: ${({ theme }) =>
          theme.isDark ? theme.colors.gray500 : theme.colors.gray400};
        border-radius: 50%;
        cursor: pointer;
      }

      &.slick-active > button {
        background: ${({ theme }) => theme.colors.fg};
      }
    }
  }
`;

export default StyledSlider;
```

+ [`/src/components/common/Carusel/index.tsx`](TODO:){:target="_blank"}

```tsx
import Image from "next/image";

// util
import { blurDataURL } from "@src/utils";

// style
import StyledSlider from "./style";

// type
import type { Settings } from "react-slick";
interface Props {
  photos: string[];
}

/** 2023/04/08 - react-slick setting - by 1-blue */
const settings: Settings = {
  dots: true,
  infinite: true,
  slidesToShow: 1,
  slidesToScroll: 1,
  touchMove: true,
  arrows: false,
  dotsClass: "custom-dots",
};

/** 2023/04/13 - 이미지 캐루셀 - by 1-blue */
const Carousel: React.FC<Props> = ({ photos }) => (
  <StyledSlider {...settings}>
    {photos.map((photo) => (
      <figure key={photo}>
        <Image
          src={photo}
          alt="이미지"
          fill
          quality={75}
          placeholder="blur"
          blurDataURL={blurDataURL}
        />
      </figure>
    ))}
  </StyledSlider>
);

export default Carousel;
```

## 📮 레퍼런스
1. [react-slick](https://react-slick.neostack.com){:target="_blank"}