---
title: '[Javascript] 비동기, Promise란 무엇인가'
excerpt: '헷갈리기 쉬운 Promise 객체에 대해 알아봅니다.'


categories:
  - Javascript
tags:
  - [Javascript, JS, js, Promise]

permalink: /javascript/what-is-promise/

toc: true
toc_sticky: true

date: 2022-07-12
last_modified_at: 2022-07-12
---

javascript는 <u>싱글 스레드</u>로 동작합니다. Heap, Call stack, Event Queue, Event Loop 의 흐름에 따라 한 번에 한 가지의 흐름만 실행시킬 수 있습니다. 하지만 javascript에서도 <u>비동기로 흐름을 처리</u>할 수 있는 방법이 있습니다. 그 중 **Promise**에 대해 정리해보려 합니다.

## 📌 비동기 처리란?

**비동기 처리**란 <u>하나의 작업이 종료될 때까지 기다리지 않고, 다음 작업을 진행하는 비순차적 처리 방식</u>을 말합니다. Web API에 해당하는 `setTimeout` 함수, `addEventListener` 함수, `XMLHttpRequest` 객체 등이 비동기 처리를 지원하고 있습니다.

## 📌 비동기 흐름의 실행 순서

비동기적으로 처리되는 코드 내부에서는 실행 순서를 컨트롤하는 것이 쉽지 않습니다. 서버에서 응답을 받은 데이터에 간단한 가공이 필요한 상황이라고 가정해보겠습니다. 비동기식 흐름에서는 서버로 요청을 보내고, 언제 올지 모르는 응답을 기다리지 않고, 다음 코드를 실행하게 됩니다. 이후 언제든 서버에서 응답이 왔을 때, 돌아와서 특정 가공을 하도록 알려주며 흐름을 제어해야 합니다. 

이처럼 <u>비동기 흐름 내에서 실행 순서를 제어하기 위해서</u> **콜백 함수**를 주로 활용하고 있었습니다.

## 📌 콜백 지옥과 Promise의 필요성

비동기 흐름을 제어하기 위해 **콜백 함수**를 주로 사용합니다. 하지만 로직이 복잡해질수록 콜백의 콜백, 콜백의 콜백의 콜백 같은 기괴한 흐름이 생기곤 합니다.

예를 들어 보도록 하겠습니다. 예제에서는 총 2번의 비동기 요청을 보내겠습니다.

1. 로그인 :  `loginUser({ id, password } , onSuccess, onError)`
2. role 가져오기:   `getRoles(user, onSuccess, onError)`

각각의 메소드들은 비동기 처리를 구현하기 위해 `setTimeout` 함수를 사용하여 2초 뒤에 실행됩니다. 

`loginUser` 메소드와 `getRoles` 메소드 모두 첫 번째 파라미터가 특정 조건에 해당하면, 두 번째 파라미터이자 콜백함수인 `onSuccess`를 호출하고, 조건에 해당하지 않는다면 세 번째 파라미터이자 콜백함수인 `onError`를 호출합니다. 

아래는 **"선언"** 부분 입니다.

```javascript
class UserStorage {
  //* 파라미터로 받은 id와 password가 정상이라면 onSuccess 아니라면 onError
  loginUser({ id, password } , onSuccess, onError) {
    setTimeout(() => {
      if (id === 'uhjee' && password === '0000') {
        onSuccess(id);
      } else {
        onError(new Error('not found'));
      }
    }, 2000);
  }
  //* 파라미터 user가 정상이면 onSuccess 아니라면 onError
  getRoles(user, onSuccess, onError) {
    setTimeout(() => {
      if (user === 'uhjee') {
        onSuccess({ name: 'uhjee', role: 'admin' });
      } else {
        onError(new Error('no access'));
      }
    }, 2000);
  }
}
```

아래는 **"호출"** 부분입니다. 다음의 흐름대로 처리됩니다.

1. `loginUser` 메소드를 호출하며, 성공했을 경우 `getRoles` 호출
2. 각각 실패했을 경우, 에러 로그 발생

```javascript
const userStorage = new UserStorage(); // instance 생성

const id = 'uhjee';
const password = '0000';

userStorage.loginUser({ id, password },
  // callback 1! loginUser의 onSuccess
  (user) => {
    userStorage.getRoles(
      user,
      // callback 1-1! getRoles의 onSuccess
      (userWithRole) => {
        console.log(
          `Hello ${userWithRole.name}, you have a ${userWithRole.role} role`,
        );
      },
      // callback 1-2! getRoles의 onError
      (error) => {
        console.log(error);
      },
    );
  },
  // callback 2! loginUser의 onError
  (error) => {
    console.log(error);
  },
```

