---
layout: post
title: blegram - 간단한 구현
author: admin
date: 2023-04-13 21:16:00 +900
lastmod: 2023-04-22 01:35:00 +900
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

## 🚟 전역 모달
전역 모달을 위해서 `recoil`을 이용했습니다.<br />

모달마다 각자의 레이아웃이 있기 때문에 각각의 모달마다 서로 다른 `atom`과 레이아웃을 세트로 만들려고 합니다.<br />
`atom`에 `postIdx`를 저장해서 모달이 열릴 때마다 현재 게시글의 식별자를 저장하고 모달을 관리합니다.<br />
모달이 열리면 `postIdx`를 이용해서 해당 게시글을 식별해서 각자의 처리를 수행합니다.<br />

+ [`/src/recoil/atoms/index.ts`](https://github.com/1-blue/blegram/tree/master/src/recoil/atoms/index.ts){:target="_blank"}

```ts
import { atom } from "recoil";

interface AtomModalOfPost {
  isOpen: boolean;
  isMine: boolean;
  postIdx: null | number;
}
/** 2023/04/14 - post modal atom - by 1-blue */
export const atomModalOfPost = atom<AtomModalOfPost>({
  key: "AtomModalOfPost",
  default: {
    isOpen: false,
    isMine: false,
    postIdx: null,
  },
});

interface AtomModalOfLiker {
  isOpen: boolean;
  postIdx: null | number;
}
/** 2023/04/25 - liker modal atom - by 1-blue */
export const atomModalOfLiker = atom<AtomModalOfLiker>({
  key: "AtomModalOfLiker",
  default: {
    isOpen: false,
    postIdx: null,
  },
});

```

+ [`/src/hooks/recoil/usePostModal.ts`](https://github.com/1-blue/blegram/tree/master/src/hooks/recoil/usePostModal.ts){:target="_blank"}

```ts
import { useCallback } from "react";
import { useRecoilState } from "recoil";

// atom
import { atomModalOfPost } from "@src/recoil/atoms";

/** 2023/04/14 - 전역 게시글 모달 훅 - by 1-blue */
const usePostModal = () => {
  const [postModalData, setPostModalData] = useRecoilState(atomModalOfPost);

  /** 2023/04/14 - 모달 닫기 핸들러 - by 1-blue */
  const closePostModal = useCallback(
    () => setPostModalData((prev) => ({ ...prev, isOpen: false })),
    [setPostModalData]
  );

  /** 2023/04/14 - 모달 열기 핸들러 - by 1-blue */
  const openPostModal = useCallback(
    (isMine: boolean, postIdx: number) =>
      setPostModalData((prev) => ({ ...prev, isOpen: true, isMine, postIdx })),
    [setPostModalData]
  );

  return { postModalData, closePostModal, openPostModal };
};

export default usePostModal;
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

## 🫧 이벤트 버블링
> 아래 코드는 실제 코드가 아닌 dataset과 버블링을 활용해서 이벤트를 하나만 사용하는 예시입니다.<br />실제 코드가 보고 싶다면 [`PostComments`의 `onDeleteComment()`](https://github.com/1-blue/blegram/tree/master/src/components/Post/PostComments/index.tsx){:target="_blank"}와 [`PostComment`의 삭제 버튼(`data-idx`)](https://github.com/1-blue/blegram/tree/master/src/components/Post/PostComment/index.tsx){:target="_blank"}을 참고해주세요!<br />
{:.prompt-info}


댓글의 삭제 기능을 구현하는데 이벤트 버블링과 `datset`을 활용했습니다.<br />

원래는 각 버튼에 `onClick`으로 댓글을 삭제하는 버튼을 모두 `props`로 내리는 것이 아닌 모든 댓글을 갖는 컴포넌트 하나에 `onClick`을 달고 `dataset`으로 해당 댓글의 식별자를 얻어서 처리하면 하나의 `onClick` 이벤트를 이용해서 모든 댓글 삭제에 대한 이벤트를 처리할 수 있습니다.<br />

```tsx
import { useCallback } from "react";

const comments = Array(20)
  .fill(null)
  .map((v, i) => ({ idx: i }));

const Component = () => {
  const onClick: React.MouseEventHandler<HTMLUListElement> = useCallback(
    (e) => {
      if (!(e.target instanceof HTMLButtonElement)) return;

      // 1, 2, 3, 4, 5 ... ( "dataset"을 이용해서 하나의 이벤트 핸들러 함수로 각 댓글을 구별할 수 있습니다. )
      console.log(e.target.dataset.idx);
    },
    []
  );

  return (
    <ul onClick={onClick}>
      {comments.map((comment) => (
        <li key={comment.idx}>
          <button type="button" data-idx={comment.idx}>
            삭제
          </button>
        </li>
      ))}
    </ul>
  );
};
```

## 🛝 모달 오픈 시 외부 스크롤 금지
모달이 열려있는지 여부에 의해서 `body`의 `overflow`에 `hidden` or `auto`를 주면 됩니다.<br />

+ [`/src/components/Post/index.tsx`](https://github.com/1-blue/blegram/tree/master/src/components/src/components/Post/index.tsx){:target="_blank"}

```tsx
// 특정 컴포넌트 내부

/** 2023/04/25 - 외부 스크롤 금지 - by 1-blue */
useEffect(() => {
  // 모달이 열려있다면
  if (postModalData.isOpen || likerModalData.isOpen) {
    document.body.style.overflow = "hidden";
  }
  // 모달이 닫혀있다면
  else {
    document.body.style.overflow = "auto";
  }
}, [postModalData, likerModalData]);
```

## 🚪 모달 외부 클릭 시 닫기
모달은 어디든 배치될 수 있기 때문에 이벤트 버블링을 활용해서 `window`에 이벤트를 등록하고, 모달이 닫히면 이벤트를 해제하도록 코드를 구성했습니다.<br />
그리고 등록한 이벤트에서 `Node.contain()`을 이용해서 현재 클릭한 엘리먼트가 모달 내부에 존재하는지 확인하고 모달을 닫거나 열도록 로직을 구성했습니다.<br />

+ [`/src/components/Post/index.tsx`](https://github.com/1-blue/blegram/tree/master/src/components/src/components/Post/index.tsx){:target="_blank"}

```tsx
// 특정 컴포넌트 내부

/** 2023/04/25 - 외부 클릭 시 모달 닫기 - by 1-blue */
useEffect(() => {
  const modalCloseHandler = (e: MouseEvent) => {
    if (!likerModalData.isOpen) return;
    if (!(e.target instanceof HTMLElement)) return;
    if (e.target instanceof HTMLButtonElement) return;
    if (!modalRef.current) return;
    // 위쪽은 부가적인 부분이라 수정/삭제를 해도 되고 아래 부분이 핵심
    // 현재 클릭한 엘리먼트가 모달의 내부에 존재하는 엘리먼트인지 확인
    if (modalRef.current.contains(e.target)) return;

    // 모달을 닫는 함수
    closeLikerModal();
  };

  window.addEventListener("click", modalCloseHandler);
  return () => window.removeEventListener("click", modalCloseHandler);
}, [likerModalData, closeLikerModal]);
```

## 📮 레퍼런스
1. [react-slick](https://react-slick.neostack.com){:target="_blank"}
1. [recoil](https://recoiljs.org/ko){:target="_blank"}