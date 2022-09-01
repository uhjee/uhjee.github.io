---
title: '[Typescript, Data Structure] Queue, 우선순위 Queue 자료 구조 구현하기'
excerpt: 'Queue을 typescript로 직접 구현하며 이해하려 노력해봅니다.'

categories:
  - algorithm
tags:
  - [
      Typescript,
      algorithm,
      node,
      unit test,
      jest,
      ts,
      nest,
      Queue,
      Priority Queue,
      data structure,
    ]

permalink: /algorithm/lets-make-queue/

toc: true
toc_sticky: true

date: 2022-09-01
last_modified_at: 2022-09-01
---

<u>Learning JavaScript Data Structures and Algorithms 한국어판</u> 라는 책을 보며 공부한 내용에 대해 정리하도록 하겠습니다.

해당 서적의 예제는 javascript로 작성되어 있기 때문에 이해하기 무척 편했습니다. 하지만 읽는 동안 자료구조 및 알고리즘과 같이 강타입이 어울리는 주제와 물렁한 javascript가 잘 어울리지 않는다는 느낌을 떨칠 수 없었습니다. 따라서 제 포스트에서는 typescript로 각 자료구조, 알고리즘을 구현하며 이해해보도록 하겠습니다.

하단의 코드는 모두 [여기](https://github.com/uhjee/TIL/tree/master/data-structure/typescript)에서 자세히 보실 수 있습니다.

## <strong>#</strong> Queue란?

Stack은 기본적인 자료구조 중 하나로, 먼저 들어온 데이터가 먼저 나가게 되는 구조를 갖습니다.

Queue는 크게 선형 큐와 환형 큐로 분류할 수 있습니다. 이번 포스트에서는 선형 큐에 대해서 알아보도록 하겠습니다.

## #선형 Queue

영어 사전에 검색을 하면, '대기줄'이라는 의미를 갖는다고 말해줍니다. 1등을 많이 배출한 로또 복권방을 대기줄을 상상해보겠습니다. 실제 먼저 줄을 선 사람이 먼저 복권을 사게 되고, 가장 나중에 줄을 선 사람이 가장 마지막에 복권을 구매하게 됩니다. 또 다르게 표현하면, 복권을 사기 위해 줄을 서려면 대기줄의 가장 뒤에 위치해야합니다. 새치기는 경범죄와 다를 바가 없기 때문이죠. 그리고 자신의 차례가 되어서 줄을 나가게 될 때는 대기줄의 가장 앞에서 나가게 됩니다.

위의 예제에서 '대기줄' 이 자료구조 **Queue**에 해당됩니다. Queue 가장 앞의 위치를 **front** 또는 head라고 부르고, 가장 끝 위치를 **rear** 또는 tail이라고 부릅니다. 줄을 서기 위해 대기줄 끝에 새로운 사람이 추가되는 것을 **enqueue**라고 부르고, 대기줄 가장 앞에서 나가게 되는 것을 **dequeue**라고 부릅니다.

![datastructure-queue](/assets/images/posts_img/algorithm/datastructure-queue.png){: .align-center}

<strong>선형 Queue의 특징</strong>은 다음과 같습니다.

- <u>FIFO</u>: 가장 빨리 들어온 요소가 가장 빨리 나가게 됩니다. (First-in, First-out)
- <u>front</u>: front는 Queue의 가장 앞 위치를 가리킵니다. 즉 front에 해당하는 요소는 가장 먼저 나갈 수 있습니다.
- <u>rear</u>: rear는 Queue의 가장 마지막 위치를 가리킵니다. 새로운 요소가 구조에 들어온다면 바로 이 위치에 들어오게 됩니다.

추가로 Queue에는 다음과 같은 용어가 있습니다. Queue의 크기가 동적일 경우에는 발생하지 않습니다. 물론 자원이나 메모리의 소비가 효율적이진 않겠지만요...

- <u>Overflow</u>: Queue가 가득 차 있을 때, 요소를 삽입하려 할 경우 발생
- <u>Underflow</u>: Queue가 텅텅 비어있을 때, 요소를 꺼내려 시도할 경우 발생

## <strong>#</strong> 선형 Queue주요 메소드

선형 Queue 자료 구조는 주로 다음과 같은 메소드를 갖고 있습니다. 각각의 기능은 다음과 같습니다.

- `enqueue()`: rear에 요소를 삽입한다.
- `dequeue()`: front에 있는 요소를 가져오고, 삭제한다.
- `front()`: front에 있는 요소를 가져오고, 삭제하지 않는다.
- `isEmpty()`: queue내부에 요소가 있는지 여부를 판단한다.
- `clear()`: queue내부의 모든 요소를 삭제한다.
- `size()`: queue 내부에 있는 요소의 개수를 반환한다.
- `print()`: console에 queue내부 요소들의 값을 출력한다.

## <strong>#</strong> 선형 Queue구현을 위한 테스트 코드 작성

위에 작성한 각각의 메소드에 대한 테스트 케이스를 정리해 테스트 코드를 작성합니다. `jest`, `ts-jest` 라이브러리를 사용해 테스트를 진행하도록 하겠습니다.

코드의 양이 많아 포스트에는 `enqueue()`와 `dequeue()`에 대한 테스트를 예시로 다루도록 하겠습니다. 각각의 테스트 케이스와 테스트 코드는 다음과 같이 정의하도록 하겠습니다.

그리고 테스트 코드 내에서 mockup용 queue의 클래스 인스턴트에 접근하기 위해 Reflect API를 사용하도록 할게요.

`enqueue()`

1. 요소를 하나 enqueue하면, 가장 마지막 인덱스에 추가되어야 한다.

테스트 코드로 작성해보도록 하겠습니다.

```typescript
/* queue.spec.ts */
describe('Test for Class Queue', () => {
  describe('enqueue()', () => {
    it('요소를 하나 enqueue하면, 가장 마지막 인덱스에 추가되어야 한다.', () => {
      const queue = new Queue();
      Reflect.set(queue, 'items', ['sadness', 'blue']);
      queue.enqueue('happiness');
      const items = Reflect.get(queue, 'items');
      expect(items[items.length - 1]).toEqual('happiness');
    });
      
  // ...
});
```

`dequeue()`

1. 비어있는 items에 dequeue한 경우, undefined를 반환해야 한다.
   - 저희는 가변길이 queue를 구현하기 때문에, underflow를 발생시키지 않겠습니다.
2. dequeue하면, 첫 번째 인덱스의 데이터를 반환해야 한다.
3. dequeue하면, items의 길이가 하나 줄어들어야 한다.

테스트 코드로 작성해보도록 하겠습니다.

```typescript
/* queue.spec.ts */
describe('Test for Class Queue', () => {
 	// ...
    
    it('비어있는 items에 dequeue한 경우, undefined를 반환해야 한다.', () => {
      const queue = new Queue();
      const dequeuedValue = queue.dequeue();
      expect(dequeuedValue).toBeUndefined();
    });
    
    it('dequeue하면, 첫 번째 인덱스의 데이터를 반환해야 한다.', () => {
      const queue = new Queue();
      Reflect.set(queue, 'items', ['sadness', 'blue']);
      const dequeuedValue = queue.dequeue();
      expect(dequeuedValue).toEqual('sadness');
    });
    
    it('dequeue하면, items의 길이가 하나 줄어들어야 한다.', () => {
      const queue = new Queue();
      Reflect.set(queue, 'items', ['sadness', 'blue']);
      queue.dequeue();
      const items = Reflect.get(queue, 'items');

      expect(items.length).toBe(1);
    });
});
```

## <strong>#</strong> Queue 클래스 구현

테스트 코드를 모두 작성했으니 실제 선형 Queue 클래스를 구현해보도록 하겠습니다.

javascript는 언어 레벨에서 Queue 자료구조를 지원하고 있지 않습니다. 따라서 배열을 베이스로 queue을 구현해보도록 하겠습니다.

사이즈는 고정이 아닌 변동으로 구현하도록 하겠습니다. 따라서 Overflow와 같은 상황은 우리가 구현하는 Queue에서는 발생하지 않습니다. 반대로 Underflow 또한 예외를 던지지 않고 undefined를 반환하는 방향으로 구현하도록 하겠습니다.

```typescript
/**
 * queue 자료 구조 구현체
 */
export default class Queue<T> {
  private items: T[] = [];

  enqueue(element: T) {
    this.items.push(element);
  }

  dequeue() {
    return this.items.shift();
  }

  front() {
    return this.items[0];
  }

  isEmpty() {
    return this.items.length === 0;
  }

  size() {
    return this.items.length;
  }

  clear() {
    this.items = [];
  }

  print() {
    console.log(this.items.toString());
  }
}
```

`class Queue<T>{}`

- 제네릭으로 특정 타입을 받을 수 있도록 구현하였습니다.

`private items: T[] = [];`

- private 로컬 변수로 items를 선언하였습니다. `private`으로 선언했기 때문에 외부에서 직접 접근할 수 없습니다.
- 제네릭으로 받은 타입의 배열을 타입으로 갖도록 선언하였습니다.

## <strong>#</strong> 테스트 코드 실행

Queue구현을 마쳤으니 작성해두었던 테스트 코드를 실행시켜보도록 하겠습니다.

![datastructure-queue-test](/assets//images/posts_img/algorithm/datastructure-stack-test.png){: .align-center}

다행히 모든 테스트가 의도한대로 통과한 것을 확인할 수 있습니다.

## <strong>#</strong> 마무리

선형 Queue의 문제점은 enqueue로 데이터가 빠져나간 자리를 다시 사용하지 못한다는 점입니다. 

Typescript에서 Stack 클래스를 구현하기 위해 Stack에 대한 정의를 살펴보고, 구현을 위한 테스트 코드를 작성하고 마지막으로 실제 구현 후 테스트까지 진행해보았습니다.

Stack은 간단한 자료구조이지만, 다양한 곳에서 사용되고 있고, FE 개발자라면 익숙할 javascript의 콜스택 또한 이 자료구조를 사용하는 개념입니다. 잘 숙지해서 적시적소에 사용하실 수 있길 바라겠습니다.

긴 글 읽어주셔서 감사합니다.

---

## 📚 참조

[Learning JavaScript Data Structures and Algorithms 한국어판, 로이아니 그로네르 저, 이일웅 역, 2015](http://www.yes24.com/Product/Goods/22885878)
