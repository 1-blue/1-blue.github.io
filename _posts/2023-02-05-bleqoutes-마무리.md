---
layout: post
title: 개인 프로젝트 - blequotes 마무리
author: admin
date: 2023-02-05 20:57:00 +900
lastmod: 2023-02-18 20:05:00 +900
sitemap:
  changefreq: monthly
  priority: 0.5
categories: [Project, bleqoutes]
tags: [프로젝트, 개인 프로젝트, 명대사]
---

> 해당 포스트는 개인 프로젝트인 `blequotes` 마무리 포스트입니다.<br />
{: .prompt-info}

# 😶 blequotes란
> `ble`는 제가 만든 프로젝트에 식별자처럼 사용하는 접두사입니다. ( 여러가지 조합하다가 `GitHub`이름인 `1-blue`의 일부분인 `ble`를 사용하게 되었습니다... 🙂 )<br />
{: .prompt-tip}

영화 / 드라마 / 도서의 명대사를 간단하게 작성하는 커뮤니티입니다.<br />

# 🧑‍💻 사용한 기술
1. `TypeScript`
2. `React.js`
3. `TailwindCss`
4. `Redux-Toolkit`
5. `React-Hook-Form`
6. `Express`
7. `Prisma`
8. `AWS-S3`
9. `AWS-EC2` ( 사용 예정 )

# 🔨 사용한 툴
1. `Git` / `GitHub`
2. `Notion`
3. `SourceTree`
4. `VSCode`
5. `Figma` ( 처음부터 사용한 것은 아님 )

