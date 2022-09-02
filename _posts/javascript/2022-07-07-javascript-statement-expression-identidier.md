---
title: '[Javascript] Statement? Expression? Identifier?'
excerpt: '문? 식? 식별자? 혼용하던 개념을 살펴봅니다.'


categories:
  - javascript
tags:
  - [Javascript, JS, js]

permalink: /javascript/statement-expression-identidier/

toc: true
toc_sticky: true

date: 2022-07-07
last_modified_at: 2022-07-07
---

 실무에서 코드에 대해 이야기 할 때, '조건문, 반복문, 조건식' 등의 표현들을 구분하며 이야기하는 것처럼 느껴지지 않았습니다. 더 이상 혼용하지 않기 위해서 <u>Statement</u>, <u>Expression</u>, <u>Identifier</u> 에 대해 리마인드할 겸 적어보려 합니다.

## <strong>#</strong> Statement: 문


**Statement**는 '진술', '서술'이란 뜻으로, 실행가능한(executable) 최소한의 독립적인 코드 조각을 말합니다. javascript 엔진(인터프리터, 다른 언어에서는 주로 컴파일러)에게 주는 <u>힌트</u>로 메모리에 데이터가 남지 않는다는 점이 특징입니다..  문법적으로 해당 언어에 적합한 모든 코드와 블록은 statement라고 볼 수 있습니다. 따라서 statement는 주로 한 개 이상의 expression과 프로그래밍 키워드를 포함한 경우가 많습니다.<br>

> **e.g. 공문, 식문, 제어문(if, while, for 등), 선언문, 단문( `;` ), 중문(`{}`)**
>
> - 선언문: 메모리 상에 변수를 할당하는 문장

```js
let a = 0;

function test() {}

if (true) {
  // ...
}

for (let i=0; i<10; i++) {
  // ...
}
```



**Sync Flow**는 명령어가 메모리에 적재되어 순차적으로 수행되는 흐름을 말하며, runtime에서 프로그래머는 저 흐름에 간섭할 수 없습니다. 이러한 흐름을 미리 제어하기 위해 제어문 등을 사용합니다.

## <strong>#</strong> Expression: 식

**Expression**은 '수식'이라는 뜻으로, 그 자체가 <u>하나의 값을 수렴(reduce)되어 표현</u>될 수 있는 코드를 말합니다. 즉 Expression은 평가(evaluate)가 가능하기 때문에 하나의 '값'으로 반환될 수 있다는 것을 의미합니다.<br/>

> **e.g. 값식, 연산식, 호출식, JSX**

```js
// 아래의 식은 모두 하나의 값을 반환합니다.
10;
1 + 2 + 3; 	
a + b;
10 > 3;
arr[2];
'string'.toUpperCase();
(() => console.log('happy'))(); // 호출식; IIFE;  undefined 반환
<div>행복은 대체로 멀리있습니다.</div> // JSX 역시 표현식
```

- 호출식: javascript는 모든 함수가 호출될 때 값을 반환합니다. (`undefined`, `null`)



## <strong>#</strong> Identifier: 식별자

변수 중 하나. 메모리 주소의 별칭.



---
## 📚 출처

- [코드스피츠 ES6 기초편 강의](https://youtu.be/0j_eGoF8Q98)
- [코드 단위인 Expression과 Statement의 차이를 알아보자](https://shoark7.github.io/programming/knowledge/expression-vs-statement)

- [JavaScript에서 expression과 statement는 어떤 차이가 있을까요?](https://2ssue.github.io/common_questions_for_Web_Developer/docs/Javascript/expression_statement.html#%E1%84%8B%E1%85%A1%E1%86%AF%E1%84%8B%E1%85%A1%E1%84%83%E1%85%AE%E1%84%86%E1%85%A7%E1%86%AB-%E1%84%8C%E1%85%A9%E1%87%82%E1%84%8B%E1%85%B3%E1%86%AB-%E1%84%80%E1%85%A5%E1%86%BA)
