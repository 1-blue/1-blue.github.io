---
layout: post
title: react-query에 대한 기본 개념과 사용 예시
author: admin
date: 2023-04-23 19:17:00 +900
lastmod: 2023-04-23 19:17:00 +900
sitemap:
  changefreq: monthly
  priority: 0.5
categories: [FrontEnd, ReactQuery]
tags: [react-query]
---

> 해당 포스트는 [`blegram`](https://github.com/1-blue/blegram){:target="_blank"}이라는 솔로 프로젝트를 만들면서 사용했던 `react-query`에 대한 내용을 기록하는 포스트입니다.<br />
{:.prompt-info}

# 🧨 옵션 값
아래의 옵션 값들은 공통적으로 사용할 수 있는 옵션 값이고, 구체적인 요청에서 다시 설정할 수 있습니다.<br />
( `ChatGPT`가 번역한 설명 )<br />

1. `cacheTime`: 데이터가 캐시되어 유지되는 시간을 결정
2. `staleTime`: 캐시된 데이터가 만료되기 전에 해당 데이터를 사용할 수 있는 시간을 설정
3. `refetchOnWindowFocus`: 윈도우 포커스를 되찾았을 때 자동으로 새로고침 여부를 결정
4. `refetchOnMount`: 컴포넌트가 마운트될 때 새로고침 여부를 결정
5. `refetchInterval`: 데이터를 주기적으로 자동으로 업데이트할 시간 간격을 설정
6. `retry`: 네트워크 요청이 실패했을 때 다시 시도할 횟수를 결정
7. `retryDelay`: 네트워크 요청이 실패한 후 다시 시도하는 시간 간격을 설정
8. `onSuccess`: 요청이 성공한 후 실행할 함수를 설정
9. `onError`: 요청이 실패한 후 실행할 함수를 설정
10. `queryFn`: 데이터를 가져오는 함수를 설정
11. `queryKey`: 쿼리에 대한 고유 식별자를 설정

## 0️⃣ cacheTime vs staleTime
`staleTime`은 데이터가 유효한지 판단하는 시간을 의미합니다.<br />
기본적으로 `0`으로 설정되어 있기 때문에 데이터를 요청하고 받자마자 캐시하지도 않고 최신 정보로 인지하지도 않습니다.<br />

따라서 `staleTime`을 따로 설정하지 않으면 각 컴포넌트에서 요청할 때마다 계속 네트워크 비용이 발생하게 됩니다.<br />

또한 `cacheTime`은 데이터의 저장시간을 의미합니다.<br />
`staleTime`과 같이 기본 값이 `0`으로 설정되어있어서 불러온 데이터를 저장하지 않습니다.<br />
또한 `staleTime` 보다 작은 값을 설정하게 되면 데이터는 유효하지만 버려서 사용할 데이터가 없는 상황이 발생하게 됩니다.<br />
따라서 항상 `cacheTime`을 `staleTime` 보다는 조금 더 길게 적용해야 합니다.<br />

두 개의 시간을 기본 값으로 사용하는 것보단 유지 시간을 지정하는 것이 더 맞는 사용법이라고 생각하기 때문에 실제로 사용할 때는 설정 값을 바꿔주는 것이 좋다고 생각합니다.<br />
또한 두 개의 유지 시간을 무조건 길게 작성하는 것보다는 적당한 기준을 찾아서 상황에 맞게 적용해야 합니다.<br />

## 1️⃣ onSuccess(), onError()
모든 요청에 대한 기본 성공/실패처리를 수행하는 속성입니다.<br />

성공과 실패에 대한 메세지를 받는다면 해당 요청에 대한 토스트 메세지를 전역적으로 설정해두면 편합니다.<br />
모든 성공과 실패에 `onSuccess()`, `onError()`를 등록하기 너무 번거롭기 때문에 전역으로 설정해두고 추가 작업이 필요하면 해당 요청에서만 구체적으로 메서드를 등록하는 방식으로 코드를 구성하는 것이 좋습니다.<br />

저 같은 경우 `axios`, `react-toastify`를 주로 사용하고 서버의 요청에 대한 모든 응답에 `message: string`을 넣어서 주도록 만들기 때문에 모든 요청에 대한 응답에서 `message`를 토스트 메세지로 보여주도록 설정하고 시작하는 편입니다.<br />

# ⚖️ 세팅
## 0️⃣ 설치
`react-query`를 사용하기 위한 패키지 설치입니다.<br />

```bash
npm i react-query

npm i -D @types/react-query
```

## 1️⃣ provider 컴포넌트 및 기본 핸들러 등록
`react-query`의 `Provider` 컴포넌트를 커스터마이징해서 기본적인 성공과 실패의 핸들러를 등록하는 예시 코드입니다.<br />

`axios`를 이용해서 네트워크 요청을 처리하고, `react-toastify`를 이용해서 성공과 실패에 대한 토스트 메시지를 화면에 띄워줍니다.<br />
해당 설정은 기본 설정이기 때문에 각각의 상세한 요청에서 다른 핸들러를 등록해서 덮어씌울 수 있습니다.<br />

+ [`MyReactQueryProvider.tsx`](https://github.com/1-blue/blegram/tree/master/src/providers/MyReactQueryProvider.tsx){:target="_blank"}

```tsx
import { isAxiosError } from "axios";
import { QueryClientProvider, QueryClient } from "react-query";
import { ReactQueryDevtools } from "react-query/devtools";
import { toast } from "react-toastify";

/** 2023/03/30 - 에러 처리 핸들러 - by 1-blue */
const queryErrorHandler = (error: unknown) => {
  let message = "알 수 없는 오류가 발생했습니다.";

  // Axios Error
  if (isAxiosError(error)) {
    message = error.response?.data.message;
  }
  // Error
  else if (error instanceof Error) {
    message = error.message;
  }

  toast.warning(message);
};

/** 2023/03/30 - 성공 처리 핸들러 - by 1-blue */
const querySuccessHandler = (data: unknown) => {
  if (
    typeof data === "object" &&
    data &&
    "message" in data &&
    typeof data.message === "string"
  ) {
    toast.success(data.message);
  }
};

/** 2023/03/30 - 전역 설정 적용 - by 1-blue */
const queryClient = new QueryClient({
  defaultOptions: {
    queries: {
      onError: queryErrorHandler,
      onSuccess: querySuccessHandler,
      staleTime: 1000 * 60 * 10, // 10 분
      cacheTime: 1000 * 60 * 15, // 15 분
    },
    mutations: {
      onError: queryErrorHandler,
      onSuccess: querySuccessHandler,
    },
  },
});

/** 2023/03/26 - "react-query" Provider 적용 - by 1-blue */
const MyReactQueryProvider: React.FC<React.PropsWithChildren> = ({
  children,
}) => (
  <QueryClientProvider client={queryClient}>
    <ReactQueryDevtools initialIsOpen position="top-left" />
    {children}
  </QueryClientProvider>
);

export default MyReactQueryProvider;
```

# 📑 사용 예시
> 코드 예시는 모두 커스텀 훅이고, 구체적인 `type`과 `axios` 코드는 제외하고 작성했습니다.<br />
{:.prompt-info}

## 0️⃣ useQuery(key, fetcher[, options])
> 자세한 인자와 반환 값에 대해서는 [react-query - `useQuery<T>(key, fetcher[, options])`](https://tanstack.com/query/v4/docs/react/reference/useQuery){:target="_blank"}를 참고해주세요!<br />
{:.prompt-info}

+ 매개변수
  1. `key`: 유일한 식별자로 사용 ( 배열도 가능하며, 배열로 작성하지 않아도 배열로 등록됨 )
  1. `fetcher`: 데이터를 패칭하는 함수 ( 반드시 `Promise` 반환 / 예외 처리 )
  1. `options`: 너무 많은 옵셥들이 있어서 설명은 생략하고 필요한 부분은 [공식 문서](https://tanstack.com/query/v4/docs/react/reference/useQuery){:target="_blank"}를 참고해주세요!<br />

+ 타입
  1. `<T>`: `useQuery()`에서 전달한 `fetcher`가 반환하는 타입을 명시하는 부분

아래 코드는 `useQuery()`를 기반으로 만든 커스텀 훅입니다.<br />
굳이 커스텀 훅으로 만든 이유는 많은 컴포넌트에서 사용할 것이라고 생각했기 때문입니다.<br />
`staleTime`을 설정해두면 한 번 패치한 데이터는 유지되고 같은 요청을 보내도 재사용되기 때문에 중복된 네트워크 요청이 발생하지는 않고, 상태 관리 라이브러리처럼 `React`의 데이터 흐름에 관계없이 여러 컴포넌트에서 쉽게 사용할 수 있습니다.<br />
( 추가적으로 아래에서는 `onSuccess`와 `onError`를 빈 값으로 사용했기 때문에 기본 세팅을 무시하게 됩니다. )<br />

+ [`useMe.ts`](https://github.com/1-blue/blegram/tree/master/src/hooks/query/useMe.ts){:target="_blank"}

```ts
import { useQuery } from "react-query";

// api
import { apiServiceMe } from "@src/apis";

// key
import { queryKeys } from ".";

// type
import type { ApiFetchMeResponse } from "@src/types/api";
type UseMeHandler = () => {
  me?: ApiFetchMeResponse["user"];
  isFetchingMe: boolean;
};

/** 2023/03/29 - 로그인한 유저 정보를 얻는 훅 - by 1-blue */
const useMe: UseMeHandler = () => {
  const { data, isLoading } = useQuery<ApiFetchMeResponse>(
    ["user", "me"],
    apiServiceMe.apiFetchMe,
    { onSuccess() {}, onError() {}, retry: false }
  );

  return { me: data?.user, isFetchingMe: isLoading };
};

export default useMe;
```

## 1️⃣ useInfiniteQuery(key, queryFn[, options])
> 자세한 인자와 반환 값에 대해서는 [react-query - `useInfiniteQuery<T>(key, queryFn[, options])`](https://tanstack.com/query/v4/docs/react/reference/useInfiniteQuery){:target="_blank"}를 참고해주세요!<br />
{:.prompt-info}

+ 매개변수
  1. `key`: 유일한 식별자로 사용 ( 배열도 가능하며, 배열로 작성하지 않아도 배열로 등록됨 )
  1. `queryFn`: 다음 페이지의 데이터를 패칭하는 함수이고, 첫 번째 인자에 객체로 `pageParam`로 현재 페이지 값이 들어옴 ( 반드시 `Promise` 반환 / 예외 처리 )
  1. `options`: `getNextPageParam()`에서 반환한 값이 `queryFn`의 `pageParam`로 들어감 ( 다른 옵션들은 [공식 문서](https://tanstack.com/query/v4/docs/react/reference/useInfiniteQuery){:target="_blank"}를 참고해주세요! )

+ 타입
  1. `<T>`: `useInfiniteQuery()`에서 전달한 `queryFn`가 반환하는 타입을 명시하는 부분

+ 반환 값
  1. `fetchNextPage`: 다음 페이지를 패치하는 함수
  1. `hasNextPage`: 다음 페이지가 존재하는지 여부
  1. ... 다른 반환 값들은 [공식 문서](https://tanstack.com/query/v4/docs/react/reference/useInfiniteQuery){:target="_blank"}를 참고해주세요!

`GET` 요청을 하지만 해당 요청으로 페이지네이션이나 무한 스크롤링을 구현해야 하는 경우 `useInfiniteQuery()`를 사용하면 쉽게 데이터를 페이지 단위로 구분해서 패치할 수 있습니다.<br />
`useInfiniteQuery()`의 `data`에 `pages` 별로 데이터 패칭 함수의 데이터들이 `pages`라는 배열에 들어가게 됩니다.<br />

```ts
type Data = {
  // 응답 데이터의 배열 ( 아래 예시를 기준으로 작성하면 ApiFetchPostsResponse의 배열 )
  pages: ApiFetchPostsResponse[],
  // 모든 페이지 매개변수를 포함하는 배열 ( 어떤 데이터인지 모르겠음 )
  pageParams: unknown[],
}
```

+ [`usePosts.ts`](https://github.com/1-blue/blegram/tree/master/src/hooks/query/usePosts.ts){:target="_blank"}

```ts
import { useInfiniteQuery } from "react-query";

// api
import { apiServicePosts } from "@src/apis";

// key
import { queryKeys } from ".";

// type
import type { ApiFetchPostsResponse } from "@src/types/api";
interface Props {
  take: number;
  lastIdx?: number;
}

/** 2023/04/08 - 게시글들을 얻는 훅 - by 1-blue ( 2023/04/10 ) */
const usePosts = ({ take, lastIdx = -1 }: Props) => {
  const { data, fetchNextPage, hasNextPage } =
    useInfiniteQuery<ApiFetchPostsResponse>(
      [queryKeys.post],
      ({ pageParam = lastIdx }) =>
        apiServicePosts.apiFetchPosts({ take, lastIdx: pageParam }),
      {
        getNextPageParam: (lastPage, allPage) =>
          lastPage.posts?.length === take
            ? lastPage.posts[lastPage.posts.length - 1].idx
            : null,
      }
    );

  return { data, fetchNextPage, hasNextPage };
};

export default usePosts;
```

## 2️⃣ useMutation(mutationFn[, options])와 queryClient.setQueryData(queryKey, updater[, options])
> 자세한 인자와 반환 값에 대해서는 [react-query - `useMutation(mutationFn[, options])`](https://tanstack.com/query/v4/docs/react/reference/useMutation){:target="_blank"}와 [react-query - `queryClient.setQueryData(queryKey, updater)`](https://tanstack.com/query/v4/docs/react/reference/QueryClient#queryclientsetquerydata){:target="_blank"}를 참고해주세요!<br />
{:.prompt-info}

+ 매개변수
  1. `mutationFn`: 네트워크 요청을 수행할 함수 ( 반드시 `Promise` 반환 / 예외 처리 )
  1. `options`: `onSuccess()`를 이용해서 변경사항을 반영 ( 다른 옵션들은 [공식 문서](https://tanstack.com/query/v4/docs/react/reference/useMutation){:target="_blank"}를 참고해주세요! )

+ 반환 값
  1. 뮤테이션을 실행할 함수 ( 변경 요청을 수행할 함수 )

`GET`을 제외한 요청 즉, 사용자와 상호작용을 하는 요청에 대한 처리를 하는데 사용합니다.<br />
사용자와 상호작용이라고 한다면 렌더링하는 순간이 아닌 렌더링 이후의 특정 이벤트에 의해서 실행되는 요청들을 의미합니다.<br />
( 추가, 수정, 삭제 ( `POST`, `PATCH`/`PUT`, `DELETE` ) 요청들은 처리하는 용도로 사용 )<br />

`useMutation()`의 두 번째 인자는 옵션 값인데 그 중 `onSuccess()`를 이용해서 요청이 성공했을 경우 수정된 데이터를 반영하도록 처리합니다.<br />
( `queryClient.setQueryData()`를 이용해서 데이터를 추가/수정/삭제 가능 )<br />

`queryClient.setQueryData()`는 기존에 패칭한 데이터를 수정하는 메서드입니다.<br />
데이터를 패칭할 때 사용한 `key`를 이용해서 어떤 데이터인지 식별하고 해당 데이터를 수정할 수 있습니다.<br />
타입을 지정해주지 않으면 타입을 추론하지 못하기 때문에 지정해주는 것이 좋고, `useInfiniteQuery()`를 이용한 데이터라면 `InfiniteData<T>`를 사용해서 타입을 지정해야 합니다.<br />

+ [`useDeletePost.ts`](https://github.com/1-blue/blegram/tree/master/src/hooks/query/useDeletePost.ts){:target="_blank"}

```ts
import { useMutation, useQueryClient } from "react-query";
import { toast } from "react-toastify";

// api
import { apiServicePost } from "@src/apis";

// key
import { queryKeys } from ".";

// type
import type { UseMutateFunction, InfiniteData } from "react-query";
import type {
  ApiDeletePostRequest,
  ApiDeletePostResponse,
  ApiFetchPostsResponse,
} from "@src/types/api";

/** 2023/04/11 - 게시글 제거 훅 ( 서버 ) - by 1-blue */
const useDeletePost = (): UseMutateFunction<
  ApiDeletePostResponse,
  unknown,
  ApiDeletePostRequest,
  unknown
> => {
  const queryClient = useQueryClient();

  const { mutate } = useMutation(apiServicePost.apiDeletePost, {
    onSuccess(data, { idx }, context) {
      queryClient.setQueryData<InfiniteData<ApiFetchPostsResponse> | undefined>(
        [queryKeys.post],
        (prev) =>
          prev && {
            ...prev,
            pages: prev.pages.map((page) => ({
              ...page,
              posts:
                page.posts && page.posts.filter((post) => post.idx !== idx),
            })),
          }
      );

      toast.success(data.message);
    },
  });

  return mutate;
};

export default useDeletePost;
```

# 📮 레퍼런스
1. [Udemy - react-query](https://kmooc.udemy.com/course/react-query-react){:target="_blank"}
1. [react-query - `useQuery<T>(key, fetcher[, options])`](https://tanstack.com/query/v4/docs/react/reference/useQuery){:target="_blank"}
1. [react-query - `useInfiniteQuery<T>(key, queryFn[, options])`](https://tanstack.com/query/v4/docs/react/reference/useInfiniteQuery){:target="_blank"}
1. [react-query - `useMutation(mutationFn[, options])`](https://tanstack.com/query/v4/docs/react/reference/useMutation){:target="_blank"}
2. [react-query - `queryClient.setQueryData(queryKey, updater)`](https://tanstack.com/query/v4/docs/react/reference/QueryClient#queryclientsetquerydata){:target="_blank"}