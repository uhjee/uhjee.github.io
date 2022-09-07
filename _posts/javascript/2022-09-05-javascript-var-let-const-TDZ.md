---
title: '[Javascript] TDZ(Temporal Dead Zone)과 var, let, const에 대해 알아봅니다.'
excerpt: 'TDZ(Temporal Dead Zone)과 var, let, const에 대해 알아봅니다.'

categories:
  - javascript
tags:
  - [
      javascript,
      TDZ,
      Temporal Dead Zone,
      var,
      let,
      const
    ]

permalink: /javascript/var-let-const-TDZ/

toc: true
toc_sticky: true

date: 2022-09-05
last_modified_at: 2022-09-05

---

변수를 선언할 때 javascript는 `var`, `let`, `const` 3가지 키워드를 사용할 수 있습니다. 각각의 키워드들의 역할과 내부 동작은 각각 차이가 있습니다. 이 점들에 대해 정리한 내용입니다.


## <strong>#</strong> var
ES5까지 변수를 선언할 수 있는 유일한 방법은 `var` 키워드를 사용하는 것이었습니다. `var` 키워드의 특징은 다음과 같습니다.

1. **함수 레벨 스코프(Function-level scope)** 를 갖는다.
	>초기 javascript 는 함수를 기준으로 세워진 기준들이 많습니다. `var` 키워드가 갖는 함수 레벨 scope도 그 중 하나입니다.  <br>
	> 함수의 코드 블록(`{ 여기 }`)만 scope로 인정해 for문에서 선언한 변수를 외부에서 참조할 수 있습니다.<br>
	> 이는 전역 변수들로 등록될 가능성이 높아, 혼란을 야기합니다.

2. `var` 키워드 생략 허용
	> 암묵적으로 전역 변수를 양산할 가능성이 높습니다.

3. 변수 중복 선언 허용
	> 의도치 않게 상단에 선언된 변수의 값이 변경될 가능성이 높습니다.

4. 변수의 호이스팅
	> 코드 흐름 상 변수가 선언되기 전에 참조가 가능합니다.

```javascript
console.log(i); // 0

for (var i = 0; i < 5; i++) {
  console.log(`inner i: ${i}`);
}
```

위의 코드를 보시면 문제가 많습니다. 먼저 코드 흐름 상 선언되어 있지 않은 `i` 변수에  대해 접근하려 하고 있습니다. 그리고 심지어 그 `i`는 for문의 내부에서 사용하는 변수입니다. 하만 위 코드를 실행시키면 `0`이라는 값을 정상적으로 출력합니다. 왠지 일반적인 상식과는 조금 다르게 돌아가는 것을 알 수 있습니다.
    

## <strong>#</strong> let
위에서 보신 `var` 키워드의 단점을 ES6에서 도입된 `let` 키워드는 다음과 같은 특징을 갖습니다.

1. **블록 레벨 스코프(Block-level scope)** 를 갖습니다.
	> 대부분의 프로그래밍 언어와 같이 `let `키워드를 통해 선언된 변수는 블록 레벨 스코프를 갖습니다. <br>
	> 모든 코드 블록(함수, if문, for문, while문, try..catch문 등) 내에서 선언된 변수는 해당 코드 블록 내에서만 유효합니다. <br>
	> 즉 코드 블록 내부에서 선언한 변수는 지역변수(local variable)입니다.

2. 변수 중복을 허용하지 않습니다.
3. 호이스팅이 이루어지지 않습니다.
4. **TDZ**(일시적 사각지대)를 갖습니다.
    
## <strong>#</strong> const
let과 동일하게 ES6에 추가된 const 키워드는 let 키워드와 유사하지만, 몇 가지 차이점이 있습니다. 특징은 다음과 같습니다.

1. **블록 레벨 스코프(Block-level scope)** 를 갖습니다.
2. 변수 중복을 허용하지 않습니다.
3. 호이스팅이 이루어지지 않습니다.
4. **선언과 동시에 할당** 이 이루져야 합니다.
5. **재할당이 불가**합니다.

let과의 가장 큰 차이점은 4, 5번 째 특징입니다.  `const` 키워드를 통해 변수를 선언하는 경우, 선언과 할당이 동시에 이루어져야 합니다. 

아래와 같이 선언만 한 경우, Syntax Error 가 발생하게 됩니다.

```javascript
const FOO; // SyntaxError: Missing initializer in const declaration
```

그리고 const 키워드를 통해 생성된 변수는 재할당이 불가능합니다. 하지만 내부 프로퍼티의 추가 및 프로퍼티 값의 수정은 가능합니다.


---
## <strong>#</strong> 변수 선언 3단계

각 키워드의 특징을 살펴보았습니다. 이제는 각 키워드 별로 내부적으로 scope에 어떻게 등록되고 선언(생성)되는지 알아보도록 하겠습니다.


![tdz](/assets/images/posts_img/javascript/tdz.png){: .align-center width="60%" height="60%"}


javascript에서는 scope이 생성될 때, 아래의 흐름으로 변수를 선언(생성)합니다.

### <strong>##</strong> 1. 선언
저희가 작성한 변수의 식별자가 실행 컨텍스트에  등록되는 단계를 말하며, 변수의 선언은 `var`, `let,` `const` 키워드를 통해 이루어 집니다.
javascript 엔진에게 변수의 존재를 알리는 단계로,  선언 시점 이후로는 이 때 등록된 변수를 참조합니다.
런타임이 아닌 scope를 생성하는 시점에 먼저 수행됩니다. 
        