위와 같이 총 4번의 콜백 함수를 작성해야 합니다. 콜백 함수의 콜백 함수로 작성하기 때문에 코드의 indent가 점점 가운데로 들어갔다가 다시 나오게 됩니다.( `>` 모양)

콜백 함수가 다른 콜백 함수를 갖는 상황이 끊임 없이 이어진다면, 비동기 코드의 호출 부분에서의 코드 가독성이 상당히 떨어지게 됩니다. 하지만 ECMAScript 6부터 **Promise 객체**가 추가되면서 비동기 호출 부분에서의 콜백 지옥 문제가 해결되었습니다. 

## 📌 Promise는 무엇인가?

Promise 객체는 <u>비동기 작업을 실행하고, 그 처리가 끝난 결과(상태)에 따라 어떠한 처리를 실행하겠다는 약속을 제공</u>합니다.

Promise 객체를 **선언부(Provider)**와 **호출부(Consumer)**로 나누어 접근하면 이해가 더욱 쉽습니다. 우선 아래는 **Promise 객체의 선언부**입니다. 

### Promise 객체 선언부 - Provider

```javascript
// Promise 선언부
const promise1 = new Promise((resolve, reject) => {
    // 최초 비동기 처리 코드(excutor)
    if(실패) {
        reject(); // 실패
    }
    resolve();    // 성공
})
```



Promise 객체는 생성자 파라미터로 함수를 받습니다. 이 함수는 최초로 실행되는 비동기 코드입니다. 이 함수의 실행 결과에 따라 `resolve`, `reject` 를 호출하게 됩니다.

executor 함수의 인자로 받는 `resolve`, `reject` 에 대해 이야기하기 위해서는 Promise의 상태에 대해 이해해야 합니다. **Promise의 상태(State)** 는 총 3가지가 있습니다.

1. <u>대기</u> (pending): 초기 상태로써 아직 이행 또는 거부되지 않은 상태
2. <u>이행</u> (fulfilled) : 로직이 성공한 상태
3. <u>거부</u> (rejected): 로직이 실패한 상태

executor의 실행 결과에 따라 아래와 같이 상태가 변경됩니다.

- 로직 성공:  `resolve()` 호출 -> 이행 상태
- 로직 실패:  `reject()` 호출 -> 거부 상태

위와 같이 상태가 대기에서 이행 또는 거부로 변경될 경우, **executor의 비동기 처리가 종료**됩니다. 이후 promise 객체의 상태에 따라 후처리를 담당하는 메소드가 달라집니다. 호출부에서 자세히 살펴볼게요.

### Promise 객체 호출부 - Consumer

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
promise1.then((executor함수의 결과) => {
    // 상태가 '이행'되었을 때의 후 처리 코드
}).catch((e) => {
    // 상태가 '거부'되었을 때의 후 처리 코드
}).finally(() => {
    // 상태가 '이행' 또는 '거부'일 때의 후 처리 코드
})
```

호출부의 코드는 메소드체이닝 형식으로 작성됩니다. (promise 객체의 `then()` 과 `catch()` 메소드 모두 다시 Promise 객체를 반환하기 때문에 가능)

- `then()`: 상태가 **이행**되었을 경우, 후 처리 코드
- `catch()`: 상태가 **거부**되었을 때의 후 처리 코드
- `finally()`: // 상태가 **이행** 또는 **거부**일 때의 후 처리 코드


비동기 요청이 미래에 맞이할 상태를 대변한다.

1. State: resolve 또는 reject 될 때까지 상태를 유지하고 있다.
2. Producer와 Consumer 가 분리된다.

Promise 객체는 생성될 때, executor(callback 함수)를 실행시킨다.

```jsx
'use strict';

// Promise : javscript에서 제공하는 비동기 처리 객체
//    1. State: pending -> fulfilled or rejected 될 때까지
//    2. Producer, Consumer : 둘의 role이 separate 되어있다.

// 1. Producer
// ⚠️ When new Promise is created, the executor runs automatically!!!
const promise = new Promise((resolve, reject) => {
  // e.g. network, read files
  setTimeout(() => {
    resolve('jee');
  }, 3000);
});

