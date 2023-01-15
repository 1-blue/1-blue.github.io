---
title: script의 defer vs async
author: admin
date: 2023-01-05 09:09:00 +900
lastmod: 2023-01-15 13:56:00 +900
sitemap:
  changefreq: monthly
  priority: 0.5
categories: [FrontEnd, JavaScript]
tags: [script, defet, async]
---

> 해당 포스트는 `JavaScript`의 `defer`와 `async`에 대해 정리한 포스트입니다.<br />[javascript info - async vs defer](https://ko.javascript.info/script-async-defer){:target="_blank"}에서 읽은 내용을 정리한 게시글이라서 해당 페이지를 읽는 것을 더 추천드립니다.<br />
{: .prompt-info}


# 📌 defer
백그라운드에서 `JavaScript`를 다운로드합니다. ( `HMTL`과 동시에 불러옴 )<br />
또한 `HTML`이 모두 로드되기 전에 `JavaScript`를 실행하지 않습니다. ( 지연 실행 )<br />
그리고 `DOMContentLoaded` 발생 전에 `JavaScript`가 실행됩니다.<br />
외부 스크립트에만 유효합니다. ( 즉, `src`가 있는 경우에만 유효 )<br />

+ `defer`는 호출 순서 보장함

# 📌 async
`defer`와 마찬가지로 백그라운드에서 실행됩니다.<br />
`HTML`이나 다른 `script`의 실행에 영향을 받지 않고 독립적으로 실행합니다.<br />
또한 여러 `async script`가 있다면 병렬적으로 실행되며 먼저 로드를 완료한 순서로 JS가 실행됩니다.<br />
또한 `DOMContentLoaded`를 기다리지 않고 실행됩니다.<br />
따라서 현재 페이지와 크게 상관 없는 JS를 실행하는 경우 주로 사용됩니다. ( ex) 방문자 카운트, 광고 등 ) <br />

+ `async`는 호출 순서가 보장되지 않음 ( 먼저 로드한 순서대로 실행 )

# 📌 동적 스크립트
`JavaScript`코드로 `<script src="">`를 추가하는 것을 의미합니다.<br />
기본적으로 `async`로 동작합니다.<br />
즉, 순서를 지키지 않고 먼저 로드되면 실행한다는 의미입니다.<br />
하지만 순서를 지키고 싶을 때는 아래 `(2)`처럼 사용하면 됩니다.<br />

```js
// JavaScript info 의 예시입니다.
function loadScript(src) {
  let script = document.createElement('script');
  script.src = src;
  // (2)
  script.async = false;
  document.body.append(script);
}

// async=false이기 때문에 long.js가 먼저 실행됩니다.
loadScript("/article/script-async-defer/long.js");
loadScript("/article/script-async-defer/small.js");
```

# 📮 레퍼런스
1. [javascript info - async vs defer](https://ko.javascript.info/script-async-defer){:target="_blank"}