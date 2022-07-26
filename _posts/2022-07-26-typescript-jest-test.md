---
title: '[Typescript, Jest] Typescript를 Jest로 테스트 작성'
excerpt: 'Typescript와 Jest를 사용해 간단한 함수의 테스트를 진행해보았습니다.'

categories:
  - Typescript
tags:
  - [Typescript, test, unit test, jest, ts]

permalink: /typescript/jest-test/

toc: true
toc_sticky: true

date: 2022-07-26
last_modified_at: 2022-07-26
---

이전에 mocha 라이브러리를 사용해 간단한 테스트 코드를 작성해보았던 기억이 있습니다.  
하지만 npm trends에서 test 라이브러리들을 비교해보니, jest가 mocha 보다 많이 사용되고 있었습니다.
jest가 보다 간결한 테스트 코드를 작성할 수 있다는 것이 그 이유인 것 같습니다.  
 마침 typescript 공부를 하며 테스트 코드 작성 연습도 할 겸 jest 공식 문서를 따라 간단히 테스트를 구성해보았습니다.

## 📦 테스트 환경 구축

1. typescript 환경은 구축되어 있다는 전제 하에 추가로 아래 package들을 설치하였습니다.

   ```powershell
    $ npm i -D  jest ts-jest @types/jest
   ```

   - <u>ts-jest</u> : typescript로 작성된 코드를 테스트할 수 있는 jest 트랜스포머
   - <u>@types/jest</u> : jest 라이브러리의 타입 데이터를 갖고 있는 패키지

2. ts-jest 를 초기화 합니다.

   ```powershell
   $ npx ts-jest config:init
   ```

3. 편리한 사용을 위해 package.json의 script에 작성합니다.
   ```json
   // package.json
   ...
   "scripts": {
     // ./test/test.ts 사용
      "test": "jest test"
   },
   ...
   ```

## 📌 테스트 코드 작성

테스트의 대상이 될 `add` 함수와, `range` 함수를 구현해보았습니다.
`add` 는 단순히 입력받은 값을 더해 반환하는 함수이고, `range` 는 첫 번째 인자로부터 두 번째 인자까지 1씩 더해가며 배열로 반환해주는 함수입니다.

```typescript
// ./src/index.ts

export const add = (a: number, b: number): number => a + b;

export const range = (from: number, to: number): number[] =>
  from < to ? [from, ...range(from + 1, to)] : [];
```

다음은 실제 jest 테스트 코드입니다.

```ts
import { sum, range } from '../src/index';

test('[sum] 1 + 2 to equal 3?', () => {
  // toBe() : primitive type을 비교할 때 사용
  expect(sum(1, 2)).toBe(3);
});

test('[range] from 1 to 5 equal [1,2,3,4,5]?', () => {
  // toStrictEqual() : 같은 타입, 같은 구조의 object를 비교할 때 사용
  expect(range(1, 5 + 1)).toStrictEqual([1, 2, 3, 4, 5]);
});
```

실행 결과 콘솔입니다.

![typescript-jest](/assets/images/posts_img/typescript/typescript-jest.png){: .align-center}

## **📚 출처**

- [ts-jest Docs](https://kulshekhar.github.io/ts-jest/docs/)
