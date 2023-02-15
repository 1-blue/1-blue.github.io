---
layout: post
title: 쿠키와 세션 ( cookie & session )
author: admin
date: 2023-02-11 22:58:00 +900
lastmod: 2023-02-15 16:52:00 +900
sitemap:
  changefreq: monthly
  priority: 0.5
categories: [Common, Cookie & Session]
tags: [Cookie, Session, cookie-parser]
---

> 해당 포스트는 `cookie`와 `session`에 대해 정리한 포스트입니다.<br />
{: .prompt-info}

# 🔐 쿠키와 세션
## 0️⃣ 쿠키 ( Cookie )
주로 웹 서버에 의해 제작되며 클라어언트와 서버간의 인증 목적으로 가장 많이 사용됩니다.<br />

일반적으로 `cookie` 인코딩하고, 최대 크기는 `4KB`, 최대 개수는 20여 개 정도로 한정된다고 합니다.<br />

따로 설정해주지 않아도 서버에서 `cookie`를 보내면 브라우저에 등록되며, `cookie`를 보낸 서버로 응답을 하면 쿠키를 실어서 보냅니다.<br />

+ 쿠키 옵션
  1. `path`: 명시된 경로나 하위 경로에서만 `cookie`에 접근할 수 있음 ( 일반적으로 `/` )
  2. `domain`: `cookie`를 접근할 수 있는 도메인 결정 ( 일반적으로 생성한 도메인 ) ( 값을 입력하면 서브 도메인에서도 접근 가능 )
  3. `expires`: 유효 일자 ( `date.toUTCString()`으로 세팅 )
  4. `max-age`: 만료 기간 ( `ms` )
  5. `secure`: `HTTPS`인 경우에만 `cookie`가 전송됨
  6. `samesite`: `XSRF` 공격을 막기 위함
  7. `httponly`: `JavaScript`로 `cookie`를 읽지 못하게 만듦

만약 지속 시간을 입력하지 않으면 브라우저가 닫힐 때 제거됩니다. ( 세션 쿠키라고 부름 )<br />
( 지속 시간은 `expires`와 `max-age`를 입력하지 않았을 때를 의미합니다. )<br />

## 1️⃣ 세션 ( Session )
세션이란 서버에 클라이언트의 데이터를 저장하는 기술을 의미합니다.<br />
기본적으로 서버측의 메모리에 저장되며 설정에 따라서 `file`, `database`에 저장하기도 합니다.<br />

## 2️⃣ 쿠키와 세션을 사용하는 이유
`HTTP`/`HTTPS`를 이용하면 `stateless` 즉, 상태를 보존하지 않기 때문에 클라이언트가 누구인지 알아볼 수단이 필요합니다.<br />
바로 그 수단으로 `cookie`와 `session`을 사용합니다.<br />

+ 인증 방식
  1. 클라이언트가 로그인 요청을 서버로 보내고 로그인에 성공
  2. 서버에서 해당 유저의 식별자를 담은 `session`을 만들고 `session`을 식별할 식별자를 `cookie`에 담아서 (암호화해서) 클라이언트로 전달
  3. 클라이언트는 받은 `cookie`를 브라우저에 저장했다가 같은 서버에게 요청을 보내는 경우 `cookie`를 같이 실어서 전달
  4. 서버에서 클라이언트에서 받은 `cookie`를 통해서 `session` 식별자를 얻고 `session` 식별자를 이용해서 유저의 식별자를 얻어 로그인한 유저인지 판단 및 로그인한 유저의 정보 흭득

`HTTP`/`HTTPS`가 상태를 유지하지 않기 때문에 `session`을 이용해서 상태를 유지하고, `cookie`를 이용해서 `session`을 식별할 식별자를 클라이언트에게 전달하는 방식으로 유저를 식별합니다.<br />

여기서 `session`은 서버의 메모리, `file`, `DB`(`redis`) 등 여러 가지로 사용될 수 있는 것으로 알고 있습니다.<br />

# 🧐 사용 예시
## 0️⃣ 브라우저에서 쿠키 생성
`;`으로 구분하고 옵션 값들을 넣을 수 있습니다.<br />

```js
document.cookie = "x=10; path=/; max-age=10;"
```

## 1️⃣ 서버에서 쿠키 생성 ( using Express )
프론트는 `http://localhost:5500`, 서버는 `http://localhost:3000`이라고 가정하겠습니다.<br />

### 1. 프론트 코드
```html
<!DOCTYPE html>
<html lang="ko">
<head>
  <meta charset="UTF-8">
  <meta http-equiv="X-UA-Compatible" content="IE=edge">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Document</title>

  <script defer src="./index.js"></script>
</head>
<body>
  <button type="button">click me</button>
  
  <script>
    const $button = document.querySelector("button");

    $button.addEventListener("click", () => {
      fetch("http://localhost:3000", {
        method: "POST",
        // 서버로 응답할 때 쿠키를 같이 보내기 위한 설정
        credentials: "include",
        headers: {
          "Content-Type": "application/json",
        },
      })
        .then((res) => res.json())
        .then(console.log);
    });
  </script>
</body>
</html>
```

