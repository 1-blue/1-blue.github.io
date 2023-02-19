---
layout: post
title: Tree Shaking
author: admin
date: 2023-02-20 07:44:00 +900
lastmod: 2023-02-20 07:44:00 +900
sitemap:
  changefreq: monthly
  priority: 0.5
categories: [FrontEnd, Util]
tags: [Tree Shaking, Side Effect, 트리쉐이킹, 사이드이펙트]
---

> 해당 포스트는 `Tree Shaking`에 대한 포스트입니다.<br />혹시 `webpack`에 대한 지식이 필요하다면 [webpack](/posts/Webpack/){:target="_blank"}를 참고해주세요!
{: .prompt-info}

# 🍂 Tree Shaking
**사용되지 않는 코드를 제거하기 위해 `JavaScript` 컨텍스트에서 일반적으로 사용되는 용어**라고 `webpack` 공식 문서에 적혀 있습니다.<br />
즉, `import`하고 사용하지 않는 코드는 번들링 후에 제거하는 기술을 의미합니다.<br />

# 🌳 Tree Shaking 예시
아래의 코드를 예시로 설명하겠습니다. ( [`webpack` - `tree-shaking` 참고](https://webpack.kr/guides/tree-shaking/#add-a-utility){:target="_blank"} )<br />

`index.js`에서는 `math.ts`의 모든 `export`를 가져옵니다.<br />
하지만 사용은 `cube()`라는 함수 하나만 사용하고 있습니다.<br />
이런 경우 굳이 `square()`를 번들링에 포함할 필요가 있을까요?<br />

이렇게 사용하지 않는 코드를 번들링에서 제외하는 기술을 `Tree Shaking`이라고 합니다.<br />
다음으로는 어떻게 `Tree Shaking`을 적용하는지에 대해서 알아보겠습니다.<br />

```ts
// math.ts
export function square(x: number) {
  return x * x;
}

export function cube(x: number) {
  return x * x * x;
}

// index.ts
import * as math from "./math";

console.log(math.cube(5));
```

# 🪓 Tree Shaking 적용 방법
## 0️⃣ CommonJS 모듈 사용하지 않기
[`CommonJS` 모듈 시스템](/posts/자바스크립트-완벽-가이드-10장/#1%EF%B8%8F⃣-node의-내보내기와-가져오기){:target="_blank"}이란 `require`과 `module.exports`를 이용해서 파일을 내보내고 가져오는 방식을 의미합니다.<br />
`CommonJS`를 사용하면 `Tree Shaking`을 적용할 수 없다고 합니다.<br />

저희는 `React`를 사용할 때는 항상 [`ES6` 모듈 시스템](/posts/자바스크립트-완벽-가이드-10장/#-es6의-모듈-시스템){:target="_blank"}을 사용합니다.<br />
그렇다면 신경쓰지 않아도 되는 부분이 아니라고 생각할 수 있습니다.<br />

`create-react-app`같은 경우 내부적으로 어떻게 설정되어 있는지 모르겠지만, `webpack`을 직접 설정한다면 대부분 `@babel/preset-env`를 사용할 것입니다.<br />
[`@babel/preset-env`는 기본적으로 `CommonJS`로 변환시켜주는 기능](https://babeljs.io/docs/babel-preset-env#modules){:target="_blank"}을 갖고 있습니다.<br />
따라서 명시적으로 변환시키지 말라는 코드를 작성해줘야 합니다.<br />

```js
// babel.config.js

const presets = [
  [
    "@babel/preset-env",
    {
      modules: false, // CommonJS 모듈 시스템으로 변환하지 않게 하는 속성
    },
  ],
];
const plugins = [];

module.exports = { presets, plugins };
```

## 1️⃣ Side Effect 고려하기
> `SideEffect`란 외부의 데이터에 영향을 끼치는 행위를 의미합니다.<br />
{:.prompt-tip}

아래 예시에서 `func()`는 `Side Effect`가 있는 함수입니다.<br />
외부의 데이터인 `person`을 변경시켰기 때문입니다.<br />

```ts
const person = {
  name: "Aatrox",
};

const func = () => {
  person.age = 26;
};
```

여기서 중요한 것은 `func()`를 사용하지 않았어도 `import`한다면 `Side Effect`가 있는 함수이기 때문에 `Tree Shaking`이 적용되지 않습니다.<br />

해결 방법은 크게 두 가지가 있습니다.<br />
1. `package.json`에 `"sideEffects": false` or `"sideEffects": [특정 파일 경로, ...]` 설정하기
2. `/*#__PURE__*/` 라벨 붙이기

1번은 명시적으로 패키지와 디펜던시들에 `Side Effect`가 발생하지 않는다고 알려주는 것입니다.<br />
2번은 특정 함수에 사용하고 그 함수는 `Side Effect`가 발생하지 않는다고 알려주는 것입니다. ( 실제로 사용해본 적은 없음 )<br />

## 2️⃣ 필요한 것만 import 하기
첫 예시에서 모든 것을 `import`하는 구문을 사용했는데 그것보다는 사용하는 것만 `import`해야 합니다.<br />

```ts
// import * as math from "./math";
import { cube } from "./math";

console.log(cube(5));
```

# 🍁 Tree Shaking 전/후 비교
`Tree Shaking`을 적용하기 전과 후의 코드의 차이는 모르겠지만, 번들링된 `index.ts`의 크기가 적용 후의 코드가 더 적은 것으로 보아하니 정상적으로 적용이 된 것 같습니다.<br />
( 물론 지금은 코드량이 얼마 없어서 큰 차이를 느낄 수는 없습니다. )<br />

## 0️⃣ Tree Shaking 전
```ts
// bundle.js

// ... 나머지 생략

eval("/* harmony import */ var _math__WEBPACK_IMPORTED_MODULE_0__ = __webpack_require__(/*! ./math */ \"./src/math.ts\");\n\nconsole.log(_math__WEBPACK_IMPORTED_MODULE_0__.cube(5));\n\n//# sourceURL=webpack://tree-shaking/./src/index.ts?");

/**
 * 번들링 결과
 * 
 * asset bundle.js 3.65 KiB [emitted] (name: main)
 * runtime modules 396 bytes 2 modules
 * cacheable modules 151 bytes
 *   ./src/index.ts 58 bytes [built] [code generated]
 *   ./src/math.ts 93 bytes [built] [code generated]
 * webpack 5.75.0 compiled successfully in 1134 ms
 * /
```

## 1️⃣ Tree Shaking 후
```ts
// bundle.js

eval("__webpack_require__.r(__webpack_exports__);\n/* harmony import */ var _math__WEBPACK_IMPORTED_MODULE_0__ = __webpack_require__(/*! ./math */ \"./src/math.ts\");\n\nconsole.log((0,_math__WEBPACK_IMPORTED_MODULE_0__.cube)(5));\n\n//# sourceURL=webpack://tree-shaking/./src/index.ts?");

/**
 * 번들링 결과
 * 
 * asset bundle.js 4.19 KiB [compared for emit] (name: main)
 * runtime modules 670 bytes 3 modules
 * cacheable modules 145 bytes
 *   ./src/index.ts 52 bytes [built] [code generated]
 *   ./src/math.ts 93 bytes [built] [code generated]
 * webpack 5.75.0 compiled successfully in 957 ms
 */
```

# 📮 레퍼런스
1. [webpack - `tree-shaking`](https://webpack.kr/guides/tree-shaking/#conclusion){:target="_blank"}
2. [Younkue Choi - Webpack에서 Tree Shaking 적용하기](https://medium.com/naver-fe-platform/webpack%EC%97%90%EC%84%9C-tree-shaking-%EC%A0%81%EC%9A%A9%ED%95%98%EA%B8%B0-1748e0e0c365){:target="_blank"}
3. [ToastUI - 트리 쉐이킹으로 자바스크립트 페이로드 줄이기](https://ui.toast.com/weekly-pick/ko_20180716){:target="_blank"}

4. [1-blue - `webpack`](/posts/Webpack/){:target="_blank"}
5. [1-blue - `CommonJS` 모듈 시스템](/posts/자바스크립트-완벽-가이드-10장/#1%EF%B8%8F⃣-node의-내보내기와-가져오기){:target="_blank"}
6. [1-blue - `ES6` 모듈 시스템](/posts/자바스크립트-완벽-가이드-10장/#-es6의-모듈-시스템){:target="_blank"}
7. [babel - `@babel/preset-env`](https://babeljs.io/docs/babel-preset-env#modules){:target="_blank"}