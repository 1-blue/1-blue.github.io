---
title: React-Router-Dom V6의 replace
author: admin
date: 2022-12-10 19:48:00 +900
lastmod: 2023-02-18 20:27:00 +900
sitemap:
  changefreq: monthly
  priority: 0.5
categories: [FrontEnd, React]
tags: [React-Router-Dom, replace, SPA]
---

> `react-router-dom`의 `replace`에 대한 포스트입니다.<br />`state`, `useLocation()`같은 다른 부분들은 다루지 않습니다.
{: .prompt-info }

# 🪧 react-router-dom
`react-router-dom`가 어떤 역할을 하는지 알기 위해서는 `React`의 특징인 `SPA`에 대한 이해가 필요합니다.

## 0️⃣ SPA
`SinglePageApplication`으로 페이지 이동 대신 `HTML`파일의 내용을 `JS`를 이용해 `DOM`을 변경시키는 것을 말합니다.<br />

일반적인 웹사이트는 각 페이지마다 `HTML`이 존재하고 페이지를 이동할 때마다 `HTML`을 변경합니다.<br />
따라서 페이지를 이동할 때 화면 전부가 지워지고 다시 생성되기 때문에 흰색 화면이 깜빡거리게 보입니다.<br />
하지만 `SPA`인 웹사이트는 `URL`에 따라서 HTML의 내용을 변경시켜주는 것이기 때문에 화면의 전환이 부드럽습니다.<br />

`URL`을 변경하면 새로운 `HTML`을 주는 것이 일반적이지만 `SPA`에서는 다르게 처리해야 하므로 그런 처리를 쉽게 해주는 라이브러리가 `react-router-dom`입니다. ( + `react-router` )

## 1️⃣ replace 기능
웹사이트를 만들다 보면 히스토리에 남기고 싶지 않은 페이지들이 존재합니다.<br />
저의 경우에는 현재 영화/드라마/도서 관련 웹사이트를 만들고 있습니다.<br />
검색 기능을 구현하고 있는데 기본 `URL`은 `/search`에 `QueryString`으로 `/search?title=올빼미&category=movie`같은 형식으로 동작합니다.<br />
이때 초기에 검색어를 입력하지 않았을 때는 `/search`와 같은 형태는 기록에 남기지 않는 것이 더 좋습니다.<br />
아무것도 입력되지 않은 검색을 유저가 뒤로가기로 찾아볼 일은 없기 때문이죠.<br />

이럴 때 `replace`를 사용하면 히스토리에 남기지 않고 페이지 이동이 가능합니다.<br />

## 2️⃣ replace 동작 방식
> 만약 A, B, C라는 페이지가 존재하고, B라는 페이지의 히스토리를 없애야 하는 경우를 예시로 설명하겠습니다.<br />
{: .prompt-info }

처음 사용할 때 동작이 이상하게 하길래 라이브러리 자체의 오류가 있나 의심했습니다.<br />
제가 이해한 방식은 A -> B로 이동하는 경우 A에서 B로 이동하는 링크에 `replace`를 설정해야 B가 히스토리에 남지 않는다고 생각했습니다.<br />
하지만 동작이 묘하게 이상하게 하나씩 늦거나 빠르게 히스토리에서 제거되는 것을 확인했습니다.<br />
그래서 테스트하면서 구글링해보니 A -> B -> C 라면 B에서 C로 이동하는 링크에 `replace`를 설정해야 한다는 것을 알게 되었습니다.<br />
즉, 히스토리를 남기고 싶지 않은 페이지에서 이동할 때 `replace`를 설정해야 한다는 의미입니다.<br />

많은 내용은 없지만 뭐가 문제고 어디가 잘못된 것인지 감을 못 잡아서 시간을 많이 낭비해서 포스트를 작성했습니다.