---
layout: post
title: 리덕스 툴킷 ( Redux-Toolkit + TypeScript + React ) 
author: admin
date: 2023-02-17 23:41:00 +900
lastmod: 2023-02-18 22:09:00 +900
sitemap:
  changefreq: monthly
  priority: 0.5
categories: [FrontEnd, Redux]
tags: [React, Redux, Redux-Toolkit, RTK, TypeScript]
---

> 해당 포스트는 `React` + `Redux-Toolkit` + `TypeScript`의 사용법에 대한 포스트입니다.<br />순수 `Redux`나 사용 이유에 대한 내용은 [해당 링크](/posts/Redux/){:target="_blank"}를 참고해주세요!<br />
{:.prompt-info}

# 🗒️ 설치
```bash
npm install react-redux @reduxjs/toolkit

npm install -D @types/react-redux 
```

# 📜 Redux-Toolkit 사용 설명서
## 0️⃣ createAction ( redux-action )
쉽게 `Action`을 만들어주는 헬퍼 함수입니다.<br />

```ts
import { createAction } from "@reduxjs/toolkit";

// 액션 크리에이터 생성
const actionCreator = createAction("MYACTION", (n: number) => ({ payload: n }));

console.log(actionCreator.type); // "MYACTION"
console.log(actionCreator(1)); // {type: 'MYACTION', payload: 1}
```

## 1️⃣ createReducer
쉽게 `Reducer`를 만들어주는 헬퍼 함수입니다.<br />
객체 혹은 함수 형태로 `Reducer`를 정의할 수 있지만, `RTK`에서 사용하기 편한 함수 형태로 예시를 작성하겠습니다.<br />
( `Reducer`를 정상적으로 동작하게 하기 위해서 필요한 다른 것들이 필요해서 일단 다 추가하고 이후에 각각 나눠서 설명하겠습니다. )<br />

`TODO`를 입력하는 간단한 리듀서입니다.<br />
( `TODO`는 `string[]` )<br />

```ts
import { createAction, createReducer } from "@reduxjs/toolkit";
import type { AnyAction, PayloadAction } from "@reduxjs/toolkit";

const TODO = "TODO";

/** "TODO"의 액션 크리에이터 */
export const todoCreator = createAction(TODO, (todo: number) => ({ payload: { todo } }));

/** 초깃값 타입 */
type InitialState = { todos: string[] };

/** 초깃값 */
const initialState: InitialState = { todos: [] };

/** "addCase"가 아니면서 "action"으로 들어오는 "payload"가 정성적인 타입인지 확인 ( 사용자 정의 타입 가드 ) */
const isActionWithStringPayload = (action: AnyAction): action is PayloadAction<string> => typeof action.payload === "string";

/** 리듀서 정의 */
const todoReducer = createReducer(initialState, (builder) => {
  /** todo 추가 */
  builder.addCase(todoCreator, (state, action) => {
    return {
      ...state,
      todos: [...state.todos, action.payload.todo + ""],
    };
  });

  /** "addCase"에 맞지 않으며 작성한 조건("isActionWithStringPayload")에 맞지 않는 액션이 들어오는 경우 처리 */
  builder.addMatcher(isActionWithStringPayload, (state, action) => {
    console.log("매칭 >> ", state, action);

    return { ...state };
  });

  /** "addCase", "addMatcher"에 맞지 않는 경우 처리 */
  builder.addDefaultCase((state, action) => {
    console.log("예외 >> ", state, action);

    return { ...state };
  });
});

export default todoReducer;
```

## 2️⃣ combineReducers
여러 `Reducer`를 합치는 헬퍼 함수입니다.<br />

```ts
import { combineReducers } from "@reduxjs/toolkit";

// 모든 "Reducer"가 있다고 가정
import counterReducer from "./counter";
import userReducer from "./user";
import todoReducer from "./todo";

const rootReducer = combineReducers({
  counter: counterReducer,
  user: userReducer,
  todo: todoReducer,
});

export default rootReducer;
```

## 3️⃣ immer
`immer`는 내부적으로 돌아가기 때문에 아래 예시를 테스트하려면 설치해야 합니다.<br />

