---
layout: post
title: blegram - 경고/에러 해결
author: admin
date: 2023-04-03 00:00:00 +900
lastmod: 2023-04-03 00:00:00 +900
sitemap:
  changefreq: monthly
  priority: 0.5
categories: [Project, blegram]
tags: [blegram, Next.js, TypeScript, react-query]
---

> 해당 프로젝트는 `Next.js` + `TypeScript`를 기반으로 만드는 인스타그램 클론 개인 프로젝트입니다.<br />프로젝트를 진행하면서 만났고 해결한 간단한 경고와 에러에 대한 부분을 작성하는 포스트입니다.<br />
{:.prompt-info}

## 🎛️ Next.js 13 Link와 styled-components 사용 문제
13 버전 이전의 `Link`는 `<a>`를 이용했지만, 13 버전부터는 `<a>`를 사용하지 않습니다.<br />

`styled-components`를 `Link`에 사용할 때는 `styled.a`와 같은 형태로 사용했는데 이제는 `styled(Link)`와 같이 사용해야 합니다.<br />
속성으로 무언가를 넘겨주지 않는 경우에는 상관이 없는데 넘겨주는 경우에는 아래와 같은 경고가 발생하게 됩니다.<br />

`"Warning: React does not recognize the "속성명" prop on a DOM element. If you intentionally want it to appear in the DOM as a custom attribute, spell it as lowercase "속성명" instead. If you accidentally passed it from a parent component, remove it from the DOM element."`

여러 방법을 시도해봤지만 결국 성공한 방법은 없었고 가장 간단하게 `<Link>`의 `legacyBehavior`를 사용해서 13 버전 이전의 방법처럼 `<a>`에 `styled-components`를 적용해서 사용했습니다.<br />

## 🧽 Next.js 13 404 페이지
Next.js 13 버전에서는 [`not-found.tsx`](https://beta.nextjs.org/docs/api-reference/file-conventions/not-found){:target="_blank"}와 같은 형태로 파일을 작성하면 일치하지 않는 경로의 페이지로 대체하는 것으로 이해했습니다.<br />

근데 실제로 작성해보면 제대로 동작을 하지 않습니다.<br />
[stackoverflow](https://stackoverflow.com/questions/75302340/not-found-page-does-not-work-in-next-js-13){:target="_blank"}를 참고해서 `/app`의 최상위에 `[...not-found]`를 사용해서 `404`를 처리했습니다.<br />
동적 라우팅이기 때문에 `[...page]`처럼 이름을 바꿔도 동일하게 동작합니다.<br />

```tsx
import { notFound } from "next/navigation";

/** 2023/04/03 - 존재하지 않는 경로 처리 ( [참고](https://stackoverflow.com/questions/75302340/not-found-page-does-not-work-in-next-js-13) ) - by 1-blue */
const NotFoundCatchAll = () => notFound();

export default NotFoundCatchAll;
```


## 📮 레퍼런스
1. [stackoverflow](https://stackoverflow.com/questions/75302340/not-found-page-does-not-work-in-next-js-13){:target="_blank"}

1. [GitHib - blegram](https://github.com/1-blue/blegram){:target="_blank"}