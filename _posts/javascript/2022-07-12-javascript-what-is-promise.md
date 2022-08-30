---
title: '[Javascript] 비동기 처리를 위한 Promise란 무엇인가'
excerpt: '헷갈리기 쉬운 Promise 객체에 대해 알아봅니다.'

categories:
  - javascript
tags:
  - [Javascript, JS, js, 콜백지옥, Promise, then, catch, Promise.all]

permalink: /javascript/what-is-promise/

toc: true
toc_sticky: true

date: 2022-07-12
last_modified_at: 2022-07-12
---

javascript는 <u>싱글 스레드</u>로 동작합니다. Heap, Call stack, Event Queue, Event Loop 의 흐름에 따라 한 번에 한 가지의 흐름만 실행시킬 수 있습니다. 하지만 이 와중에 javascript는 몇 가지 방법으로 <u>비동기 흐름을 처리</u>를 하곤 합니다. 그 중 **Promise**에 대해 정리해보려 합니다.

## 📌 비동기 처리란?

**비동기 처리**란 <u>하나의 작업이 종료될 때까지 기다리지 않고, 다음 작업을 진행하는 비순차적 처리 방식</u>을 말합니다.

Web API에 해당하는 `setTimeout` 함수, `addEventListener` 함수, `XMLHttpRequest` 객체 등이 비동기 처리를 지원하고 있고, Node.js 환경에서도 여러가지 비동기 함수들을 지원하고 있습니다.

이전에는 <u>비동기 흐름 내에서 실행 순서를 제어하기 위해서</u> **콜백 함수**를 주로 활용하고 있었습니다.

## 📌 콜백 지옥과 Promise의 필요성

비동기 흐름을 제어하기 위해 **콜백 함수**를 주로 사용합니다. 하지만 로직이 복잡해질수록 콜백의 콜백, 콜백의 콜백의 콜백 같은 기괴한 흐름이 생기곤 합니다.

예를 들어 보도록 하겠습니다. 예제에서는 총 3번의 비동기 요청을 보내겠습니다.

1. 과일이 맞는지 확인
1. 사과가 맞는지 확인
1. 빨간색인지 확인

각각의 메소드들은 비동기 처리를 구현하기 위해 `setTimeout` 함수를 사용하여 1초 뒤에 실행됩니다.

아래는 **"선언"** 부분 입니다.

```javascript
// Provider
const isFruit = (fruit, callback1, callback2) => {
  setTimeout(() => {
    if (['apple', 'orange', 'banana'].includes(fruit.name)) {
      callback1(fruit);
    } else {
      callback2();
    }
  }, 1000);
};

const isApple = (fruit, callback1, callback2) => {
  setTimeout(() => {
    if (fruit.name === 'apple') {
      callback1(fruit);
    } else {
      callback2();
    }
  }, 1000);
};

const isRedApple = (color, callback1, callback2) => {
  setTimeout(() => {
    if (color === 'red') {
      callback1(color);
    } else {
      callback2();
    }
  }, 1000);
};
```

아래는 **"호출"** 부분입니다. 다음의 흐름대로 처리됩니다.

```javascript
// Consumer
const apple = {
  name: 'apple',
  color: 'black',
};

isFruit(
  apple,
  // callback 1-1
  fruit => {
    console.log('과일이라고 합니다.');
    isApple(
      fruit,
      // callback 2-1
      apple => {
        console.log('사과라고 합니다.');
        isRedApple(
          apple.color,
          // callback 3-1
          color => {
            console.log(`${color} 사과가 맞아요.`);
          },
          // callback 3-2
          () => {
            console.log('빨간 색이 아니에요.');
          },
        );
      },
      // callback 2-2
      () => {
        console.log('사과가 아니래요.');
      },
    );
  },
  // callback 1-2
  () => {
    console.log('과일이 아니래요.');
  },
);
```

위와 같이 총 6번의 콜백 함수를 작성해야 합니다. 콜백 함수의 콜백 함수로 작성하기 때문에 코드의 indent가 점점 가운데로 들어갔다가 다시 나오게 됩니다.( `>` 모양) 또 메인이 되는 로직이 오른쪽 안쪽으로 밀려나게 되어 가독성이 떨어집니다.