원래 `Redux`의 `state`를 변경할 때는 기존 값과 비교를 위해 원본을 수정하는 행위(`(1)`)를 하면 안됩니다.<br />
하지만 `immer`를 사용하면 `(2)`처럼 원본을 수정하는 행위를 해도 내부적으로 불변성을 유지하도록 조작해줍니다.<br />
( `Array.prototype.push` 같은 메서드를 사용해도 됩니다. )<br />

아직 `immer`에 대해 제대로 알지는 못하기 때문에 설명은 여기까지만 적겠습니다.<br />
일단은 내부적으로 사용하기 때문에 `Redux-Toolkit`에서는 원본을 마음대로 변경해도 된다고 생각하면 될 것 같습니다.<br />

```ts
import produce from "immer";

const state = { count: 1 };

// (1)
// state.count += 1;

// (2)
const increase = produce(state, draft => {
  draft.number += 1;
});

console.log(increase); // { count: 2 }
```

## 4️⃣ configureStore + custom middleware
`Reducer`들을 합쳐서 `store`를 만드는 헬퍼 함수입니다.<br />
`(1)`, `(2)`, `(3)` 미들웨어를 등록하는 방법입니다.<br />
하지만 `(1)`, `(2)`를 사용하는 경우 미들웨어를 거치면서 타입 추론에 대한 문제가 발생합니다.<br />
제 기준으로 `(1)`, `(2)` 그리고 `(3)`에 `prepend()`를 사용하면 `createAsyncThunk()`를 `dispatch()`할 때 타입 추론에 대한 오류가 발생합니다.<br />

