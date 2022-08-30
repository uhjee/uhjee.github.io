---
title: '[Typescript, Data Structure] Stack 자료 구조 구현하기'
excerpt: 'Stack을 typescript로 직접 구현하며 이해하려 노력해봅니다.'

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
      stack,
      data structure,
    ]

permalink: /algorithm/lets-make-stack/

toc: true
toc_sticky: true

date: 2022-08-30
last_modified_at: 2022-08-30
---

<u>Learning JavaScript Data Structures and Algorithms 한국어판</u> 라는 책을 보며 공부한 내용에 대해 정리하도록 하겠습니다.

해당 서적의 예제는 javascript로 작성되어 있기 때문에 이해하기는 편했습니다. 하지만 읽는 동안 자료구조 및 알고리즘과 같이 강타입이 어울리는 주제와는 조금 동떨어진 것 같다는 느낌을 떨칠 수 없었습니다. 따라서 제 포스트에서는 typescript로 각 자료구조, 알고리즘을 구현하며 이해해보도록 하겠습니다.

하단의 코드는 모두 [여기](https://github.com/uhjee/TIL/tree/master/data-structure/typescript)에서 자세히 보실 수 있습니다.

## <strong>#</strong> Stack이란?

Stack은 자료구조 중 하나로, 한 쪽 끝에서만 자료를 넣거나 뺄 수 있는 구조입니다.

자료들을 찾아보면 주로 '책 쌓기'가 그 예로 등장합니다. 책상에 책을 눕힌 채로 쌓는 상황을 상상해볼게요. 책은 책상부터 하나씩 쌓일 것이고, 새로운 책을 쌓을 때는 책들의 가장 상단에 두게 됩니다. 그리고 책들의 탑에서 책을 꺼낼 때에는 탑이 무너지지 않기 위해서 가장 위의 것부터 하나씩 꺼내게 됩니다. 이런 상황이라면 가장 나중에 쌓은 책이 가장 먼저 꺼내지게 됩니다.

위의 예제에서 '책들의 탑'이 자료구조 <strong>Stack</strong>에 해당됩니다. '책들의 탑' 가장 위에 있는 책은 Stack에서 <strong>Top</strong>이라는 용어로 불립니다. 그리고 '책들의 탑 가장 위에 책을 쌓는 행위'가 stack의 `push()` 메소드에 해당됩니다. 마찬가지로 '책들의 탑 가장 위에서 책을 꺼내는 행위'가 stack의 `pop()` 메소드에 해당합니다.

![datastructure-stack](/assets/images/posts_img/algorithm/datastructure-stack.png){: .align-center}

<strong>Stack의 특징</strong>은 다음과 같습니다.

- <u>선형 구조</u>: Stack 내부의 각 요소들은 순서를 갖고 나열되어 있습니다.
- <u>Top</u>: Top이라 불리우는 한 쪽 끝을 통해 요소가 삽입되고 삭제됩니다.
- <u>LIFO</u>: 가장 늦게 들어온 요소가 가장 빨리 나가게 됩니다. (Last-in, First-out)

추가로 Stack에는 다음과 같은 용어가 있습니다. 늘 신세를 지고 있는 그 사이트와 이름이 같습니다. 프로그래밍 시 스택의 메모리가 부족하면 자주 등장하는 에러도 이에 해당합니다.

- <u>Stack Overflow</u>: Stack이 가득 차 있을 때, 요소를 삽입하려 할 경우 발생
- <u>Stack Underflow</u>: Stack이 비어있을 때, 요소를 꺼내려 시도할 경우 발생

## <strong>#</strong> Stack 주요 메소드

Stack 자료 구조는 주로 다음과 같은 메소드를 갖고 있습니다. 각각의 기능은 다음과 같습니다.

- `push()`: 한 쪽 끝(Top)에 요소를 삽입한다.
- `pop()`: 한 쪽 끝(Top)에 있는 요소를 가져오고, 삭제한다.
- `peek()`: 한 쪽 끝(Top)에 있는 요소를 가져오고, 삭제하지 않는다.
- `isEmpty()`: Stack 내부에 요소가 있는지 여부를 판단한다.
- `clear()`: Stack 내부의 모든 요소를 삭제한다.
- `size()`: Stack 내부에 있는 요소의 개수를 반환한다.
- `print()`: console에 stack 내부 요소들의 값을 출력한다.

## <strong>#</strong> Stack 구현을 위한 테스트 코드 작성

위에 작성한 각각의 메소드에 대한 테스트 케이스를 정리해 테스트 코드를 작성합니다. `jest`, `ts-jest` 라이브러리를 사용해 테스트를 진행하도록 하겠습니다.

코드의 양이 많아 포스트에는 `push()`와 `pop()`에 대한 테스트를 예시로 다루도록 하겠습니다. 각각의 테스트 케이스와 테스트 코드는 다음과 같이 정의하도록 하겠습니다.

그리고 테스트 코드 내에서 mockup용 stack의 클래스 인스턴트에 접근하기 위해 Reflect API를 사용하도록 할게요.

`push()`

1. push한 파라미터의 개수 만큼 items의 길이가 추가되어야 한다.

테스트 코드로 작성해보도록 하겠습니다.

```typescript
/* stack.spec.ts */
describe('Test for Class Stack', () => {
  describe('push()', () => {
    it('push한 파라미터의 개수 만큼 items의 길이가 추가되어야 한다. ', () => {
      const stack = new Stack();
      Reflect.set(stack, 'items', [1, 2, 3]);

      stack.push(4, 5, 6, 7);
      const items = Reflect.get(stack, 'items');
      expect(items.length).toBe(7);
    });
  });
  // ...
});
```

`pop()`

1. 가장 마지막 데이터가 없을 경우, undefined를 반환해야 한다.
2. items의 가장 마지막 데이터를 반환해야 한다.
3. 성공적으로 값을 반환했을 경우, items의 개수가 하나 줄어들어야 한다.

테스트 코드로 작성해보도록 하겠습니다.

```typescript
/* stack.spec.ts */
describe('Test for Class Stack', () => {
  // ...
  describe('pop()', () => {
    it('가장 마지막 데이터가 없을 경우, undefined를 반환해야 한다.', () => {
      const stack = new Stack();
      expect(stack.pop()).toBeUndefined();
    });

    it('items의 가장 마지막 데이터를 반환해야 한다.', () => {
      const stack = new Stack();
      Reflect.set(stack, 'items', ['happy', 'life', 'enjoy']);
      expect(stack.pop()).toEqual('enjoy');
    });

    it('성공적으로 값을 반환했을 경우, items의 개수가 하나 줄어들어야 한다.', () => {
      const stack = new Stack();
      Reflect.set(stack, 'items', ['happy', 'life', 'enjoy']);
      const items = Reflect.get(stack, 'items');
      stack.pop();
      expect(items.length).toBe(2);
    });
  });
});
```

## <strong>#</strong> Stack 클래스 구현

테스트 코드를 모두 작성했으니 실제 Stack 클래스를 구현해보도록 하겠습니다.

javascript는 언어 레벨에서 stack 자료구조를 지원하고 있지 않습니다. 따라서 배열을 베이스로 stack을 구현해보도록 하겠습니다.

사이즈는 고정이 아닌 변동으로 구현하도록 하겠습니다. 따라서 stack Overflow와 같은 상황은 작성할 Stack에서는 발생하지 않습니다. 반대로 stack Underflow 또한 예외를 던지지 않고 undefined를 반환하는 방향으로 구현하도록 하겠습니다.

```typescript
/* stack.ts */
export default class Stack<T> {
  private items: T[] = [];

  push(...args: T[]) {
    for (const arg of args) {
      this.items.push(arg);
    }
  }

  pop() {
    return this.items.pop();
  }

  peek() {
    return this.items[this.items.length - 1];
  }

  isEmpty() {
    return this.items.length === 0;
  }

  clear() {
    this.items = [];
  }

  size() {
    return this.items.length;
  }

  print() {
    console.log(this.items.toString());
  }
}
```

`class Stack<T>{}`

- 제네릭으로 특정 타입을 받을 수 있도록 구현하였습니다.

`private items: T[] = [];`

- private 로컬 변수로 items를 선언하였습니다. `private`으로 선언했기 때문에 외부에서 직접 접근할 수 없습니다.
- 제네릭으로 받은 타입의 배열을 타입으로 갖도록 선언하였습니다.

## <strong>#</strong> 테스트 코드 실행

Stack 구현을 마쳤으니 작성해두었던 테스트 코드를 실행시켜보도록 하겠습니다.

![datastructure-stack-test](/assets//images/posts_img/algorithm/datastructure-stack-test.png){: .align-center}

다행히 모든 테스트가 의도한대로 통과한 것을 확인할 수 있습니다.

## <strong>#</strong> 마무리

Typescript에서 Stack 클래스를 구현하기 위해 Stack에 대한 정의를 살펴보고, 구현을 위한 테스트 코드를 작성하고 마지막으로 실제 구현 후 테스트까지 진행해보았습니다.

Stack은 간단한 자료구조이지만, 다양한 곳에서 사용되고 있고, FE 개발자라면 익숙할 javascript의 콜스택 또한 이 자료구조를 사용하는 개념입니다. 잘 숙지해서 적시적소에 사용하실 수 있길 바라겠습니다.

긴 글 읽어주셔서 감사합니다.

---

## 📚 참조

[Learning JavaScript Data Structures and Algorithms 한국어판, 로이아니 그로네르 저, 이일웅 역, 2015](http://www.yes24.com/Product/Goods/22885878)