### 2. 서버 코드 ( Express )
`(1)`처럼 직접 헤더를 설정해도 되고 미들웨어인 `cors`를 이용하면 간단하게 헤더를 설정할 수 있습니다.<br />
( `cors()`를 이용하면 매우 간단하지만 내부적으로 모든 처리를 하기 때문에 어떤 헤더를 작성하는지 알지 못하기 때문에 명시적으로 이렇게 동작한다는 것을 보여주기 위해서 직접 헤더를 설정하는 코드를 넣었습니다. )<br />

단, 주의해야 할점은 `Access-Control-Allow-Credentials`를 `true`로 사용한다면 `Access-Control-Allow-Origin`는 구체적인 출처를 작성해야합니다. ( `*` 금지 )<br />
( 즉, `Access-Control-Allow-Credentials: true` & `Access-Control-Allow-Origin: *`는 금지 )<br />
( 쿠키를 전달하는 것이 보안적으로 위험하기 때문에 아무 대상에게나 전달하지 못하게 하기 위함인 것 같음 )<br />

`(2)`처럼 직접 헤더를 설정해서 `cookie`를 생성할 수 있습니다.<br />
하지만 `express`를 사용한다면 쉽게 `res.cookie()`를 이용해서 생성하고 옵션도 적용할 수 있습니다.<br />

`(3)`을 적용하면 즉, 서명을 하면 쿠키의 값 앞/뒤에 특이한 값들이 붙습니다.<br />
`s%3A`로 시작하고 `.`뒤에 이상한 문자열이 들어가 있습니다.<br />
특정 서버에서 만든 쿠키임을 증명하기 위해서 서명한 것입니다.<br />
서버에서 `cookieParser()` 미들웨어에 등록한 문자열을 통해서 해당 서버에서 만든 쿠키임을 증명하는 값을 생성해서 쿠키에 첨부해서 보냅니다.<br />
따라서 사용자 임의로 쿠키의 내용을 바꾸면 서버에서는 해당 쿠키가 본인이 만든 쿠키가 아니라고 생각하고 쿠키를 통한 인증이 거부됩니다.<br />

`(4)`를 허용하면 `(3)`에서 지정한 키를 이용해서 서명을 생성합니다.<br />
`(4)`를 허용하고 `(3)`을 작성하지 않으면 오류가 발생합니다.<br />
그리고 `(3)`과 `(4)`를 허용하면 `(5)`를 통해서 쿠키를 확인할 수 있습니다.<br />
( 즉, 서명을 하면 `req.cookies`가 아닌 `req.signedCookies`에서 파싱된 쿠키를 확인할 수 있습니다. )<br />

```js
const express = require("express");
const cors = require("cors");
const morgan = require("morgan");
const cookieParser = require("cookie-parser");

const app = express();

app.use(morgan("dev"));
app.use(cookieParser());
// // (3) 서명된 쿠키 파싱
// app.use(cookieParser("keyboard cat"));

app.options("/", cors({ origin: "http://localhost:5500", credentials: true }));

app.post("/", (req, res) => {
  // (1)
  res.header("Access-Control-Allow-Credentials", true);
  // (1)
  res.header("Access-Control-Allow-Origin", "http://localhost:5500");

  // // (2)
  // res.header("Set-Cookie", "my-cookie=keyboard; path=/; max-age=10;");
  res.cookie("my-cookie", "keyboard", {
    httpOnly: true,
    secure: true,
    sameSite: "none",
    domain: "localhost",
    path: "/",
    maxAge: 1000,
    expires: new Date().toUTCString(),
    // (4) 쿠키 서명
    // signed: true,
  });
  
  console.log("req.headers.cookie >> ", req.headers.cookie);
  console.log("req.cookies >> ", req.cookies);
  // (5)
  console.log("req.signedCookies >> ", req.signedCookies);

  /**
   * 서명 하지 않았다면
   * req.headers.cookie >>  my-cookie=keyboard
   * req.cookies >>  { 'my-cookie': 'keyboard' }
   * req.signedCookies >>  [Object: null prototype] {}
   * 
   * 서명 했다면
   * req.headers.cookie >>  my-cookie=s%3Akeyboard.3fEHCtxZ9k%2BZikEaKheY1Lgs9WN9ob9dSio%2BsEA%2FI6Y
   * req.cookies >>  {}
   * req.signedCookies >>  [Object: null prototype] { 'my-cookie': 'keyboard' }
   */

  res.json({ message: "쿠키 생성 완료!" });
});

app.listen(3000, () => {
  console.log("Running Server :3000");
});
```

# 📮 레퍼런스
1. « Node.js 교과서 4장, 6장 » ( 조현영 지음, 길벗, 2020 ) ( [4장 - cookie와 session](https://thebook.io/080334/0209/){:target="_blank"}, [6장 - cookie-parser](https://thebook.io/080334/0272/){:target="_blank"} )
2. [javascript.info - 쿠키와 document.cookie](https://ko.javascript.info/cookie){:target="_blank"}
3. [inpa - bodyParser / cookieParser 미들웨어 💯 사용법 정리](https://inpa.tistory.com/entry/EXPRESS-%F0%9F%93%9A-bodyParser-cookieParser-%EB%AF%B8%EB%93%A4%EC%9B%A8%EC%96%B4#Express_-_cookie-parser){:target="_blank"}