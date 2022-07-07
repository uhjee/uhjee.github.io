---
title: '[Javascript] 문? 식? 식별자?'

categories:
  - Javascript
tags:
  - [Javascript, JS, js]

permalink: /javascript/statement-expression-identidier/

toc: true
toc_sticky: true

date: 2022-07-07
last_modified_at: 2022-07-07
---

실무에서 코드에 대해 이야기 할 때, 조건문, 반복문, 조건식 등의 표현들에 크게 의미를 두고 말을 하지 않는 것 같았다. 더 이상 혼용하지 않기 위해 이전에 정리해두었던 Statement, Expression, Identifier 에 대해 리마인드할 겸 적어보려 한다.

## 📌 Statement: 문

javascript 엔진에게 주는 <u>힌트</u>. 메모리에 데이터가 남지 않음.

**e.g. 공문, 식문, 제어문(if, while, for 등), 선언문, 단문( `;` ), 중문(`{ }`)**

- 선언문: 메모리 상에 변수를 할당하는 문장

## 📌 Expression: 식

식은 그 자체로 하나의 값을 수렴.

**e.g. 값식, 연산식, 호출식**

- 호출식: javascript는 모든 함수가 호출될 때 값을 반환한다. (`undefined`, `null`)



## 📌 Identifier: 식별자

변수 중 하나. 메모리 주소의 별칭.





---

### 📚 출처

- [코드스피츠 ES6 기초편 강의](https://youtu.be/0j_eGoF8Q98)

