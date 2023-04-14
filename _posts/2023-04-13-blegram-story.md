---
layout: post
title: blegram - 간단한 구현
author: admin
date: 2023-04-13 21:16:00 +900
lastmod: 2023-04-14 23:27:00 +900
sitemap:
  changefreq: monthly
  priority: 0.5
categories: [Project, blegram]
tags: [blegram, Next.js, TypeScript, react-slick, recoil]
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

## 🚟 게시글 전역 모달
전역 모달을 위해서 `recoil`을 이용했습니다.<br />

모달마다 각자의 레이아웃이 있기 때문에 각각의 모달마다 서로 다른 `atom`과 레이아웃을 세트로 만들려고 합니다.<br />
`atom`에 `postIdx`를 저장해서 모달이 열릴 때마다 현재 게시글의 식별자를 저장하고 모달을 관리합니다.<br />
모달이 열리면 `postIdx`를 이용해서 해당 게시글을 식별해서 각자의 처리를 수행합니다.<br />

+ [`/src/recoil/atoms/index.ts`](https://github.com/1-blue/blegram/tree/master/src/recoil/atoms/index.ts){:target="_blank"}

```ts
import { atom } from "recoil";

interface ModalAtom {
  isOpen: boolean;
  isMine: boolean;
  postIdx: null | number;
}

export const modalAtom = atom<ModalAtom>({
  key: "modalAtom",
  default: {
    isOpen: false,
    isMine: false,
    postIdx: null,
  },
});
```

+ [`/src/hooks/recoil/useModalOfPost.ts`](https://github.com/1-blue/blegram/tree/master/src/hooks/recoil/useMpdalOfPost.ts){:target="_blank"}

```ts
import { useCallback } from "react";
import { useRecoilState } from "recoil";

// atom
import { modalAtom } from "@src/recoil/atoms";

/** 2023/04/14 - 전역 모달 훅 - by 1-blue */
const useModalOfPost = () => {
  const [modalData, setModalData] = useRecoilState(modalAtom);

  /** 2023/04/14 - 모달 닫기 핸들러 - by 1-blue */
  const closeModal = useCallback(
    () => setModalData((prev) => ({ ...prev, isOpen: false })),
    [setModalData]
  );

  /** 2023/04/14 - 모달 열기 핸들러 - by 1-blue */
  const openModal = useCallback(
    (isMine: boolean, postIdx: number) =>
      setModalData((prev) => ({ ...prev, isOpen: true, isMine, postIdx })),
    [setModalData]
  );

  return { modalData, closeModal, openModal };
};

export default useModalOfPost;
```

+ [`/src/components/common/Modal/Post/index.tsx`](https://github.com/1-blue/blegram/tree/master/src/components/common/Modal/Post/index.tsx){:target="_blank"}

```tsx
import { useCallback } from "react";
import { toast } from "react-toastify";

// hook
import useDeletePost from "@src/hooks/query/useDeletePost";
import useModalOfPost from "@src/hooks/recoil/useModalOfPost";

// component
import Icon from "@src/components/common/Icon";

// style
import StyledModal from "./style";

/** 2023/04/14 - 게시글의 모달 ( 수정, 삭제, 북마크, 링크복사 ) - by 1-blue */
const Post = () => {
  /** 2023/04/11 - 게시글 제거 훅 - by 1-blue */
  const deletePostMutate = useDeletePost();

  /** 2023/04/11 - 게시글의 모달관련 훅 - by 1-blue */
  const { modalData, closeModal } = useModalOfPost();

  /** 2023/04/11 - copy clipboard - by 1-blue */
  const copyLink = useCallback(() => {
    navigator.clipboard
      .writeText(window.location.origin + `?postIdx=${modalData.postIdx}`)
      .then(() => toast.success("게시글 링크를 복사했습니다."));
  }, [modalData]);

  return (
    <StyledModal onClick={closeModal}>
      <div>
        <button type="button">
          <Icon shape="bookmark" size="xs" color="#000" hover="#FFF" />
          <span>저장</span>
        </button>
        <button type="button" onClick={copyLink}>
          <Icon shape="link" size="xs" color="#000" hover="#FFF" />
          <span>링크</span>
        </button>
        {modalData.isMine && (
          <>
            <button type="button">
              <Icon shape="pencil" size="xs" color="#000" hover="#FFF" />
              <span>수정</span>
            </button>
            <button
              type="button"
              onClick={() =>
                modalData.postIdx &&
                deletePostMutate({ idx: modalData.postIdx })
              }
            >
              <Icon shape="trash" size="xs" color="#000" hover="#FFF" />
              <span>삭제</span>
            </button>
          </>
        )}
      </div>
    </StyledModal>
  );
};

export default Post;
```

## 📮 레퍼런스
1. [react-slick](https://react-slick.neostack.com){:target="_blank"}
1. [recoil](https://recoiljs.org/ko){:target="_blank"}