### <strong>##</strong> 2. 초기화
선언 단계에 서 등록된 변수에 값을 저장하기 위한 메모리 공간을 확보하는 단계입니다.
실제 값이 아닌 메모리 공간만 할당하기 때문에 참조하면 `undefined`를 반환하게 됩니다.
변수가 선언되는 키워드에 따라 시점이 달라집니다.

### <strong>##</strong> 3. 할당
초기화 단계에서 `undefined`로 초기화된 변수의 데이터 메모리에 저희가 작성한 실제 값을 할당하는 단계입니다.
선언 초기화 단계와 다르게 **런타임**에 수행됩니다.

### <strong>##</strong> 키워드 별 변수 선언 단계

`var`, `let`, `const` 키워드 별로 선언, 초기화가 수행되는 시점이 모두 다릅니다. 또 그렇기 때문에 각 키워드 별로 아래와 같은 특징을 갖게 됩니다.

#### `var`
1. 선언, 초기화
2. 할당

`var` 키워드로 선언된 변수는 **선언**과 동시에 **초기화**가 이루어집니다. 따라서 문맥 흐름과 관계없이 끌어 올려지는  호이스팅이라 불리는 현상이 발생합니다.

---
#### `let`
1. 선언
2. 초기화
3. 할당

`let` 키워드는 **선언**, **초기화**, **할당**이 모두 분리되어 있습니다. 그렇기 때문에 호스스팅 현상은 발생하지 않습니다.

하지만 `let` 키워드로 선언된 변수는 **TDZ**라는 구역이 존재합니다. 선언과 초기화가 분리되었기 때문인데요. 아래 문단에서 자세히 다루도록 하겠습니다.

---
#### `const`
1. 선언, 초기화, 할당

`const` 키워드는 변수를 생성할 때, 선언, 초기화, 할당을 동시에 해야 합니다. 그렇지 않으면 에러가 발생합니다.

---
## <strong>#</strong> TDZ(Temporal Dead Zone)

TDZ는 스코프의 **선언** 시점부터 **초기화** 시점까지의 구간을 말합니다. 이 구간에서는 `let`으로 선언된 변수를 참조하려 하면, Reference Error가 발생합니다.

![let_lifecycle](/assets/images/posts_img/javascript/let_lifecycle.jpg){: .align-center  width="60%" height="60%"}    

Reference 에러가 발생하는 이유는 변수는 선언되어 있으나 메모리가 확보되는 초기화 단계에 다다르지 못해  참조할 수 있는 주소값이 없기 때문입니다.

```javascript
// 에러 없이 상위 scope에서 행복을 찾는 코드

let happiness = '행복';

const printHappiness = () => {
  console.log(happiness);
};

printHappiness(); // '행복'

```

위 코드는 문제가 없습니다. `printHappiness` 함수의 scope에서 `happiness` 변수를 찾지 못했지만,  상위 scope에서 `happiness` 변수를 찾아 값에 접근이 가능합니다.

```javascript
// Reference Error 발생 코드

let happiness = '행복';

const printHappiness = () => {
  console.log(happiness); // 이 시점엔 happiness 변수의 '선언'만 실행됨
  let happiness = '행복행복' // '초기화' 및 '할당' 단계
};

printHappiness(); // ReferenceError: Cannot access 'happiness' before initialization
```

위 코드는 에러가 발생합니다. `printHappiness` 함수의 scope 생성 시, 함수 내에서 let으로 선언된 happiness 변수의 식별자는 자신의 scope에 등록합니다. (선언 단계)
하지만 `happiness` 변수에 접근하는 시점에는 변수는 접근이 가능한 반면 실제 메모리가 할당되지 않았기 때문에, Reference Error 가 발생합니다. 초기화와 할당 단계에 도달하지 못했기 때문입니다. 

이러한 시점이 **TDZ**라고 불리며, 이는 `let`키워드로 선언된 변수가 각각 선언, 초기화, 할당 단계가 분리되어 있기 때문에 발생하는 증상입니다.

즉 **TDZ**는  `let`으로 선언된 변수가  **선언**과  **초기화** 단계가 분리되어 있기 때문에 발생하는 구역이라고 볼 수 있습니다.

## <strong>#</strong> 마무리
밥 먹듯 사용하고 있지만, 처음 javascript를 배울 때를 제외하고는 자주 떠올리지 않는 변수의 선언에 대해서 알아보았습니다. 

추가로  키워드는 다음과 같은 상황에 맞추어 사용하는 것이 좋습니다.
- ES6 이상을 사용한다면 `var` 키워드는 사용하지 않는 것이 좋습니다.
- 재할당이 필요한 경우 또는 선언과 할당이 분리되어야 하는 경우에는 `let` 키워드를 사용합니다.
- 재할당이 이루어지지 않는 경우에는 `const` 키워드를 사용합니다.

다시 오랜만에 기본으로 돌아가 개념을 정리하게 되어 매우 기쁩니다.
긴 글 읽어주셔서 감사합니다.

---

## 📚 참조
- [TDZ(Temporal Dead Zone)이란?](https://noogoonaa.tistory.com/78)
- [# var, let, const의 차이 ⏤ 변수 선언 및 할당, 호이스팅, 스코프](https://www.howdy-mj.me/javascript/var-let-const/)
- [let, const와 블록 레벨 스코프](https://poiemaweb.com/es6-block-scope)