// 2. Consumer: then, catch, finally
promise //
  .then((value) => {
    console.log(value);
  })
  .catch((error) => {
    console.log(error);
  })
  .finally(() => {
    console.log('finally!');
  });

// 3. promise chaining(Promise Obj 는 Promise Obj를 return 한다)
const fetchNumber = new Promise((resolve, reject) => {
  setTimeout(() => {
    resolve(1);
  }, 1000);
});

fetchNumber //
  .then((result) => result * 2)
  .then((result) => result * 3)
  .then((result) => {
    return new Promise((resolve, reject) => {
      setTimeout(() => {
        resolve(result - 1);
      }, 2000);
    });
  })
  .then((result) => {
    console.log(result);
  });

// 4.Error handling
// Promise 가 바로 실행되지 않도록 고차함수로 관리
const getHen = () =>
  new Promise((resolve, reject) => {
    setTimeout(() => {
      resolve('hen');
    }, 1000);
  });

const getEgg = (hen) =>
  new Promise((resolve, reject) => {
    setTimeout(() => {
      hen === 'hen' ? resolve('egg') : reject(new Error('error! its not hen'));
    }, 3000);
  });

const cook = (egg) =>
  new Promise((resolve, reject) => {
    setTimeout(() => {
      resolve('nice cooked egg');
    }, 1000);
  });

getHen() //
  // .then((res) => getEgg(res))  // 아래와 같이 생략 가능
  .then(getEgg)
  .catch(new Error('not hen!')) // getEgg 의 catch문 작성 가능
  .then(cook)
  .then((res) => {
    console.log(res);
  });
```

# callback to promise!

### callback 함수로 작성된 async 관리

```javascript
// Callback Hell example
class UserStorage {
  loginUser(id, password, onSuccess, onError) {
    setTimeout(() => {
      if ((id === 'jee' && password === 'haeng') || (id === 'jee' && password === 'jee')) {
        onSuccess(id);
      } else {
        onError(new Error('not found'));
      }
    }, 2000);
  }

  getRoles(user, onSuccess, onError) {
    setTimeout(() => {
      if (user === 'jee') {
        onSuccess({ name: 'jee', role: 'admin' });
      } else {
        onError(new Error('no access'));
      }
    }, 1000);
  }
}

const userStorage = new UserStorage();
const id = prompt('enter your id');
const password = prompt('enter your passrod');
userStorage.loginUser(
  id,
  password,
  // callback 1! loginUser의 onSuccess
  (user) => {
    userStorage.getRoles(
      user,
      // callback 1-1! getRoles의 onSuccess
      (userWithRole) => {
        alert(`Hello ${userWithRole.name}, you have a ${userWithRole.role} role`);
      },
      // callback 1-2! getRoles의 onError
      (error) => {
        console.log(error);
      },
    );
  },
  // callback 2! loginUser의 onError
  (error) => {
    console.log(error);
  },
);
```

### 위 코드를 Promise를 사용해 변경

```jsx
'use strict';

class UserStorage {
  loginUser(id, password) {
    return new Promise((resolve, reject) => {
      setTimeout(() => {
        if ((id === 'je' && password === 'haeng') || (id === 'jee' && password === 'jee')) {
          resolve(id);
        } else {
          reject(new Error('not found!'));
        }
      }, 2000);
    });
  }

  getRoles(userId) {
    return new Promise((resolve, reject) => {
      setTimeout(() => {
        if (userId === 'jee') {
          resolve({ name: 'jee', role: 'admin' });
        } else {
          reject(new Error('no access'));
        }
      }, 3000);
    });
  }
}

const userStorage = new UserStorage();

const id = prompt('이름!');
const password = prompt('비번!');

userStorage //
  .loginUser(id, password)
  .then(userStorage.getRoles)
  .then((userInfo) => {
    console.log(`${userInfo.name}은 ${userInfo.admin}이야!`);
  })
	.catch(console.log);
```



[[출처 ]dream-ellie/learn-javascript](https://github.com/dream-ellie/learn-javascript/blob/master/notes/async/promise.js)

- 선언과 호출의 분리를 기억하자...
- then 내부 콜백함수의 인자:  이전 then 의 return 값 또는 이전 promise 객체의  resolve()의 인자



https://tangoo91.tistory.com/42
