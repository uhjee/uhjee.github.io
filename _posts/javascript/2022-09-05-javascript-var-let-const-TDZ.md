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

변수를 선언할 때 javascript에서 `var`, `let`, `const`를 사용할 수 있습니다.


## <strong>#</strong> var

**function-scoped** ( block-scope을 갖지 않는다. → hosting되기 때문에 선언된 블럭 바깥에서도 접근 가능)
    
    {  
        var x = 2;  
    }  
    // x can be used here!
    
-   동명의 변수 재선언 가능
    

## <strong>#</strong> let

-   **block-scoped** 을 갖는다. (선언된 scope 안에서만 사용 가능 → hosting되지 않기 때문)

```javascript

{  
	let x = 2;  
}  
```    
-   동명의 변수 재선언 불가, 변수의 재할당 가능
    
![let_lifecycle](/assets/images/posts_img/javascript/let_lifecycle.jpg){: .align-center}    

## <strong>#</strong> const

-   **block-scoped** 을 갖는다. ( let 과 마찬가지로 선언된 scope 안에서만 사용 가능 → hosting되지 않기 때문))
    
-   동명의 변수 재선언 불가, 변수의 재할당 불가능(내부 프로퍼티의 추가 및 프로퍼티의 값 자체는 수정이 가능하다)


---
## <strong>#</strong> 변수 선언 3단계


![tdz](/assets/images/posts_img/javascript/tdz.png){: .align-center}

javascript에서는 scope이 생성될 때, 아래의 흐름으로 변수를 선언합니다.

### <strong>##</strong> 1. 선언
저희가 작성한 변수의 식별자가 실행 컨텍스트에  등록되는 단계를 말합니다.  변수의 선언은 `var`, `let,` `const` 키워드를 통해 이루어 집니다.
javascript 엔진에게 변수의 존재를 알리는 단계로,  선언 시점 이후로는 이 때 등록된 변수를 참조합니다.

런타임이 아닌 scope를 생성하는 시점에 먼저 수행됩니다. 
        
### <strong>##</strong> 2. 초기화
선언 단계에 서 등록된 변수에 값을 저장하기 위한 메모리 공간을 확보하는 단계입니다.
실제 값이 아닌 메모리 공간만 할당하기 때문에 참조하면 `undefined`를 반환하게 됩니다.

- `var` 키워드로 선언된 변수는 **(1) 선언**과 동시에 **(2) 초기화**가 이루어지기 때문에 호이스팅이라 불리는 현상이 발생합니다.
- `let`은  **(1) 선언**과  **(2) 초기화** 의 시점이 분리되어 있기 때문에 **TDZ** 가 존재합니다.

### <strong>##</strong> 3. 할당
초기화 단계에서 `undefined`로 초기화된 변수의 데이터 메모리에 저희가 작성한 실제 값을 할당하는 단계입니다.
**(1) 선언**, **(2) 초기화** 와는 다르게 런타임에 순차적으로 실행됩니다.

---
## <strong>#</strong> TDZ(Temporal Dead Zone)

TDZ는 스코프의 선언 시점부터 초기화 시점까지의 구간을 말합니다. 이 구간에서는 `let,` 과 `const`로 선언된 변수를 참조하려 하면, Reference Error가 발생합니다.

Reference 에러가 발생하는 이유는 변수는 선언되어 있으나 메모리가 확보되는 초기화 단계에 다다르지 못해  참조할 수 있는 주소값이 없기 때문입니다.

즉 **TDZ**는  `let`, `const`가  **(1) 선언**과  **(2) 초기화**가 분리되어 있기 때문에 발생하는 구역이라고 볼 수 있습니다.

## <strong>#</strong> 마무리


긴 글 읽어주셔서 감사합니다.

---

## 📚 참조
- [TDZ(Temporal Dead Zone)이란?](https://noogoonaa.tistory.com/78)
- [# var, let, const의 차이 ⏤ 변수 선언 및 할당, 호이스팅, 스코프](https://www.howdy-mj.me/javascript/var-let-const/)