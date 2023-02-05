---
title: AWS-S3와 presignedURL
author: admin
date: 2022-12-22 16:46:00 +900
lastmod: 2023-02-05 14:29:00 +900
sitemap:
  changefreq: monthly
  priority: 0.5
categories: [Common, presignedURL]
tags: [AWS-S3, presignedURL, 미리 서명된 URL]
image:
  path: https://user-images.githubusercontent.com/63289318/216803071-4f8df029-bdaa-4e4d-b004-f295a3ab4c0f.png
  alt: presignedURL의 동작 방식
---

> 해당 포스트는 `TypeScript`로 `AWS-S3`와 `presignedURL`를 사용하는 방법에 대한 포스트입니다.
{: .prompt-info}

# 🔑 ACCESS_KEY, ACCESS_SECRET_KEY 구하기
두 개의 키들은 `JS`(`TS`)로 `S3`에 접근할 때 식별하기 위해 사용하는 `Key`입니다.<br />

## 0️⃣ root 계정으로 생성 ( 비추천 )
> 관리자 계정으로 키를 생성했다가 키를 탈취당하면 모든 곳의 접근 권한이 생기므로 위험하기 때문에 `IAM`을 통해서 키를 생성하는 것이 더 좋습니다.
{: .prompt-warning}

1. 관리자 계정으로 로그인 후 [여기](https://us-east-1.console.aws.amazon.com/iam/home#/security_credentials$mfa){:target="_blank"} 접근
2. 액세스 키(엑세스 키 ID 및 비밀 액세스 키) 클릭
3. 새 액세스 키 만들기 클릭

바로 새로운 액세스 키가 생성되며 두 가지의 키를 얻을 수 있습니다.<br />
`ACCESS_KEY`는 이후에도 확인할 수 있으나 `ACCESS_SECRET_KEY`는 생성한 시점에만 확인할 수 있습니다.<br />

## 1️⃣ IAM 생성
1. 관리자 계정 혹은 IAM 생성 권한이 있는 계정으로 [여기](https://console.aws.amazon.com/iam){:target="_blank"} 접근
2. 사용자 클릭
3. 사용자 추가 클릭
4. 상황에 맞게 권한을 부여한 사용자 생성 ( 저의 경우에는 `S3`에 대한 모든 권한 부여 )

사용자를 생성하면 즉시 `ACCESS_KEY`, `ACCESS_SECRET_KEY`를 얻을 수 있습니다.<br />
물론 `ACCESS_SECRET_KEY`는 생성한 시점 이후에 확인이 불가능합니다.<br />
또한 콘솔 로그인 권한을 부여했다면 바로 로그인 페이지로 이동하면서 `id`를 기입해주는 링크를 제공받습니다.<br />
해당 링크로 접근하면 편하게 생성한 `IAM`계정으로 로그인할 수 있습니다.<br />

# ✍️ S3 버킷 생성 및 권한 부여
```json
// 버킷 정책 수정
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "Statement1",
            "Effect": "Allow",
            "Principal": "*",
            "Action": "s3:GetObject",
            "Resource": "arn:aws:s3:::버킷명/*"
        }
    ]
}

// CORS 수정
[
    {
        "AllowedHeaders": [
            "*"
        ],
        "AllowedMethods": [
            "HEAD",
            "GET",
            "PUT",
            "POST",
            "DELETE"
        ],
        "AllowedOrigins": [
            "*"
        ],
        "ExposeHeaders": []
    }
]
```

# 🔗 Node.js와 AWS-S3 연결 및 사용 예시
## 0️⃣ 설치
```bash
# aws와 node.js를 연결시키기 위한 패키지
npm i aws-sdk
```

## 1️⃣ 연결 코드
```ts
import AWS from "aws-sdk";

AWS.config.update({
  // 현재 사용중인 region ( "ap-northeast-2" )
  region: process.env.AWS_REGION,
  // IAM에서 얻은 ACCESS_KEY
  accessKeyId: process.env.AWS_ACCESS_KEY,
  // IAM에서 얻은 ACCESS_SECRET_KEY
  secretAccessKey: process.env.AWS_ACCESS_SECRET_KEY,
});

const S3 = new AWS.S3({
  apiVersion: "2012-10-17",
  signatureVersion: "v4",
});
```

## 2️⃣ presignedURL 생성 헬퍼 함수
```ts
// S3에 presignedURL api 요청 수신 타입 ( Server -> S3 )
type ApiFetchPresignedURLRequest = {
  name: string;
};
// S3에 presignedURL api 요청 송신 타입 ( S3 -> Server  )
type ApiFetchPresignedURLResponse = {
  preSignedURL: string;
};
// S3에 presignedURL API 요청 함수 시그니처 ( "S3" )
export type ApiFetchPresignedURLHandler = (body: ApiFetchPresignedURLRequest) => ApiFetchPresignedURLResponse;

/**
 * "이미지.확장자"를 받아서 "경로/이미지_시간.확장자"으로 변경해주는 함수
 * 현재 사용하고 있는 s3 폴더 구조는 `"개발모드"/images/이미지파일.확장자` 형태입니다.
 *
 * @param name "이미지.확장자" 형태로 전송
 * @returns "경로/이미지_시간.확장자" 형태로 반환
 */
const convertS3ImagePath = (name: string) => {
  const [filename, ext] = name.split(".");

  return `${process.env.NODE_ENV}/images/${filename}_${Date.now()}.${ext}`;
};

/**
 * S3의 "preSignedURL"을 생성하는 함수
 * @param name 이미지 이름  ("이미지.확장자" 형태 )
 * @returns "preSignedURL"와 "photoURL"을 반환 ( "photoURL"은 정상적으로 완료 시 이미지 url )
 */
export const getPresignedURL: ApiFetchPresignedURLHandler = ({ name }) => {
  const photoURL = convertS3ImagePath(name);

  // 20초동안 이미지를 업로드할 수 있는 미리 서명된 URL 생성
  const preSignedURL = S3.getSignedUrl("putObject", {
    // 버킷을 생성할 때 지정한 유니크한 이름
    Bucket: process.env.AWS_BUCKET,
    // 경로 + 파일명
    // ( "image"라는 폴더에 "cat1.jpg"로 저장하고 싶다면 -> "images/cat1.jpg" )
    Key: photoURL,
    // presigendURL 유지 시간 ( 초 )
    Expires: 20,
  });

  // 서명된 URL 반환
  return { preSignedURL };
};
```

## 3️⃣ presignedURL 사용 함수
```tsx
type CreateImageRequest = {
  preSignedURL: string;
  file: File;
}
type CreateImageResponse = {}
type CreateImageHandler = (body: CreateImageRequest) => CreateImageResponse;

/**
 * S3에 이미지 생성 요청
 * @param preSignedURL AWS-S3의 presignedURL
 * @param file 입력받은 파일 객체
 * @returns S3에 이미지 생성 요청 Promise ( 네트워크 요청의 응답 데이터는 사용하지 않음 )
 */
const apiCreateImage: CreateImageHandler = async ({ preSignedURL, file }) =>
  axios.put(preSignedURL, file, { headers: { "Content-Type": file.type } });


// S3 이미지 제거 요청 수신 타입
type DeleteImageRequest = {
  name: string;
};

// S3 이미지 제거 요청 송신 타입
type DeleteImageResponse = ApiResponse<{}>;

// S3 이미지 제거 요청 API 함수 시그니처
export type DeleteImageHandler = (
  body: DeleteImageRequest
) => Promise<AxiosResponse<DeleteImageResponse, any>>;

/**
 * S3에 이미지 제거 요청
 * @param name 삭제할 이미지의 이름
 * @returns S3에 이미지 제거 요청 Promise ( 네트워크 요청의 응답 데이터는 사용하지 않음 )
 */
const apiDeleteImage: DeleteImageHandler = async ({ name }) =>
  serverInstance.delete(`/api/image`, { params: { name } });
```

## 4️⃣ Express + React 예시
```ts
// *** Node.js ***
// "http://localhost:3050"이라고 가정
// 해당 라우터 코드

import express from "express";

// util
import { getPresignedURL } from "../utils";

// type
import type { Request, Response, NextFunction } from "express";

/**
 * S3에 presignedURL 요청 수신 타입 ( B -> S3 )
 */
type ApiFetchPresignedURLRequest = {
  name: string;
};
/**
 * S3에 presignedURL 요청 송신 타입 ( S3 -> B )
 */
type ApiFetchPresignedURLResponse = {
  preSignedURL: string;
};

const imageRouter = express.Router();

// S3에 presignedURL 요청
imageRouter.get(
  // 실제로 여기는 "/"만 사용하지만 명시적으로 보여주기 위해 사용
  // 원래는 진입점 파일(app.ts)에서 "app.use("/api/image", imageRouter);" 사용
  "/api/image",
  async (
    req: Request<{}, {}, {}, FetchPresignedURLRequest>,
    res: Response<FetchPresignedURLResponse>,
    next: NextFunction
  ) => {
    try {
      const { name } = req.query;

      // S3에서 선언한 헬퍼 함수
      const { preSignedURL } = getPresignedURL({ name });

      res.json({ preSignedURL });
    } catch (error) {
      next(error);
    }
  }
);

export default imageRouter;
```

```tsx
// *** React ***
// "http://localhost:3000"이라고 가정

import { useRef, useCallback } from "react";
import axios from "axios";

/**
 * presignedURL 요청 송신 타입 ( B -> F )
 */
type GetPresignedURLResponse = {
  preSignedURL: string;
};

const MyComponent = () => {
  const inputRef = useRef<null | HTMLInputElement>(null);

  const uploadImage = useCallback(
    async (e: React.FormEvent<HTMLFormElement>) => {
      e.preventDefault();

      if (!inputRef || !inputRef.current || !inputRef.current.files)
        return alert("이미지를 등록해주세요!");

      try {
        const file = inputRef.current.files[0];

        // 백엔드로 요청을 보내서 "presignedURL" 받기
        const {
          data: { preSignedURL },
        } = await axios.get<GetPresignedURLResponse>("http://localhost:3050/api/image", {
          params: { name: file.name },
        });

        // S3로 이미지 업로드 요청
        await axios.put(preSignedURL, file, {
          headers: { "Content-Type": file.type },
        });

        // 중간에 에러가 없다면 즉, catch로 들어가지 않고 여기까지 왔다면 업로드 성공
      } catch (error) {
        console.error(error);
      }
    },
    []
  );

  return (
    <form onSubmit={uploadImage}>
      <label htmlFor="image">이미지 입력</label>
      <input id="image" type="file" ref={inputRef} />
      <button type="submit">submit</button>
    </form>
  );
};

export default MyComponent;
```

# 🤔 presignedURL을 사용하는 이유
`presignedURL`이란 `미리 서명된 URL`으로 일정 시간동안 유효한 `URL`을 만들어서 이미지를 업로드할 수 있도록 하는 방법입니다.<br />
이 방법을 사용하면 `Key`가 없이 이미지를 등록할 수 있기 때문에 브라우저에서 직접 `AWS-S3`로 이미지 등록 요청을 보낼 수 있습니다.<br />

따라서 `presignedURL`을 사용하는 이유는 보안 키를 숨기고 네트워크 비용을 최소화하면서 이미지를 업로드하기 위해서 입니다.<br />

아래는 이미지를 등록하는 3가지 방법입니다.<br />
결론적으로 `presignedURL`을 사용하는 방법이 가장 안전하고 효율적이라고 생각합니다.<br />

## 0️⃣ 이미지 등록 방법 1
`브라우저 -> S3 -> 브라우저`의 순서로 이미지를 등록하는 방법입니다.<br />
가장 간단하고 이미지를 이미지를 한 번 전송하기 때문에 네트워크 비용이 비교적 적습니다.<br />
하지만 브라우저에서 `AWS`의 `Key`를 가지고 요청해야 하기 때문에 사용자에게 `Key`를 노출당할 수 있습니다.<br />

![서버를 거치지 않는 이미지 등록 방식 이미지](https://user-images.githubusercontent.com/63289318/216803073-e193d131-e118-4c48-9c91-7a1dd35076db.png)
_브라우저 -> S3 -> 브라우저_

## 1️⃣ 이미지 등록 방법 2
`브라우저 -> 서버 -> S3 -> 서버 -> 브라우저`의 순서로 이미지를 등록하는 방법입니다.<br />
이미지를 두 번 전송해야 하기 때문에 네트워크 비용이 비교적 큽니다.<br />
하지만 서버를 통해서 요청하기 때문에 `Key`를 숨길 수 있습니다.<br />

![서버를 거치는 이미지 등록 방식 이미지](https://user-images.githubusercontent.com/63289318/216803074-d14742a6-a51b-43f0-871a-998615f6904f.png)
_브라우저 -> 서버 -> S3 -> 서버 -> 브라우저_

## 2️⃣ 이미지 등록 방법 3 ( presignedURL )
`브라우저 -> 서버 -> S3 -> 서버 -> 브라우저 -> S3 -> 브라우저`의 순서로 이미지를 등록하는 방법입니다.<br />
가장 복잡하고 네트워크의 낭비도 심해보이지만 `Key`를 숨기면서 이미지 전송도 최소화하는 가장 좋은 방법입니다.<br />

![presignedURL 동작 방식 이미지](https://user-images.githubusercontent.com/63289318/216803071-4f8df029-bdaa-4e4d-b004-f295a3ab4c0f.png)
_브라우저 -> 서버 -> S3 -> 서버 -> 브라우저 -> S3 -> 브라우저_