---
title: '[Javascript] 얕은 복사와 깊은 복사 (Shallow copy & Deep copy)'
excerpt: ''

categories:
  - javascript
tags:
  - [
      Typescript,
      javascript,
      node,
      shallowcopy,
      deepcopy,
      callbyreference,
      callbyvalue
    ]

permalink: /javascript/shallowcopy-deepcopy/

toc: true
toc_sticky: true

date: 2022-09-03
last_modified_at: 2022-09-03

---

실무에서 Single Page App을 지원하는 Vue.js를 사용하고 있습니다. Vue.js 와 React 같은 프레임워크(또는 라이브러리)는 **state가 변하면 다시 렌더링한다.** 라는 큰 흐름으로 동작합니다.

따라서 Vue.js와 React에서는 state를 **불변 객체**로써 관리하는 것이 상당히 중요합니다. 불변객체로 다루지 않는다면, 실무에서 state를 변경했으나, 화면이 렌더링되지 않는 상황이 있고, 반대로 의도치 않았는데 렌더링이 상당히 많이 이루어지는 경우가 있습니다.

그렇기 때문에 실무 코드에는 전개 연산자를 통해 할당하는 방법과 lodash 라이브러리의 deepClone를 사용해 객체를 복사해 사용하는 경우가 많습니다. 이 둘은 각각 얕은 복사와 깊은 복사에 해당합니다. **얕은 복사**와 **깊은 복사**에 대해 알아보고, 이를 구현하는 몇 가지 방법을 알아보도록 하겠습니다.

## <strong>#</strong> 얕은 복사와 깊은 복사의 차이점

얕은 복사, 깊은 복사의 차이점은 복사하려는 객체의 속성(property)으로 객체를 가지고 있을 경우(중첩된 객체, 참조형 안에 참조형 데이터), <u>속성으로 가진 객체를 참조 값으로 복사하느냐 또는 그 값 자체(primitive type)를 복사하는지 여부</u>입니다.

![javascript-copy](/assets/images/posts_img/javascript/javascript-copy.png){: .align-center}

**얕은 복사**는 속성의 **주소**를 복사하는 반면, **깊은 복사**는 내부의 모든 depth의 **값**들을 하나 하나 찾아서 전부 복사하는 방법입니다.

이제 각각의 구현 방법에 대해 알아보겠습니다.


## <strong>#</strong> Shallow Copy; 얕은 복사

복사하고자 하는 객체 안의 속성이 primitive type이라면 값 자체를 복사하지만, 속성으로 객체를 갖고 있는 경우, 참조값을 복사하게 됩니다.

따라서 얕은 복사를 했을 경우, 복사한 객체 내부 속성으로 정의된 객체를 변경할 경우, 원본 데이터에도 변경을 가하게 되므로, 중첩된 객체의 모든 값을 새롭게 복사할 때에는 적합하지 않습니다.

```javascript
// 중첩된 객체 선언
const testData = {  
    a: {  
        aaa: 1  
    },  
    b: 2  
};  
  
// 얕은 복사  
const copyData = {...testData};   

// 복사한 데이터에서 속성 객체 변경  
copyData.a.aaa = 100;  
// 원본 객체 데이터 확인
console.log(testData);  // {a: {aaa: 100}, b: 2} -- 변경됨
```

따라서 주로 Call by Value의 primitive type의 복사에 사용됩니다. javascrip로 얕은 복사를 구현할 수 있는 몇 가지 방법에 대해서 알아봅니다.

### <strong>##</strong> 함수를 만들어서 사용
새로운 객체를 만들어 원본 객체의 값을 넣어 반환하는 함수를 만들어 사용합니다.

```javascript
// 얕은 복사 함수 생성 및 데이터 할당
function shallowCopy(val){
	const result = {};

	for(const key in val){
		result[key] = val[key];
	}
	return result;
}

// 호출
const testData = {
	a: 1, 
	b: 2 
};
const copyData = shallowCopy(testData);

console.log(copyData);  // { a: 1, b: 2 }
```

### <strong>##</strong> Object.assign() 메소드 사용

ES6에 추가된 메소드로, 객체를 병합하는 메소드 입니다.
하지만 빈 객체와 병합할 경우, 얕은 복사와 같은 기능으로 동작합니다.

```javascript
const testData = {  
    a: 1,  
    b: 2  
};  
const copyData = Object.assign({}, testData); // 빈 객체와 병합 => Copy
  
console.log(copyData);  // { a: 1, b: 2 }
```

### <strong>##</strong> 전개 연산자 사용

ES6부터 지원하는 전개 연산자(spread syntax)를 사용한 방법입니다.
전개 연산자로 각 property만을 가져오고 새로운 객체로 감싸줍니다.
간단하기 때문에 실무에서 얕은 복사가 필요한 경우, 이 방법을 가장 많이 사용합니다.

```javascript
const testData = {  
    a: 1,  
    b: 2  
};  
const copyData = {...testData};  
  
console.log(copyData);  // { a: 1, b: 2 }
```

---


## <strong>#</strong> Deep Copy; 깊은 복사

속성으로 정의된 객체의 참조 값을 복사하는 것이 아닌 primitive type 값으로 복사합니다.
따라서 원본 객체와 새롭게 복사한 객체의 연결이 완전히 끊어지게 됩니다.

### <strong>##</strong> JSON Object 사용

JSON 문자열로 변환 후, 다시 JS 객체로 변환하는 방법입니다.

하지만 성능적으로 좋지 못하고, 복사하려는 객체의 속성 중 값이 함수이거나 undefined일 경우 완전한 복사가 되지 않는다는 문제가 있습니다.

```javascript
// a는 속성으로 정의된 객체
const testData = { a: { aa: 1 }, b: 2 };
const copyData = JSON.parse(JSON.stringify(testData));

copyData.a.aa = 100;

console.log(testData);  // { a: { aa: 1 }, b: 2 } -- 변경 안됨
```

### <strong>##</strong> 함수를 만들어 사용

객체의 속성 타입이 `object` 인 경우, 재귀호출로 내부를 다시 복사하게 됩니다.
따라서 모든 depth 속성(property)들의 데이터는 원본 데이터와 연결이 끊어지게 됩니다.

```javascript
// 깊은 복사 함수 생성 및 변수 할당
function deepCopy(value){  
    const result = {};  
  
    for(const key in value){  
        // 만약 속성의 데이터타입이 'object' 라면,  
        if(typeof value[key] === 'object' && typeof value[key] !== null){  
            // 재귀 호출  
            result[key] = deepCopy(value[key]);  
        } else {  
            result[key] = value[key];  
        }  
    }  
    return result;  
}  
  
// a는 속성으로 정의된 객체  
const testData = { a: { aa: 1 }, b: 2, c: null};  
const copyData = deepCopy(testData);  
  
copyData.a.aa = 100;  
  
console.log(testData);  // { a: { aa: 1 }, b: 2 } -- 변경 안됨
```

## <strong>#</strong> 마무리
실제 개발 시, 얕은 복사와 깊은 복사의 개념이 헷갈릴 때가 종종 있습니다.  얕은 복사와 깊은 복사는 자주 쓰이는 기능인 만큼 이번 기회에 단단히 기억하고, 각각 목적에 따라 적절히 사용할 수 있었으면 하는 마음으로 끝을 내겠습니다.

긴 글 읽어주셔서 감사합니다.

---

## 📚 참조

[코어 자바스크립트, 정재남 저, 위키북스, 2019](http://www.yes24.com/Product/Goods/78586788)
