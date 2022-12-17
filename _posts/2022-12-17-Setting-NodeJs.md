---
title: Node Express + TypeScript 세팅
author: admin
date: 2022-12-17 17:47:00 +900
categories: [Setting, Node.js]
tags: [Node.js, Express, TypeScript, Setting]
---

> 본 포스트는 `Node`의 `Express`에 `TypeScript`를 이용한 백엔드 구축 세팅에 대한 포스팅입니다.
{: .prompt-info }

# 📌 설치
## 1. 필수로 설치할 패키지
```bash
npm i express
npm i -D typescript nodemon @types/node @types/express concurrently
```

1. `typescript`: 타입스크립트를 사용하기 위해 설치 
2. `nodemon`: 코드 변경에 의한 서버 자동 재시작을 위해 설치
3. `@types/node`: `Node.js`에 타입을 적용하기 위해 설치
4. `@types/express`: `Express`에 타입을 적용하기 위해 설치
5. `concurrently`: 여러 명령어를 동시 실행하기 위해 설치

## 2. 추가로 설치하면 좋은 패키지
```bash
# ".env" 파일 즉 환경 변수를 사용하기 위해 설치
npm i dotenv

# "cors" 문제를 쉽게 해결하기 위해 설치
npm i cors
npm i -D @types/cors

# API 요청을 위해 설치
npm i axios

# 이후에 더 설치하면 추가할 예정...
```

# 📌 세팅
## 1. TypeScript 세팅
```bash
# typescript를 설치해야 가능 ( 혹은 글로벌로 설치했다면 가능 )
npx tsc --init
```

명령어를 실행하면 `tsconfig.json`이 생성됩니다.<br />
해당 파일을 `TypeScript`을 세팅하는 파일이므로 원하는 부분이 있다면 수정하시면 됩니다.<br />

## 2. 기본 세팅
### 2.1 app.js ( 진입점 )
```ts
// ##### app.ts #####

// // dotenv를 설치했다면
// import * as dotenv from "dotenv";
// dotenv.config();

import express, { Request, Response } from "express";
// // cors 설치했다면
// import cors from "cors";

// // 각 router
// import movieRouter from "./routes/movie";
// import dramaRouter from "./routes/drama";
// import bookRouter from "./routes/book";

// 모든 에러를 처리하는 핸들러
import { errorHandler } from "./handler";

const app = express();
app.set("port", 3050);

// // "cors"를 설치했다면 ( 개발모드에서는 "http://localhost:3000"에서의 송수신을 허락한다는 의미 )
// const corsOrigin =
//   process.env.NODE_ENV === "development" ? ["http://localhost:3000"] : [""];
// app.use(cors({ credentials: true, origin: corsOrigin }));

// 기본으로 서버 URL에 들어왔을 때 "index.html" 보여주기 ( 이후에 API문서처럼 만들면 좋을 것 같음 )
app.get("/", (req: Request, res: Response) => {
  res.sendFile(__dirname + "/index.html");
});

// // router 연결
// app.use("/api/movie", movieRouter);
// app.use("/api/drama", dramaRouter);
// app.use("/api/book", bookRouter);

// error 처리 핸들러(미들웨어)
app.use(errorHandler);

// express 실행
app.listen(app.get("port"), () => {
  console.log(`${app.get("port")}번 실행중...`);
});
```

### 2.2 type.ts ( 공통으로 사용할 타입을 정의 )
```ts
// ##### type.ts #####

type ApiType = "axios" | "prisma" | "unknown";

/**
 * "Response" 필수 타입을 지정한 타입
 * 모든 응답에서 아래와 같은 규약에 맞게 응답하기위해 사용하는 타입
 */
export type ApiResponse<T> = {
  meta: { ok: boolean; type?: ApiType };
  data: { message: string } & T;
};
/**
 * 에러를 위한 "Response"
 */
export type ApiErrorResponse = ApiResponse<{}>;
```

### 2.3 handler.ts ( 에러 처리 핸들러(미들웨어) )
```ts
// ##### handler.ts #####

// type
import { AxiosError } from "axios";
import type { ErrorRequestHandler, Response } from "express";
import type { ApiErrorResponse } from "../types";

/**
 * 에러 처리 핸들러
 * 에러 발생 시 해당 핸들러(미들웨어)로 에러가 들어옴
 * 
 * ex) 특정 라우터에서 "next(error)"을 사용하면 해당 핸들러의 첫 번째 인자로 error가 그대로 들어옴
 * 따라서 해당 error에 맞게 적절한 응답을 해주면 됨
 * 
 * @param error 에러 객체
 * @param req "express"의 request
 * @param res "express"의 response
 * @param next "express"의 next
 */
export const errorHandler: ErrorRequestHandler = (
  error,
  req,
  res: Response<ApiErrorResponse>,
  next
) => {
  if (error instanceof AxiosError) {
    console.error("axios 에러 >> ", error);

    res.status(500).json({
      meta: { ok: false, type: "axios" },
      data: { message: "API측 문제입니다.\n잠시후에 다시 시도해주세요!" },
    });
  } else {
    console.error("알 수 없는 에러 >> ", error);

    res.status(500).json({
      meta: { ok: false, type: "unknown" },
      data: { message: "서버의 문제입니다.\n잠시후에 다시 시도해주세요!" },
    });
  }
};
```

### 2.4 router.ts ( router 작성 예시 )
```ts
// ##### router 예시 #####

import express from "express";
import axios from "axios";

// type
import type { Request, Response, NextFunction } from "express";
import type { ApiResponse } from "../types";

const router = express.Router();

router.get(
  "/",
  async (
    req: Request<{ a: string }, { b: string }, { c: string }, { d: string }>,
    res: Response<ApiResponse<{ name: string }>>,
    next: NextFunction
  ) => {
    try {
      // const { a } = req.params;
      // req.res?.json({ b: "이것은 뭔지 모르겠음" });
      // const { c } = req.body;
      // const { d } = req.query;

      const { data } = await axios.get<{name: string}>("https://jsonplaceholder.typicode.com/users/1");

      return res.status(200).json({
        meta: { ok: true },
        data: { message: `특정 이름을 찾았습니다.`, name: data.name },
      });
    } catch (error) {
      // 에러를 넘겨주면 이전에 봤던 "errorHandler"로 넘어가서 처리됨
      next(error);
    }
  }
);

export default router;
```

### 2.5 environment.d.ts ( 환경변수 타입 적용 )
```ts
// ##### @types/environment.d.ts #####

namespace NodeJS {
  interface ProcessEnv extends NodeJS.ProcessEnv {
    NODE_ENV: "development" | "production";

    KAKAO_API_URL: string;
    KAKAO_API_KEY: string;
  }
}
```

위와 같이 파일을 생성하기만 하면 `process.env.KAKAO_API_URL`의 자동 완성과 타입을 지원해줍니다.

## 3. package.json 세팅
```json
{
  "scripts": {
    "build": "npx tsc",
    "start": "node ./src/app.js",
    "dev": "concurrently \"npx tsc --watch\" \"nodemon ./src/app.js\""
  },
}
```

`concurrently`를 이용해서 두 개의 명령어를 동시에 실행합니다.<br />
1. `npx tsc --watch`를 이용해서 코드가 변화를 감지하여 `ts` -> `js`로 바꿔줍니다.
2. `nodemon ./src/app.js`을 이용해서 코드가 변화하면 서버가 자동으로 재시작하게 해줍니다.

> `./src/app.js`는 현재 진입점 파일이 존재하는 경로로 작성하면 됩니다.
{: .prompt-warning }