정확하게 무슨 문제인지는 모르겠지만, [공식 문서](https://redux-toolkit.js.org/api/getDefaultMiddleware#intended-usage){:target="_blank"}에는 `(2)`를 사용하면 미들웨어를 거치는 동안 타입 추론을 못한다고 하네요.<br />
( 일단 저는 `(3)`으로만 사용하는 편입니다. )<br />

그리고 미들웨어를 직접 만들 때는 `Middleware` 타입을 이용하고 아래와 같은 형태로 사용하면 됩니다.<br />

```ts
import { configureStore, MiddlewareArray } from "@reduxjs/toolkit";
import type { Middleware } from "@reduxjs/toolkit";

import rootReducer from "./reducers";

// (4) 커스텀 미들웨어 "logger"
const myLogger: Middleware = (store) => (next) => (action) => {
  console.group("logger");
  console.log("type >> ", action.type);
  console.log("payload >> ", action.payload);
  console.groupEnd();

  return next(action);
};

const store = configureStore({
  reducer: rootReducer,
  // (1)
  // middleware: new MiddlewareArray().prepend([]).concat([]),

  // (2)
  // middleware: [myLogger],

  // (3)
  middleware: (getDefaultMiddleware) => getDefaultMiddleware().concat([myLogger]),
});

export { store };
```

## 5️⃣ createSelector ( reselect )
컴포넌트에서 `Redux`의 데이터를 가져올 때 [`memoization`](/posts/디바운싱-쓰로틀링-메모이제이션/#-메모이제이션--memoization-){:target="_blank"}을 적용해주는 헬퍼 함수입니다.<br />

`useSelector()`는 기본적으로 렌더링마다 실행되기 때문에 최적화를 해주면 성능 향상이 될 수 있습니다.<br />
`createSelector()`를 이용하면 `useSelector(callback)`의 매개변수로 전달되는 값이 바뀌지 않으면 `memoization`한 값을 그대로 사용합니다.<br />

```ts
import { createSelector } from "@reduxjs/toolkit";
import store from "../configureStore";

/** "store"에서 필요한 "state" 선택 */
const getToDos = (myStore: ReturnType<typeof store.getState>) => {
  return myStore.todo;
};

/** 선택한 "state"에서 메모이제이션할 값 선택 ( 캐싱 ) */
export const getToDoList = createSelector(getToDos, (todos) => {
  // 테스트용 ( 정상적으로 동작한다면 즉, 메모이제이션 된다면 초기와 "todos"가 바뀌는 경우를 제외하고 콘솔이 호출되지 않음 )
  console.log("메모이제이션 안됨!!!");

  return todos;
});

// 특정 컴포넌트에서 아래와 같이 사용
const { todos } = useSelector(getToDoList);
```

## 6️⃣ 타입 적용하기 ( dispatch, selector )
일반적인 `useDispatch()`, `useSelector()`를 사용하면 `TypeScript`에서 `Redux`의 타입을 추론하지 못합니다.<br />
따라서 아래와 같은 방법을 적용한 `useAppSelector()`와 `useAppDispatch()`를 사용하면 타입을 적용한 채로 사용할 수 있습니다.<br />

```ts
import { useDispatch, useSelector } from "react-redux";

// store
import store from "@src/store/configureStore";

// type
import type { TypedUseSelectorHook } from "react-redux";

type RootState = ReturnType<typeof store.getState>;
type AppDispatch = typeof store.dispatch;

/**
 * 작성한 타입이 적용된 "useSelector()"
 */
export const useAppSelector: TypedUseSelectorHook<RootState> = useSelector;
/**
 * 작성한 타입이 적용된 "useDispatch()"
 */
export const useAppDispatch: () => AppDispatch = useDispatch;
```

## 7️⃣ createSlice
위에서 공부했던 `createAction`, `createReducer`, `createSelector`, `immer`를 한 번에 처리해주는 헬퍼 함수입니다.<br />

```ts
import { createSlice } from "@reduxjs/toolkit";

// type
import type { PayloadAction } from "@reduxjs/toolkit";

const initialState = {
  count: 0,
};

const counterSlice = createSlice({
  // 액션을 생성하는데 사용
  name: "counter",

  // 초기 상태
  initialState,

  // 리듀서 정의
  reducers: {
    // "PayloadAction"를 이용해서 "payload"타입 지정
    increment(state, action: PayloadAction<number>) {
      state.count += action.payload;
    },
    decrement: (state, action: PayloadAction<number>) => {
      state.count -= action.payload;
    },
  },
});

// 아래 값을 이용해서 "Action"을 생성
// 즉, "dispatch(counterAction.increment(1))"과 같은 형태로 사용
export const counterAction = counterSlice.actions;

// 리듀서 합칠 때 사용
export default counterSlice.reducer;
```

## 8️⃣ createAsyncThunk
`Redux`에서 비동기를 처리할 때 `createAsyncThunk()`를 사용합니다.<br />
`createAsyncThunk<성공반환타입, 매개변수타입, 옵션타입(실패반환타입)>` 형태로 타입을 지정할 수 있습니다.<br />

```ts
import { createAsyncThunk, createSlice } from "@reduxjs/toolkit";

// =================== type ===================
type User = {
  id: number;
  name: string;
  // ... 생략
};

// =================== state ===================
// 초기 상태의 타입
type InitialState = {
  users: User[];
  user: User | null;
};

// 초기 상태
const initialState: InitialState = {
  users: [],
  user: null,
};

// =================== api type ===================
type ApiFetchUsersRequest = {};
type ApiFetchUsersResponse = User[];
type ApiFetchUserRequest = { id: number };
type ApiFetchUserResponse = User;

type ApiFetchUsersHandler = (
  body: ApiFetchUsersRequest
) => Promise<ApiFetchUsersResponse>;
type ApiFetchUserHandler = (
  body: ApiFetchUserRequest
) => Promise<ApiFetchUserResponse>;

// =================== api 요청 함수 ===================
const apiFetchUsers: ApiFetchUsersHandler = async () =>
  (await fetch(`https://jsonplaceholder.typicode.com/users`)).json();
const apiFetchUser: ApiFetchUserHandler = async ({ id }) =>
  (await fetch(`https://jsonplaceholder.typicode.com/users/${id}`)).json();

// "createAsyncThunk"의 예외로 사용할 타입
type CreateAsyncThunkErrorType = { rejectValue: { message: string } };

// =================== thunk ===================
/** 모든 유저들 패치 */
export const usersThunk = createAsyncThunk<
  { users: User[] },
  null,
  CreateAsyncThunkErrorType
>(
  "users",
  async (_, { rejectWithValue }) => {
    try {
      const users = await apiFetchUsers({});

      return { users };
    } catch (error) {
      return rejectWithValue({ message: "유저들 검색 실패" });
    }
  }
);
/** 특정 유저 패치 */
export const userThunk = createAsyncThunk<
  { user: User },
  { id: number },
  CreateAsyncThunkErrorType
>(
  "user",
  async ({ id }: { id: number }, { rejectWithValue }) => {
    try {
      const user = await apiFetchUser({ id });

      return { user };
    } catch (error) {
      console.error(error);

      return rejectWithValue({ message: "유저 검색 실패" });
    }
  }
);

// =================== slice ===================
const userSlice = createSlice({
  name: "user",
  initialState,
  reducers: {
    // 동기적인 처리를 하는 부분
  },
  // 비동기적인 처리를 하는 부분
  extraReducers(builder) {
    // 패치중
    builder.addCase(usersThunk.pending, (state, action) => {
      console.log("유저들 패치중");
    });
    // 패치 성공
    builder.addCase(usersThunk.fulfilled, (state, action) => {
      // "usersThunk()"의 두 번째 인자인 함수에서 반환하는 값이 "action.payload"로 들어옴 ( 타입 추론됨 )
      state.users = action.payload.users;
    });
    // 패치 실패
    builder.addCase(usersThunk.rejected, (state, action) => {
      // "usersThunk()"의 두 번째 인자인 함수에서 "rejectWithValue()"를 사용한 값이 "action.payload"로 들어옴 ( "createAsyncThunk"에서 제네릭을 작성하면 타입 추론됨 )
      console.error("error >> ", action.payload?.message);
    });

    builder.addCase(userThunk.pending, (state, action) => {
      console.log("유저 패치중");
    });
    builder.addCase(userThunk.fulfilled, (state, action) => {
      state.user = action.payload.user;
    });
    builder.addCase(userThunk.rejected, (state, action) => {
      console.error("error >> ", action.payload?.message);
    });
  },
});

export const userAction = userSlice.actions;

export default userSlice.reducer;
```

## 8️⃣ 전체 사용 예시
`0` ~ `5`에서 설명한 내용을 기반으로 `Redux-Toolkit`을 사용한 예시 코드입니다.<br />
현재 `stackblitz`에서는 이상하게 타입 오류가 발생하는데 실제로 `VSCode`에서 그대로 실행하면 정상적으로 동작합니다.<br />
( 오류를 추적해보면 [여기](https://stackblitz.com/edit/react-ts-gpquey?file=index.tsx,src%2Fcomponents%2FApp.tsx,src%2Fstore%2Freducers%2Ftodo.ts,node_modules%2F%40reduxjs%2Ftoolkit%2Fdist%2Findex.d.ts){:target="_blank"}에 도달하는데 `immer`, `reselect`, `redux-thunk`을 찾을 수 없다고 하는데 기본적으로 `RTK`에 내장되어 있는 걸로 알고 있고, 직접 설치해도 오류가 사라지지는 않습니다... 😥 )<br />

<iframe height="600" style="width: 100%;" scrolling="no" src="https://stackblitz.com/edit/react-ts-gpquey?ctl=1&embed=1&file=index.tsx"></iframe>

# 📮 레퍼런스
1. [Redux-Toolkit 공식 문서 - TypeScript Quick Start](https://redux-toolkit.js.org/tutorials/typescript){:target="_blank"}
2. [Redux-Toolkit 공식 문서 - Usage With TypeScript](https://redux-toolkit.js.org/usage/usage-with-typescript){:target="_blank"}
3. [1-blue - `memoization`](/posts/디바운싱-쓰로틀링-메모이제이션/#-메모이제이션--memoization-){:target="_blank"}
4. [stackblitz - Redux-Toolkit 사용 예시](https://stackblitz.com/edit/react-ts-gpquey?devToolsHeight=33&embed=1&file=src/store/configureStore.ts){:target="_blank"}