이처럼 콜백 함수가 다른 콜백 함수를 갖는 상황이 끊임 없이 이어진다면, 호출 부분에서의 가독성이 상당히 떨어지게 됩니다. 하지만 ECMAScript 6부터 **Promise 객체**가 추가되면서 비동기 호출 부분에서의 콜백 지옥 문제가 해결되었습니다.

## 📌 Promise는 무엇인가?

Promise 객체는 <u>비동기 작업을 실행하고, 그 처리가 끝난 결과(상태)에 따라 어떠한 처리를 실행하겠다는 약속을 제공</u>합니다.

Promise 객체를 **선언부(Provider)**와 **호출부(Consumer)**로 나누어 접근하면 이해하기 더욱 쉽습니다. 우선 아래는 **Promise 객체의 선언부**입니다.

### 선언부 - Provider

```javascript
// Promise 선언부
const promise1 = new Promise((resolve, reject) => {
  // 최초 비동기 처리 코드(excutor)
  if (실패) {
    reject(); // 실패
  }
  resolve(); // 성공
});
```

Promise 객체는 생성자 파라미터로 함수를 받습니다. 이 함수는 최초로 실행되는 비동기 코드입니다. 이 함수의 실행 결과에 따라 `resolve`, `reject` 를 호출하게 됩니다.

executor 함수의 인자로 받는 `resolve`, `reject` 에 대해 이야기하기 위해서는 Promise의 상태에 대해 이해해야 합니다. **Promise의 상태(State)** 는 총 3가지가 있습니다.

1. <u>대기</u> (pending): 초기 상태로써 아직 이행 또는 거부되지 않은 상태
2. <u>이행</u> (fulfilled) : 로직이 성공한 상태
3. <u>거부</u> (rejected): 로직이 실패한 상태

executor의 실행 결과에 따라 `resolve`, `reject` 함수를 호출하고, 아래와 같이 상태가 변경됩니다. `resolve` 함수와 `reject` 함수는 각각 다음 상태에 실행될 함수에 데이터를 전달할 수 있습니다. 이 데이터는 각각 `then`과 `catch` 콜백함수의 파라미터로 전달됩니다.

- 로직 성공: `resolve(then 함수 인자로 전달할 데이터)` 호출 -> 이행 상태
- 로직 실패: `reject(catch문 함수 인자로 전달할 데이터)` 호출 -> 거부 상태

위와 같이 상태가 대기에서 이행 또는 거부로 변경될 경우, **executor의 비동기 처리가 종료**됩니다. 이후 promise 객체의 상태에 따라 후처리를 담당하는 메소드가 달라집니다. 아래 호출부에서 자세히 살펴볼게요.

### 호출부 - Consumer

```javascript
// Promise 선언부
const promise1 = new Promise((resolve, reject) => {
    // 최초 비동기 처리 코드(excutor)
    if(실패) {
        reject(); // 실패
    }
    resolve();    // 성공
})

// Promise 호출부
promise1.then((resolve함수의 파라미터) => {
    // 상태가 '이행'되었을 때의 후 처리 코드
}).catch((e) => {
    // 상태가 '거부'되었을 때의 후 처리 코드
}).finally(() => {
    // 상태가 '이행' 또는 '거부'일 때의 후 처리 코드
})
```

호출부의 코드는 메소드체이닝 형식으로 작성됩니다. (promise 객체의 `then()` 과 `catch()` 메소드 모두 다시 Promise 객체를 반환하기 때문에 가능)

- `then()`: 상태가 **이행**되었을 경우, 후 처리 코드 / `then()`은 인자로 함수를 받는데, 이 함수는 resolve() 함수의 파라미터를 인자로 받습니다. 따라서 상태가 변경되기 전의 데이터를 받아서 사용할 수 있습니다.

  > 두 번째 인자로 rejected 되었을 때의 콜백함수를 받을 수 있습니다. 이 경우 아래의 `catch()`문 대신 '거부' 상태의 처리를 진행할 수 있습니다.

- `catch()`: 상태가 **거부**되었을 때의 후 처리 코드

- `finally()`: 상태가 **이행** 또는 **거부**일 때의 후 처리 코드

이를 활용해 기존 콜백 함수로 관리하던 코드를 Promise로 변경해보도록 하겠습니다.

