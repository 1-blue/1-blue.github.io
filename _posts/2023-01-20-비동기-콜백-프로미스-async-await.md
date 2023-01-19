---
layout: post
title: async & (callback | promise | async/await)
author: admin
date: 2023-01-20 00:07:00 +900
lastmod: 2023-01-20 00:07:00 +900
sitemap:
  changefreq: monthly
  priority: 0.5
categories: [FrontEnd, JavaScript]
tags: [asynchronous, callback, promise, async, await]
---

> 해당 포스트는 `JavaScript`의 `Promise`에 대해 정리한 포스트입니다.<br />주관적인 생각을 정리해서 잘못 설명된 부분이 있을 수 있습니다.<br />
{: .prompt-info}

모든 예시에 나오는 `createPromiseTimer()`는 아래의 코드입니다.<br />
여러번 작성하기엔 자리를 너무 차지해서 여기서 먼저 선언하겠습니다.<br />

```js
const createPromiseTimer = (wait, isSuccess, value) =>
  new Promise((resolve, reject) => {
    setTimeout(() => {
      if (isSuccess) {
        resolve("성공 " + value);
      } else {
        reject("실패 " + value);
      }
    }, wait);
  });
```

# ❓ JavaScript에서 비동기는 왜 있을까?
> [동기와 비동기 참고](/posts/동기-비동기-블로킹-논블로킹/){:target="_blank"}
{: .prompt-tip}

아래 내용들을 공부하기 전에 왜 비동기라는 개념이 존재하고 사용하는지에 대해 이해하고 넘어가면 공부의 목적을 잡는데 도움이 됩니다.<br />

제가 이해하고 있는 비동기란 실행 순서에 대한 개념입니다.<br />
코드는 기본적으로 위에서 아래로 좌측에서 우측으로 실행됩니다.<br />
그리고 함수를 만나면 함수 안으로 들어가서 실행하고 그 응답을 기다리죠.<br />

근데 만약 요청을 하고 응답을 받는데 10초가 걸리는 네트워크 요청을 하는 코드가 있다고 가정해보겠습니다.<br />
만약 10초동안 응답을 기다리면 그 동안 자바스크립트 엔진은 아무것도 못하게 됩니다.<br />
마우스를 움직이거나 키보드를 누르는 이벤트조차 할 수가 없게 되죠.<br />
네트워크 요청 같은 실행 시간을 예측할 수 없는 불확실한 요청들을 처리하기 위해서 비동기를 적용합니다.<br />

비동기를 적용하면 엔진이 코드의 응답을 기다릴 필요없이 응답이 오면 그때가서 응답에 대한 적절한 동작을 하도록 코드를 구성할 수 있습니다.<br />
그 대표적인 예시로 `callback`함수 , `promise`가 있죠.<br />
따라서 저희가 비동기를 공부하면 `callback`함수, `promise`, `async/await`을 같이 공부하는 이유입니다.<br />
어차피 연관되는 개념이고 비동기를 사용하려면 반드시 필요한 개념이기 때문이죠.<br />