# 🕹️ 구현 기능
1. [`Movie DB API`](https://developers.themoviedb.org/3){:target="_blank"}를 이용한 영화 및 드라마들의 각종 정보 패치 및 검색 
2. [`Kakao Book API`](https://developers.kakao.com/docs/latest/ko/daum-search/dev-guide#search-book){:target="_blank"}를 이용한 도서 검색
3. `Image Carousel` ( `react-slick` 사용 )
4. 명대사 등록 기능 ( [`AWS-S3`의 `presignedURL`](https://1-blue.github.io/posts/AWS-S3-presignedURL/){:target="_blank"} 기능을 이용한 이미지 등록 )
5. 명대사에 좋아요 및 싫어요 기능
6. 영화 / 드라마 / 도서 검색 기능 ( [`Debouncing`](https://1-blue.github.io/posts/%EB%94%94%EB%B0%94%EC%9A%B4%EC%8B%B1-%EC%93%B0%EB%A1%9C%ED%8B%80%EB%A7%81-%EB%A9%94%EB%AA%A8%EC%9D%B4%EC%A0%9C%EC%9D%B4%EC%85%98/#%EF%B8%8F-%EB%94%94%EB%B0%94%EC%9A%B4%EC%8B%B1--debouncing-){:target="_blank"} 사용 )
7. 무한 스크롤링 ( [`Intersection-Observer`](https://1-blue.github.io/posts/Intersection-Observer-API/){:target="_blank"} 사용 )
8. [`React-ToolKit`](https://1-blue.github.io/posts/Redux-Toolkit/){:target="_blank"}을 이용한 전역 상태 관리
9. [`Prisma`](https://1-blue.github.io/posts/prisma/){:target="_blank"}를 이용한 `DB`관리 및 데이터 관리

# 🍀 제작 환경
1. OS: `Window11`
2. editor: `VSCode`, `Sourcetree` ( + `Notion`, `Figma` )
3. terminal: `git bash`
4. Database: `Mysql`
6. vcs: `Git` / `GitHub`
7. Front: `React.js`
8. Back: `Node`의 `Express`
9. 이미지 저장소: `AWS S3`
10. 배포: AWS-EC2 예정 ( 배포후 수정 )

# 🃏 알아둬도 쓸데없는 잡다한 이야기
## 0️⃣ 메타데이터 처리
`react-helmet-async`를 이용해서 메타데이터를 처리했습니다.<br />

```tsx
import { Helmet } from "react-helmet-async";

type Props = {
  title: string;
  description: string;
  image?: string;
};

const HeadInfo = ({ title, description, image = "/logo.png" }: Props) => {
  return (
    <Helmet>
      <title>{title + " ⟪ 1-blue ⟫"}</title>
      <meta name="description" content={description} />

      {/* FIXME: URL 결정되면 수정 */}
      {/* <meta property="og:url" content="" /> */}
      <meta property="og:title" content={title} />
      <meta property="og:description" content={description} />
      <meta property="og:image" content={image} />

      <meta name="twitter:card" content={title + "\n" + description} />
      <meta name="twitter:title" content={title} />
      <meta name="twitter:description" content={description} />
      <meta name="twitter:image" content={image} />
    </Helmet>
  );
};

export default HeadInfo;
```


## 1️⃣ 웹 접근성을 위한 노력
1. 전체적인 레이아웃은 시멘틱 태그들을 이용해서 처리했고, `<div>`보다는 의미 있는 태그들을 사용하려고 노력<br />
  ( `<header>`, `<main>`, `<footer>`, `<aside>`, `<article>`, `<section>`, `<ul>`, `<li>`, `<time>`, `<figure>` 등 )<br />
2. `<label>`, `alt` 속성 등을 사용
3. `active`, `focus`, `hover` 등에 일관성을 유지하기 위한 스타일을 적용
4. 잘못된 접근은 예외 페이지로 이동시킴
5. 추천 검색어 방향키로 이동하는 기능

## 2️⃣ 타입 설계를 위한 노력
타입을 최대한 나누고 새로운 타입이 필요한 경우 기존 타입을 이용해서 새로운 타입을 만들었고 유틸리티 타입을 최대한 이용해서 기존 타입에 의존적이도록 코드를 구현하려고 노력했습니다.<br />

또한 [이펙티브 타입스크립트](/posts/이펙티브-타입스크립트-2장/#-item-12--함수-표현식에-타입-적용하기-){:target="_blank"}에서 본 것처럼 함수 시그니처 타입을 적극적으로 사용했고, `React`에서도 제공해주는 `Handler` 타입을 사용했습니다.<br />

## 3️⃣ API 설계
> [API](https://github.com/1-blue/blequotes#-api){:target="_blank"}에서 확인할 수 있습니다.<br />
{:.prompt-info}

`HTTP API`의 규칙을 지키려고 노력했고, `HTTP Method`, `URI`, `HTTP Status Code`를 이용해서 데이터를 송/수신했으며, 어떤 데이터 형태로 주고 받을지 상세하게 작성했습니다.<br />
또한 `TypeScript`를 이용해서 송/수신 타입을 만들어서 적극적으로 사용했습니다.<br />

## 4️⃣ 반응형 레이아웃
`TailwindCss` 반응형 디자인을 이용해서 `400px`, `500px`, `640px`, `768px`, `1024px` 등의 사이즈에 맞게 레이아웃을 배치했습니다.<br />
또한 기본 폰트는 `16px`, `400px`이하일 경우 폰트는 `14px`로 조정했습니다.<br />

## 5️⃣ 빌드와 실행 ( React + Express )
현재 계획으로는 배포를 하면 하나의 서버에서 프론트, 백엔드, 데이터베이스를 모두 돌릴 계획이기 때문에 빌드한 `React`의 결과물을 `Express`의 정적 파일 제공 기능을 통해서 서버에 제공합니다. ( [참고](https://create-react-app.dev/docs/deployment/){:target="_blank"} )<br />

단, `React`로 만든 페이지는 `SPA`이기 때문에 라우팅에 대한 특별한 처리가 필요합니다.<br />
( 특정 페이지에서 새로고침 시 `404` 오류 발생 )<br />

```ts
// app.ts ( 진입점 ) ( ... 나머지 생략 )

// deploy
app.use(express.static(path.join(__dirname, "../../frontend/build")));
app.get("/", (req, res) => {
  res.sendFile(path.join(__dirname, "../../frontend/build", "index.html"));
});

// SPA routing 새로고침 문제 해결을 위한 코드
app.get("*", (req, res) => {
  res.sendFile(path.resolve(__dirname, "../../frontend/build", "index.html"));
});
```

## 6️⃣ 좋아요/싫어요 구현에 대한 비밀
현재 웹사이트는 로그인 기능이 없기 때문에 이용자를 식별할 방법이 없습니다.<br />

좋아요/싫어요를 누르면 기존에 눌렀는지를 판단할 근거가 필요한데 로그인 기능이 없기 때문에 판단할 방법이 없어서 `LocalStorage`를 이용해서 좋아요/싫어요에 대한 데이터를 기억하도록 만들었습니다.<br />

물론 누군가 `LocalStroage`를 지워버리면 같은 게시글에 중복된 좋아요/싫어요가 가능한 문제가 있지만, 실제 제공할 서비스가 아니기 때문에 큰 문제는 없을 거라고 판단했습니다.<br />

# 🥲 부족한 것들
## 0️⃣ 메모이제이션 함수들의 사용
`useCallback()`, `useMemo()`, `memo()`의 사용에 대한 명확한 기준을 정하기 힘들어서 일단 마구잡이로 사용했습니다.<br />

`useCallback()`, `useMemo()`는 모든 함수와 변수에 필수적으로 사용했고, `memo()`는 `props`가 자주 바뀌지 않을 컴포넌트라고 판단되면 사용했습니다.<br />
( `<Header>`, `<Footer>`, `<GridPosts>`, `<Icon>`에 사용함 )<br />

## 1️⃣ lazy loading
처음에는 지연로딩을 사용하려고 했는데 딱히 쓸만한 곳도 없는 것 같고, 쓴다고 크게 성능에 영향을 미칠거라고 생각이 들지 않아서 사용하지 않았습니다.<br />
( 처음부터 고려하지 않고 마지막에 생각하다 보니 어디에 쓸지 애매한 부분도 있었습니다. )<br />

# 📸 실행 영상
## 0️⃣ 반응형 레이아웃 1
![반응형 레이아웃 1](https://user-images.githubusercontent.com/63289318/216816234-ba332eff-9678-4236-afae-e4631f1fc8f1.gif)
_반응형 레이아웃 1_

## 1️⃣ 반응형 레이아웃 2
![반응형 레이아웃 2](https://user-images.githubusercontent.com/63289318/216816244-3797aa7e-7825-4d41-b21c-a83f9f63bbc4.gif)
_반응형 레이아웃 2_

## 2️⃣ 무한 스크롤링
![무한 스크롤링](https://user-images.githubusercontent.com/63289318/216816220-46126747-a1f7-48ef-9fd3-0a156a99b7b3.gif)
_무한 스크롤링_

## 3️⃣ 스켈레톤 UI
![스켈레톤 UI](https://user-images.githubusercontent.com/63289318/216816186-2293b7d4-7cd1-4426-a937-f7da664864a6.gif)
_스켈레톤 UI_

## 4️⃣ 검색
![검색](https://user-images.githubusercontent.com/63289318/216816215-de5741a1-5bef-41bb-929f-ee5a2d18ea90.gif)
_검색_

## 5️⃣ 게시글 생성
![게시글 생성](https://user-images.githubusercontent.com/63289318/216815910-6e79e03e-6740-4e3a-8deb-6d8bd366c81d.gif)
_게시글 생성_

# 🧐 다른 포스트들
1. [`Redux`](https://1-blue.github.io/posts/Redux/){:target="_blank"}
2. [`React` 스크롤 방향 찾기](https://1-blue.github.io/posts/React-%EC%8A%A4%ED%81%AC%EB%A1%A4-%EB%B0%A9%ED%96%A5/){:target="_blank"}
3. [`React-Router-Dom`의 `replace`](https://1-blue.github.io/posts/React-Router-Dom/){:target="_blank"}
4. [`AWS-S3` - `presignedURL` 사용 방법](https://1-blue.github.io/posts/AWS-S3-presignedURL/){:target="_blank"}
5. [`Node.js` + `TypeScript` 세팅 방법](https://1-blue.github.io/posts/Setting-NodeJs/){:target="_blank"}
6. [`Intersection-Observer-API`와 무한 스크롤링](https://1-blue.github.io/posts/Intersection-Observer-API/){:target="_blank"}
7. [`prisma` 사용법 정리](https://1-blue.github.io/posts/prisma/){:target="_blank"}
8. [`React-ToolKit` + `TypeScript` + `React` 사용 방법](https://1-blue.github.io/posts/Redux-Toolkit/){:target="_blank"}

# 📮 레퍼런스
1. [`React` 배포 방법](https://create-react-app.dev/docs/deployment/){:target="_blank"}