```js
// 선언부
const isFruit = fruit => {
  return new Promise((resolve, reject) => {
    if (['apple', 'orange', 'banana'].includes(fruit.name)) {
      resolve(fruit);
    } else {
      reject('이건 과일이 아니에요.');
    }
  });
};
const isApple = fruit => {
  return new Promise((resolve, reject) => {
    if (fruit.name === 'apple') {
      resolve(fruit.color);
    } else {
      reject('이건 사과가 아니에요.');
    }
  });
};
const isRedApple = color => {
  return new Promise((resolve, reject) => {
    if (color === 'red') {
      resolve();
    } else {
      reject('이건 빨간 사과가 아니에요.');
    }
  });
};
```

```js
// 호출부
const apple = {
  name: 'apple',
  color: 'red',
};

isFruit(apple)
  .then(fruit => {
    console.log('일단 과일은 맞습니다.');
    return isApple(fruit); // promise 객체를 return!
  })
  .then(color => {
    console.log('일단 사과는 맞습니다.');
    return isRedApple(color); // promise 객체를 return!
  })
  .then(() => {
    console.log('빨간 사과가 맞습니다.');
  })
  .catch(e => {
    console.log(e.message);
  });
```

제가 보기에는 확실히 깔끔해진 것 같습니다. 코드를 더 정확하게 이해할 수 있고, `catch`에서 한 번에 에러를 처리할 수 있습니다.

호출부에서 주의해야할 점은 다음 `then` 의 콜백 함수에게 데이터를 전달하기 위해서는 그전 `then` 에서 꼭 promise 객체를 <u>리턴</u>해야 한다는 점입니다. 리턴하지 않는다면 다음 `then` 절로 데이터를 전달할 수 없습니다.

추후 `async/await`문을 활용하면 보다 직관적으로 코드를 개선할 수 있습니다.

## 📌 다수의 Promise를 한 번에 호출하는 방법 - `Promise.all`

호출부분에서 `Promise.all()` 을 활용하면 간단하게 여러 개의 Promise 객체를 실행할 수 있습니다.

```js
// * 선언부
// 바로 resolve 결과 반환
const promise1 = Promise.resolve('promise1의 결과');

const isApple = fruit =>
  new Promise(resolve =>
    resolve(fruit.name === 'apple' ? '사과야.' : '사과 아니야.'),
  );

const isGrape = fruit =>
  new Promise(resolve =>
    resolve(fruit.name === 'grape' ? '포도야.' : '포도 아니야.'),
  );

// * 호출부
const apple = {
  name: 'apple',
};

Promise.all([promise1, isApple(apple), isGrape(apple)])
  .then(([res1, res2, res3]) => {
    console.log({
      res1,
      res2,
      res3,
    });
  })
  .catch(e => {
    console.dir(e);
  });
```

Prototype 메소드인 `Promise.all`은 promise 객체의 배열을 인자로 받습니다. 배열 안의 모든 promise 객체가 resolve될 때까지 기다렸다가 `then`으로 넘어가게 됩니다.

`then`의 인자로 배열로 받게 되는데, 이는 각각의 promise 결과값의 배열입니다. promise 중 하나라도 `reject`가 된다면 `catch`문으로 넘어가게 됩니다.

또 자주 사용되진 않지 않기 때문에 이 글에서는 다루지 않지만, 가장 빨리 처리된 promise 결과를 가져오는 `Promise.race()` 도 있습니다..

---

Promise 객체를 잘 활용하면 싱글 스레드로 동작하는 javascript 환경에서 대기 시간을 줄이고, 자원을 효율적으로 활용할 수 있습니다. 콜백 지옥과 달리 코드 자체로도 가독성이 상당히 좋아졌죠. 놀랍게도 이보다 더 직관적으로 처리하는 **async/await 구문**이 ES2017에서 추가되었습니다.

또 최근에 발표된 ES2022에서는 `await`문을 `async` 없이 전역적으로 사용할 수 있도록 변경되었다고 하는데요. 다음에 **async/await 구문**에 대해 자세히 다뤄보도록 하겠습니다.

## **📚 출처**

- [dream-ellie/learn-javascript](https://github.com/dream-ellie/learn-javascript/blob/master/notes/async/promise.js)

- [자바스크립트 비동기 처리와 AJAX, promise](https://tangoo91.tistory.com/42)