# 📦 비동기와 callback
[비동기](/posts/동기-비동기-블로킹-논블로킹/#1%EF%B8%8F⃣-비동기--asynchronous-){:target="_blank"} 작업을 수행하는 경우 사용할 수 있는 방법입니다.<br />

대표인 비동기 함수인 `setTimeout()`이 `callback`을 이용해서 비동기를 제어합니다.<br />

```js
// 실행 후 1초뒤에 "타이머"를 콘솔에 출력
setTimeout(() => console.log("타이머"), 1000);
```

만약 타이머에 값을 전달하고 싶다면 [고차함수](/posts/함수/#-고차-함수--higher-order-function-){:target="_blank"}를 이용해서 구현할 수 있습니다.<br />

```js
const createTimer = (wait, value) =>
  setTimeout(() => {
    console.log("타이머 >> ", value);
  }, wait);

// 실행 후 1초뒤에 "타이머 >> 끝"를 콘솔에 출력
createTimer(1000, "끝");
```

이렇게만 보면 콜백 함수를 이용한 비동기 처리는 문제 없는 좋은 방법으로 느껴집니다.<br />
물론 코드의 실행 흐름대로 해석이 되진 않지만 **"나중에 콜백을 이용해서 처리하니까 지금은 신경쓰지 않아도 되는구나"**정도로 생각하고 넘어갈 수 있습니다.<br />

## 0️⃣ 콜백 지옥
하지만 이전 값을 이용해서 새로운 값을 가져와야 하는 경우에 `callback`을 이용하면 콜백 지옥을 경험할 수 있습니다.<br />

```js
const fetchName = () => {
  return "Aatrox";
};
const fetchAge = (name) => {
  if (name === "Aatrox") return 10;

  return null;
};
const fetchAd = (age) => {
  if (age === 10) return 60;

  return null;
};
const fetchSpeed = (ad) => {
  if (ad === 60) return 345;

  return null;
};

const champion = {};

setTimeout(() => {
  champion.name = fetchName();

  setTimeout(() => {
    // 이름을 이용해서 나이를 패치
    champion.age = fetchAge(champion.name);

    setTimeout(() => {
      // 나이를 이용해서 공격력을 패치
      champion.ad = fetchAd(champion.age);

      setTimeout(() => {
        // 공격력을 이용해서 속도를 패치
        champion.speed = fetchSpeed(champion.ad);

        console.log(champion); // { name: 'Aatrox', age: 10, ad: 60, speed: 345 }
      }, 500);
    }, 500);
  }, 500);
}, 500);
```

코드를 따지고 보면 말이 안되긴 하지만 일단 각 데이터를 가져오는데 다른 값을 이용해야한다고 가정하고 예시를 작성했습니다.<br />
이렇게 이전 값을 이용해서 데이터를 불러올 때 콜백 방식을 이용하면 `deps`가 점점 들어가면서 가독성이 매우 안좋아집니다.<br />

따라서 이런 불편함을 해결하고자 `Promise`가 등장했습니다.<br />

# 🕙 비동기와 Promise
`Promise`를 정의하는 가장 올바른 말은 **실행은 바로하되 결과를 원하는 시점에 얻을 수 있는 것**이라고 생각합니다.<br />
즉, 비동기적인 실행은 일단 바로 처리하고 결괏값을 내부적(`[[PromiseResult]]`)으로 가지고 있다가 원하는 시점(`.then()`/`.catch()`)에 얻을 수 있는 방법입니다.<br />

**"여기서 실행은 바로한다."**라는 말의 의미는 동기적이라는 의미이고 즉, `Promise`는 동기적으로 실행된다라고 이해할 수 있습니다.<br />

```js
new Promise((resolve, reject) => {
  console.log("동기적으로 실행");

  resolve("then()을 호출할 때 전달");
});

console.log("밖에서 실행");

/**
 * "동기적으로 실행"
 * "밖에서 실행"
 */
```

위 코드를 실행해보면 동기적으로 실행되는 것을 알 수 있습니다.<br />
아니 근데 그러면 뭔가 이상합니다.<br />
분명 비동기를 처리하기 위해서 `Promise`를 사용한다고 들었는데 동기라고 하니 어지러울 수 있습니다.<br />

`Promise`를 비동기로 사용하기 위해서는 `Promise.prototype.then()`/`Promise.prototype.catch()`를 사용해야 합니다.<br />
그때 비로소 비동기로 실행되게 됩니다.<br />

## 0️⃣ Promise 객체 만드는 방법
글로 설명하기 보다는 바로 예시를 먼저 보여드리고 설명을 작성하겠습니다.<br />
그리고 사용법에 대해서는 [`Promise.prototype.than()`](/posts/비동기-콜백-프로미스-async-await/#0-promiseprototypethanonfulfilled-onrejected)에서 자세히 알아보고 여기서는 `Promise`객체를 만드는 방법에 대해서만 설명하겠습니다.<br />

```js
// 프로미스 객체 만들기
const instance = new Promise((resolve, reject) => {
  // (1) 성공 시 "resolve(전달할 값)"
  resolve(10);

  // (2) 실패 시 "reject(전달할 에러)"
  reject(new Error("에러"))

  // 위처럼 두 가지 경우를 다 실행한다면 먼저 실행된 것만 인식하고 나머지는 무시됩니다.
  // 위의 경우 "resolve()"로 이행됨 즉, 성공으로 이행됨
});
```

기본적으로 `Promise` 생성자 함수는 함수(`executor`) 하나를 인자로 받습니다.<br />
그리고 `executor`는 두 개의 함수를 인자로 받습니다.<br />
이 부분은 명세서에 적힌 약속이라서 외워야합니다.<br />
( 사실 굳이 외우지 않아도 쓰다보면 외워집니다. )<br />

그리고 `executor`가 받는 두 개의 인자는 성공(`(1)`)과 실패(`(2)`)의 경우로 나눠서 실행하는 데 사용하는 함수입니다.<br />
성공과 실패에 대해서는 [프로미스의 세 가지 상태](/posts/비동기-콜백-프로미스-async-await/#1%EF%B8%8F⃣-프로미스의-세-가지-상태)에서 자세히 살펴보겠습니다.<br />

## 1️⃣ 프로미스의 세 가지 상태
> `[[]]`로 감싸져 있는 속성은 내부에 숨겨진 속성이라서 개발자가 직접적으로 접근할 수 없습니다.
{: .prompt-tip}

`Promise`는 세 가지 상태를 내부 프로퍼티(`[[PromiseState]]`)로 갖고 있습니다.<br />

1. 대기(`pending`): 이행하지도, 거부하지도 않은 초기 상태.
2. 이행(`fulfilled`): 연산이 성공적으로 완료됨.
3. 거부(`rejected`): 연산이 실패함.

```js
// Promise를 제외한 다른 속성들은 생략

console.dir(new Promise((resolve, reject) => {}));
/**
 * [[PromiseState]]: "pending"
 * [[PromiseResult]]: undefined
 */

console.dir(new Promise((resolve, reject) => { resolve("성공"); }));
/**
 * [[PromiseState]]: "fulfilled"
 * [[PromiseResult]]: "성공"
 */

console.dir(new Promise((resolve, reject) => { reject("실패"); }));
/**
 * [[PromiseState]]: "rejected"
 * [[PromiseResult]]: "실패"
 */
```

위 세 가지 경우를 브라우저 개발자 도구의 콘솔에 찍어보면 위와 같은 결과를 확인할 수 있습니다.<br />

## 2️⃣ Promise prototype method
### 0. Promise.prototype.than(onFulfilled, onRejected)
`Promise` 객체의 `fulfilled`를 받는 메서드입니다.<br />
`resolve()`를 통해서 전달한 값을 `than(onFulfilled, onRejected)`의 `onFulfilled()`의 첫 번째 인자로 받습니다.<br />

```js
const instance = new Promise((resolve, reject) => {
  resolve("성공");
});

// 중간에 어떤 많은 작업을 수행...

instance.then((res) => {
  console.log(res); // "성공"
});
```

위처럼 사용하면 원하는 시점에 `fulfilled`된 값을 받아서 사용할 수 있습니다.<br />
위의 예시만 보면 그렇게 유용하게 보이지 않습니다.<br />
왜냐하면 비동기적인 코드가 없기 때문이죠.<br />

```js
const instance = new Promise((resolve, reject) => {
  setTimeout(() => {
    resolve("성공");
  }, 1000);
});

let startTime = Date.now();
let index = 0;

// (1) 약 2.5초가 걸리는 동기적인 코드
for (let i = 0; i < 300_000_000_0; i++) {
  index++;
}

console.log("동기적 실행");
console.log((Date.now() - startTime) / 1000 + "초"); // 2 ~ 3초

instance.then((res) => {
  console.log(res); // "성공"
});

/**
 * "동기적 실행"
 * "2.x초"
 * "성공"
 */ 
```

위와 같은 코드를 보면 타이머에 1초를 설정해서 타이머가 끝났음에도 불구하고 동기적으로 약 2.5초가 걸리는 코드(`(1)`)때문에 즉시 실행하지 않습니다.<br />
따라서 비동기적인 코드를 우리가 원하는 시점에 실행해서 원하는 값을 얻을 수 있죠.<br />
( 사실 타이머만 있었어도 똑같은 결과가 나오긴 합니다. 만약 그 이유가 궁금하시다면 [이벤트 루프와 태스크 큐](/posts/이벤트-루프/){:target="_blank"}에 대해서 찾아보시면 좋을 것 같습니다. )<br />

```js
const instance = new Promise((resolve, reject) => {
  reject(new Error("실패!"));
});

// (1)
instance.then(
  (success) => console.log("success >> ", success),
  (error) => console.error("error >>", error)
);

instance
  .then((success) => console.log("success >> ", success))
  // (2)
  .then(null, (error) => console.error("then : error >>", error))
  // (3)
  .catch((error) => console.error("catch : error >>", error));
```

`.then()`의 두 번째 인자를 이용해서 `.catch()` 역할을 대신(`(1)`) 할 수 있습니다.<br />
혹은 `(2)`처럼 사용하면 `.catch()`를 완전히 대체할 수 있습니다.<br />
위처럼 작성하면 `(3)`은 절대 실행되지 못합니다.<br />

### 1. Promise.prototype.catch(onRejected)
`Promise` 객체의 `rejected`를 받을 수 있도록 도와주는 메서드입니다.<br />
`reject()`를 통해서 전달한 에러를 `.catch(onRejected)`의 `onRejected`의 첫 번째 인자로 받습니다.<br />

```js
new Promise((resolve, reject) => {
  // (1)
  reject("에러 던지기");

  // (2)
  reject(new Error("에러 던지기"));
})
  .then((res) => console.log("res >> ", res))
  .catch((error) => console.log("error >> ", error)); // "error >> 에러 던지기"
```

`(1)`과 `(2)` 둘 다 `.catch()`에서 핸들링하게 됩니다.<br />
하지만 `new Error()`를 이용하면 조금 더 구체적으로 에러임을 명시할 수 있기 때문에 `(2)`가 더 좋은 방식인 것 같습니다.

그리고 아래처럼 `reject()` 사용하지 않고 `throw`로 에러를 던져도 `.catch()`에서 에러를 잡게 됩니다.<br />

```js
new Promise((resolve, reject) => {
  throw new Error("reject 없이 에러 던지기");
})
  .then((res) => console.log("res >> ", res))
  .catch((error) => console.log("error >> ", error));
```

아래처럼 `.catch()`에서 `throw` 통해 다시 에러를 던질 수 있습니다.<br />

```js
new Promise((resolve, reject) => {
  reject("에러 체이닝 0");
})
  .then((res) => console.log("res >> ", res))
  .catch((error) => {
    console.error("error0 >> ", error);

    throw Error("에러 체이닝 1");
  })
  .catch((error) => {
    console.log("error1 >> ", error);

    throw Error("에러 체이닝 2");
  })
  .catch((error) => console.log("error2 >> ", error));

/**
 * error0 >> 에러 체이닝 0
 * error1 >> 에러 체이닝 1
 * error2 >> 에러 체이닝 2
 */
```


### 2. Promise.prototype.finally()
`fulfilled`나 `rejected`에 상관없이 항상 실행됩니다.<br />

```js
createPromiseTimer(4, true, "1초")
  .then(console.log)
  .catch(console.error)
  .finally((v) => {
    console.log("항상 실행 >> ", v);
  });

/**
 * "1초"
 * "항상 실행 >> undefined"
 */
```

## 3️⃣ Promise static method
### 0. Promise.all(iterable)
먼저 `fulfilled`한 순서가 아닌 배열에서 넣어준 순서에 맞춰서 결과가 나옵니다.<br />
단, 하나라도 `rejected`가 발생하면 모든 요청이 `.catch()`로 핸들링됩니다.<br />

아래의 예시(`(1)`)의 경우에는 1 ~ 4초의 타이머가 있는데 1초짜리 타이머에서 에러가 발생한다면 나머지를 기다리지 않고 즉시 `.catch()`로 핸들링됩니다.<br />

```js
Promise.all([
  createPromiseTimer(4000, true, "4초"),
  createPromiseTimer(3000, true, "3초"),
  createPromiseTimer(2000, true, "2초"),
  createPromiseTimer(1000, true, "1초"),
])
  .then(console.log)
  .catch(console.error);
/**
 * [ '성공 4초', '성공 3초', '성공 2초', '성공 1초' ]
 */

// (1)
Promise.all([
  createPromiseTimer(4000, true, "4초"),
  createPromiseTimer(3000, true, "3초"),
  createPromiseTimer(2000, true, "2초"),
  createPromiseTimer(1000, false, "1초"),
])
  .then(console.log)
  .catch(console.error);
/**
 * "실패 1초"
 * ( 타이머는 돌아가지만 "Promise.all()"의 작업은 즉시 종료 )
 */
```

### 1. Promise.allSettled(iterable)
`fulfilled`와 `rejected` 여부를 신경쓰지 않고 모든 요청에 대한 처리가 끝날때까지 결과를 기다립니다.<br />
그리고 `fulfilled`과 `rejected`의 여부에 따라 아래(`(1)`)와 같이 결과를 응답해줍니다.<br />

```js
Promise.allSettled([
  createPromiseTimer(4000, true, "4초"),
  createPromiseTimer(3000, false, "3초"),
  createPromiseTimer(2000, true, "2초"),
  createPromiseTimer(1000, false, "1초"),
])
  .then(console.log)
  .catch(console.error);
/**
 * (1)
 * [
 *   { status: 'fulfilled', value: '성공 4초' },
 *   { status: 'rejected', reason: '실패 3초' },
 *   { status: 'fulfilled', value: '성공 2초' },
 *   { status: 'rejected', reason: '실패 1초' },
 * ]
 */
```

### 2. Promise.any()
가장 먼저 `fulfilled`인 `Promise`를 반환합니다.<br />
모두 `rejected`라면 `.catch()`에 핸들링됩니다.<br />

```js
Promise.any([
  createPromiseTimer(4000, true, "4초"),
  createPromiseTimer(3000, false, "3초"),
  createPromiseTimer(2000, true, "2초"),
  createPromiseTimer(1000, false, "1초"),
])
  .then(console.log)
  .catch(console.error);
/**
 * "2초"
 */

Promise.any([
  createPromiseTimer(4000, false, "4초"),
  createPromiseTimer(3000, false, "3초"),
  createPromiseTimer(2000, false, "2초"),
  createPromiseTimer(1000, false, "1초"),
])
  .then(console.log)
  .catch(console.error);
/**
 * [AggregateError: All promises were rejected] {
 *   [errors]: [ '실패 4초', '실패 3초', '실패 2초', '실패 1초' ]
 * }
 */
```

### 3. Promise.race()
가장 먼저 처리되는 `Promise`의 `fulfilled`/`rejected`를 반환합니다.<br />
`fulfilled`과 `rejected` 여부에 상관없이 가장 먼저 처리되는 결과에 맞게 핸들링됩니다.<br />

```js
Promise.race([
  createPromiseTimer(4000, false, "4초"),
  createPromiseTimer(3000, false, "3초"),
  createPromiseTimer(2000, false, "2초"),
  createPromiseTimer(1000, true, "1초"),
])
  .then(console.log)
  .catch(console.error); 
/**
 * "성공 1초"
 */

Promise.race([
  createPromiseTimer(4000, false, "4초"),
  createPromiseTimer(3000, false, "3초"),
  createPromiseTimer(2000, true, "2초"),
  createPromiseTimer(1000, false, "1초"),
])
  .then(console.log)
  .catch(console.error); 
/**
 * "실패 1초"
 */
```

### 4. Promise.resolve()
바로 `fulfilled`인 `Promise`를 반환합니다.<br />

```js
Promise.resolve("성공").then(console.log); // "성공"
```

### 5. Promise.reject()
바로 `rejected`인 `Promise`를 반환합니다.<br />

```js
Promise.reject(new Error("실패")).catch(console.error); // Error: 실패
```

## 4️⃣ Promise chaining
`Promise.prototype.then()`의 리턴 값은 항상 `Promise` 객체가 됩니다. ( 일반적으로 `fulfilled` )<br />

```js
Promise.resolve("🥚 -> ")
  // (1)
  .then((v) => v + "🐤 -> ")
  .then((v) => v + "🐔 -> ")
  .then((v) => v + "🔥 -> ")
  .then((v) => v + "🍗")
  .then(console.log);
/**
 * "🥚 -> 🐤 -> 🐔 -> 🔥 -> 🍗"
 */
```

`(1)`에서는 `[[PromiseState]]: "fulfilled"`면서 `[[PromiseResult]]: "🥚 -> 🐤 ->"`인 `Promise`객체가 리턴됩니다.<br />
따라서 또 다시 `.then(onFulfilled)`으로 연결할 수 있고 `onFulfilled()`의 첫 번째 인자로 `"🥚 -> 🐤 ->"`값이 들어오게 됩니다.<br />
그런 방식을 반복하는 과정을 통해서 `Promise chaining`이 가능하게 됩니다.<br />

# 🪄 비동기와 async / await
## 0️⃣ async
async로 감싼 함수는 `Promise.prototype.then()`과 같이 리턴 값이 자동으로 `Promise` 객체가 됩니다.<br />

```js
const sum = async (x, y) => x + y;

sum(2, 3).then(console.log);
```

## 1️⃣ await
`async`가 적용된 함수내에서만 사용 가능한 키워드입니다.<br />
비동기인 코드를 동기적이게 보이도록 처리하게 도와줍니다.<br />

```js
(async () => {
  const startTime = Date.now();

  try {
    // 1초 대기
    const v1 = await createPromiseTimer(1000, true, "first");
    // 2초 대기
    const v2 = await createPromiseTimer(2000, true, "second");
    // 3초 대기
    const v3 = await createPromiseTimer(3000, true, "third");

    console.log(v1, v2, v3); // "성공 first 성공 second 성공 third"
  } catch (error) {
    console.log(error);
  } finally {
    console.log(Date.now() - startTime); // 약 6000
  }
})();
/**
 * "성공 first 성공 second 성공 third"
 * 6020
 */
```

만약 중간에 `rejected`가 발생한다면 이후 실행을 중지하고 `catch() {}`로 이동하게 됩니다.<br />
```js
(async () => {
  const startTime = Date.now();

  try {
    // 1초 대기
    const v1 = await createPromiseTimer(1000, true, "first");
    // 2초 대기
    const v2 = await createPromiseTimer(2000, false, "second");
    // 3초 대기
    const v3 = await createPromiseTimer(3000, true, "third");

    console.log(v1, v2, v3); // "성공 first 성공 second 성공 third"
  } catch (error) {
    console.log(error);
  } finally {
    console.log(Date.now() - startTime); // 약 3000
  }
})();
/**
 * "실패 second"
 * 3020
 */
```

`async / await`을 살펴보면 느껴지겠지만 `callback`이나 `Promise` 방식보다 더 쉬우면서도 동기적인 코드처럼 동작하게 보입니다.<br />
보통 사람이 코드를 읽는 흐름이 동기적으로 읽기 때문에 `async / await`를 사용하는 게 가독성 측면에서도 좋습니다.<br />

# ✍️ 총 정리
## 0️⃣ callback vs promise vs async/await 비교
아래는 같은 동작을 하는 코드를 세 가지 방법으로 작성해봤습니다.<br />
물론 예시가 조금 억지긴 하지만 이전에 패치했던 데이터를 기반으로 새로운 데이터를 가져오는 경우를 예로 들었습니다.<br />

아래의 세 가지 예시의 동작 시간과 결과는 거의 동일합니다. ( 거의라고 한 이유는 `setTimeout()`이 정확하지 않기 때문 ) <br />

### 1. callback
```js
const fetchName = () => {
  return "Aatrox";
};
const fetchAge = (name) => {
  if (name === "Aatrox") return 10;

  return null;
};
const fetchAd = (age) => {
  if (age === 10) return 60;

  return null;
};
const fetchSpeed = (ad) => {
  if (ad === 60) return 345;

  return null;
};

const champion = {};

setTimeout(() => {
  champion.name = fetchName();

  setTimeout(() => {
    // 이름을 이용해서 나이를 패치
    champion.age = fetchAge(champion.name);

    setTimeout(() => {
      // 나이를 이용해서 공격력을 패치
      champion.ad = fetchAd(champion.age);

      setTimeout(() => {
        // 공격력을 이용해서 속도를 패치
        champion.speed = fetchSpeed(champion.ad);

        console.log(champion);
      }, 500);
    }, 500);
  }, 500);
}, 500);
/**
 * { name: 'Aatrox', age: 10, ad: 60, speed: 345 }
 */
```

### 2. promise
```js
const fetchName = () => new Promise((resolve, reject) => {
  setTimeout(() => {
    resolve("Aatrox")
  }, 500);
});
const fetchAge = (name) => new Promise((resolve, reject) => {
  setTimeout(() => {
    if (name !== "Aatrox") return;

    resolve(10)
  }, 500);
});
const fetchAd = (age) => new Promise((resolve, reject) => {
  setTimeout(() => {
    if (age !== 10) return;

    resolve(60)
  }, 500);
});
const fetchSpeed = (age) => new Promise((resolve, reject) => {
  setTimeout(() => {
    if (ad !== 60) return;

    resolve(345)
  }, 500);
});

const champion = {};

fetchName()
  .then((name) => {
    champion.name = name;

    return fetchAge(name);
  })
  .then((age) => {
    champion.age = age;

    return fetchAd(age);
  })
  .then((ad) => {
    champion.ad = ad;

    return fetchSpeed(ad);
  })
  .then((speed) => {
    champion.speed = speed;

    return champion;
  })
  .then(console.log);
/**
 * { name: 'Aatrox', age: 10, ad: 60, speed: 345 }
 */
```

### 3. async / await
```js
const fetchName = () => new Promise((resolve, reject) => {
  setTimeout(() => {
    resolve("Aatrox")
  }, 500);
});
const fetchAge = (name) => new Promise((resolve, reject) => {
  setTimeout(() => {
    if (name !== "Aatrox") return;

    resolve(10)
  }, 500);
});
const fetchAd = (age) => new Promise((resolve, reject) => {
  setTimeout(() => {
    if (age !== 10) return;

    resolve(60)
  }, 500);
});
const fetchSpeed = (age) => new Promise((resolve, reject) => {
  setTimeout(() => {
    if (ad !== 60) return;

    resolve(345)
  }, 500);
});

(async () => {
    const champion = {};

    const name = await fetchName();
    const age = await fetchAge(name);
    const ad = await fetchAd(age);
    const speed = await fetchSpeed(ad);

    champion.name = name;
    champion.age = age;
    champion.ad = ad;
    champion.speed = speed;

    console.log(champion);
})();

/**
 * { name: 'Aatrox', age: 10, ad: 60, speed: 345 }
 */
```

세 가지 코드를 모두 읽어보면 느껴지겠지만 확실히 `async / await`의 가독성이 가장 뛰어납니다.<br />
그 이유가 바로 동기적인 것처럼 사람이 읽을 수 있기 때문입니다.<br />
실제로 `await`을 만나면 해당 함수를 벗어나서 다른 코드를 실행하지만(비동기적) 코드의 흐름이 동기적인 것처럼 읽어도 전혀 문제가 없어서 쉽게 보인다고 생각합니다.<br />

하지만 무조건 `async / await`이 좋지는 않습니다.<br />
가끔은 `Promise.allSettled()`같은 방법을 사용하는 게 휠씬 효율적인 경우도 있습니다.<br />
그것에 대한 판단은 **비동기 요청들이 서로 연관성이 있는지**를 판단해보면 동시에 요청해도 되는지 혹은 아닌지를 결정할 수 있습니다.<br />

## 1️⃣ Promise를 쓰는 경우
가끔 `async / await`을 두고 `Promise`를 써야하는 상황이 있습니다.<br />

아래 예시는 세 가지 비동기 요청을 하는데 서로는 전혀 연관성이 없고 에러가 나지 않는다고 가정하겠습니다.<br />

+ `async / await`을 이용한 방법 ( 6초 소요 )

```js
(async () => {
  const startTime = Date.now();

  try {
    const result = [];

    result.push(await createPromiseTimer(1000, true, "first"));
    result.push(await createPromiseTimer(2000, true, "second"));
    result.push(await createPromiseTimer(3000, true, "third"));

    console.log(result); // 성공 first 성공 second 성공 third
  } catch (error) {
    console.error(error);
  } finally {
    console.log(Date.now() - startTime); // 약 6000
  }
})();
/**
 * [ '성공 first', '성공 second', '성공 third' ]
 * 6028
 */
```

+ `Promise.allSettled()`를 이용한 방법 ( 3초 소요 )

```js
(() => {
  const startTime = Date.now();

  const result = [];

  Promise.allSettled([
    createPromiseTimer(1000, true, "first"),
    createPromiseTimer(2000, true, "second"),
    createPromiseTimer(3000, true, "third"),
  ])
    .then((res) => {
      res.forEach(({ status, value }) => status === "fulfilled" && result.push(value));

      console.log(result);
    })
    .catch(console.error)
    .finally(() => console.log(Date.now() - startTime));
})();
/**
 * [ '성공 first', '성공 second', '성공 third' ]
 * 3022
 */
```

위 두 가지 방법을 비교해보면 확실한 차이가 보입니다.<br />
`async / await`은 총 6초가 걸리고 `Promise.allSettled()`를 사용하면 3초가 걸리죠.<br />
즉 `async / await`은 비동기를 실행하는 동안 기다리고, `Promise.allSettled()`은 동시에 비동기를 실행해서 최대 시간만큼만 걸리게 됩니다.<br />


# 💡 Tip
혹시 `Promise`가 어떤 내부 동작을 통해서 비동기적으로 실행되는지가 궁금하다면 [이벤트 루프와 태스크 큐](/posts/이벤트-루프/){:target="_blank"}에서 참고하는 레퍼런스들을 읽어보시면 동작에 대한 이해에 도움이 됩니다.<br />

# 📮 레퍼런스
1. [1-blue - 동기와 비동기](/posts/동기-비동기-블로킹-논블로킹/#1%EF%B8%8F⃣-비동기--asynchronous-){:target="_blank"}
2. [1-blue - 고차함수](/posts/함수/#-고차-함수--higher-order-function-){:target="_blank"}
3. [1-blue - 이벤트 루프와 태스크 큐](/posts/이벤트-루프/){:target="_blank"}
4. [Javascript info - 프라미스와 async, await](https://ko.javascript.info/async){:target="_blank"}
5. [MDN - Promise](https://developer.mozilla.org/ko/docs/Web/JavaScript/Reference/Global_Objects/Promise){:target="_blank"}