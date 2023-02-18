---
title: Intersection Observer API과 React.js로 무한 스크롤링 구현
author: admin
date: 2022-12-25 10:15:00 +900
lastmod: 2023-02-18 22:00:00 +900
sitemap:
  changefreq: monthly
  priority: 0.5
categories: [FrontEnd, React]
tags: [Intersection Observer API, InfiniteScrolling, 무한 스크롤링]
---

> 해당 포스트는 `Intersection Observer API`의 기본 사용법과 `React.js`에서의 활용을 통한 무한 스크롤링 구현 방법에 대한 포스트입니다. ( [참고한 포스트](https://velog.io/@elrion018/%EC%8B%A4%EB%AC%B4%EC%97%90%EC%84%9C-%EB%8A%90%EB%82%80-%EC%A0%90%EC%9D%84-%EA%B3%81%EB%93%A4%EC%9D%B8-Intersection-Observer-API-%EC%A0%95%EB%A6%AC){:target="_blank"} )
{: .prompt-info}

# 🔬 Intersection Observer API
`Intersection Observer API`는 루트 요소와 타겟 요소의 교차점을 관찰하고 루트 요소와 타겟 요소가 교차하는지 판단하는 기능 제공합니다.<br />
즉, 쉽게 말해서 특정 `element`가 뷰포트(화면)에 들어왔는지, 혹은 들어오는 시점을 알려주는 `API`입니다.<br />

## 0️⃣ 기본 형태
```ts
const callback = (entries: IntersectionObserverEntry[], observer: IntersectionObserver) => {
  entries.forEach(entry => {
    // // 타겟 요소가 루트 요소와 교차하는 점이 없으면 콜백을 호출했으되, 조기에 탈출한다.
    // if (entry.intersectionRatio <= 0) return

    // "isIntersecting"은 타켓요소가 루트요소에 들어오는 그 지점에 "true"가 됩니다.
    // "threshold"를 3개 줬으니 25%, 50%, 75%에 각각 한번씩 총 3번 실행됩니다.
    if (!entry.isIntersecting) return

    console.log("25% or 50% or 75%가 뷰포트 내부로 들어왔음을 감지!");
  })
};
const options = {
  // 기본값 null -> 뷰포트가 됨
  // root: document.querySelector(".container"),

  // root 요소의 범위 확장
  rootMargin: "0px",

  // 타겟요소의 어느정도가 루트요소에 들어왔는지 기준을 작성
  threshold: [0.25, 0.5, 0.75]
}

let observer = new IntersectionObserver(callback, options)

```
+ `options`
  1. `root`: 타켓 요소의 가시성을 확인할 때 사용하는 루트 요소 ( 이걸 어떻게 활용하는지는 모르겠음 )
            즉, `root`를 기준으로 타켓 요소가 `root`에 들어오는지 감시 ( 반드시 타켓의 조상 )
  2. `rootMargin`: `root`의 범위를 확장 ( 단위를 반드시 입력 ex) `px`, `em` 등 )
  3. `threshold`: 타켓 요소가 특정 범위 이상으로 뷰포트에 들어왔는지 정하는 값
            예를 들어 `0.5`라면 타켓 요소의 `50%`가 뷰포트에 들어오면 `callback`을 실행
            만약 범위별로 실행하고 싶다면 `[0, 0.25, 0.5, 0.75, 1]`과 같이 선언하면 `0%~100%`사이의 `25%`단위로 `callback`실행

# ♾️ 무한 스크롤링
무한 스크롤링이란 스크롤을 특정 지점 이하로 내릴 때마다 계속 데이터를 패치해서 화면에 렌더링해서 화면을 계속 채워서 무한으로 스크롤을 내릴 수 있는 것처럼 보이게 만드는 기능을 말합니다.<br />

보통은 스크롤 이벤트로 구현을 합니다.<br />
1. 스크롤을 하면 스크롤 이벤트가 발생
2. 현재 스크롤 위치와 브라우저 최하단 위치 계산
3. 일정부분 이하로 내려갔다면 데이터 패치 후 화면에 렌더링

추가적으로 `throttling` 혹은 `debouncing`을 적용해서 현재 데이터를 요청중이라면 추가로 데이터 요청을 보내지 않도록 설계하는 것도 중요합니다.<br />
( 데이터 요청후에도 스크롤을 계속 진행한다면 발생하는 스크롤 이벤트만큼 서버로 데이터 요청이 가기 때문에 프론트에서 `throttling` or `debouncing`를 이용해서 막아주는 것이 좋습니다. )<br />

하지만 스크롤 이벤트는 한번의 스크롤으로 인해 수십번의 이벤트가 발생하기 때문에 낭비가 발생한다는 생각이 들었습니다.<br />
저의 의도는 스크롤이 특정 위치 이하로 내려간 경우 단 한번만 실행하면 되는데 위에서 아래까지 내려오는데 수백 수천번이 실행될 수 있기 때문입니다.<br />
추가적으로 스크롤 이벤트는 동기적 실행이라서 메인 스레드에 부담이 된다고 하네요.<br />
또한 `getBoundingClientRect()`로 `element`를 관찰하는 경우에는 `reflow`가 발생해서 성능상으로 안좋다고 합니다.<br />

> `reflow`는 렌더트리의 노드들의 정확한 크기와 위치를 다시 계산하는 과정을 말합니다.
{: .prompt-info}

## 0️⃣ React에서 Intersection Observer를 이용한 무한 스크롤링 예시 ( feat. TS )
```tsx
import { useCallback, useEffect } from "react";

type Props = {
  observerRef: HTMLDivElement | null;
  fetchMore: () => void;
  hasMore: boolean;
};

// "IntersectionObserver"의 옵션들
const options: IntersectionObserverInit = {
  threshold: 0.1,
};

/**
 * 무한 스크롤링 적용 훅
 * @param observerRef 감시할 element ref
 * @param fetchMore 추가 패치를 실행할 함수
 * @param hasMore 더 패치할 수 있는지 여부
 * @returns
 */
const useInfiniteScrolling = ({ observerRef, fetchMore, hasMore }: Props) => {
  // 뷰포트 내에 감시하는 태그가 들어왔다면 패치
  const onScroll: IntersectionObserverCallback = useCallback(
    (entries, observer) => {
      if (!entries[0].isIntersecting) return;

      // "redux"를 쓴다면 () => { dispatch(/* */); }
      fetchMore();
    },
    [fetchMore]
  );
  // observer 등록 ( 해당 태그가 뷰포트에 들어오면 게시글 추가 패치 실행 )
  useEffect(() => {
    if (!observerRef) return;

    // 콜백함수와 옵션값 지정
    let observer = new IntersectionObserver(onScroll, options);
    // 특정 요소 감시 시작
    observer.observe(observerRef);

    // 더 가져올 게시글이 존재하지 않는다면 패치 중지
    if (!hasMore) observer.unobserve(observerRef);

    // 감시 종료
    return () => observer.disconnect();
  }, [observerRef, onScroll, hasMore]);

  return;
};

export default useInfiniteScrolling;
```

1. 컴포넌트 제일 하단에 보이지 않고 높이가 매우 낲은 `<div />`를 선언하고 ref를 지정합니다.
2. 데이터를 추가로 패치하는 콜백 함수를 생성합니다. ( `fetchMore()` )
3. 데이터를 더 패치할 수 있는지 판단할 변수를 생성합니다. ( `hasMore` 서버에 데이터가 남아있다면 추가로 패치 가능 )
4. `1`, `2`, `3`에서 생성한 변수/함수를 `useInfiniteScrolling()`에 넣습니다.
5. `1`에서 생성한 `<div />`가 화면에 들어온다면 데이터를 추가로 패치하고 렌더링하면 `<div />`가 다시 맨 아래로 내려가서 데이터만 있다면 무한으로 반복할 수 있습니다.

## 1️⃣ StackBlitz 예시

<iframe height="600" style="width: 100%;" scrolling="no" title="Intersection Observer" src="https://stackblitz.com/edit/react-ts-mpggtg?ctl=1&embed=1&file=src/components/App.tsx" frameborder="no" loading="lazy" allowtransparency="true" allowfullscreen="true"></iframe>

# 📮 레퍼런스
1. [elrion018 - 실무에서 느낀 점을 곁들인 Intersection Observer API 정리](https://velog.io/@elrion018/%EC%8B%A4%EB%AC%B4%EC%97%90%EC%84%9C-%EB%8A%90%EB%82%80-%EC%A0%90%EC%9D%84-%EA%B3%81%EB%93%A4%EC%9D%B8-Intersection-Observer-API-%EC%A0%95%EB%A6%AC){:target="_blank"}