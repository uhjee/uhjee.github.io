---
title: '[Nest.js, Jest] Nest.js with TDD(1) - Controller'
excerpt: 'TDD를 통해 todolist 로직 생성하기 - 컨트롤러 편'

categories:
- nest
tags:
- [Typescript, node, test, unit test, jest, ts, nest, nest.js, TDD, spyOn, mocking, controller, service, mock, module]

permalink: /nest/nestjs-with-tdd-controller/

toc: true
toc_sticky: true

date: 2022-08-25
last_modified_at: 2022-08-25
---

아쉽게도 저는 실무에서 유닛 테스트 및 E2E 테스트를 진행하고 있지 않습니다... 유지보수를 담당하는 팀에서 근무를 하기 때문에 TDD에 대해 깊게 공부하진 않았음에도  '테스트 코드가 있었다면, 이런 사소한 결함들은 줄어들지 않았을까'라는 생각이 자주 들곤 했습니다. 따라서 Nest.js와 TDD를 통해 간단한 Todo list API를 구현하며, 머릿속에 TDD를 집어넣도록 하겠습니다.

하단의 코드는 모두 [여기](https://github.com/uhjee/TIL/tree/master/nest_js/nest-tdd)에서 보실 수 있습니다.

## <strong>#</strong> 계획

전체적인 흐름은 아래와 같이 진행하도록 하겠습니다.

1. 실패하는 테스트 작성
2. 위의 실패 테스트를 통과하도록 구현
3. 리팩토링

실제 코드의 구현은 <u>controller -> service -> repository</u> 순으로 진행하도록 하겠습니다.
(전체 로직을 포스팅하기에는 양이 많아, <u>todo를 create하는 로직</u>을 예시로 사용하겠습니다.)

그리고 프로젝트 트리는 다음과 같습니다.
```text
.src/todos
├── dto
│   └── create-todo.dto.ts
├── entities
│   └── todo.entity.ts
├── specs
│   ├── mock-todo.ts
│   ├── todos.controller.spec.ts
│   └── todos.service.spec.ts
├── todo.repository.ts
├── todos.controller.ts
├── todos.module.ts
└── todos.service.ts
```



## <strong>#</strong> 파라미터 및 반환값 Mockup

먼저 테스트 코드에서 자주 쓰일 데이터들을 mocking해두도록 하겠습니다. 별도의 `mock-todo.ts` 파일을 생성해 작성하도록 하겠습니다. 이 데이터들은 테스트 코드에서 자주 호출되며, 더미 데이터 역할을 담당합니다.

- `makeMockCreateTodoDto`: `createTodoDto`의 mocking 데이터를 반환합니다.

  ```typescript
  /* src/todos/specs/mock-todo.ts */
  export const makeMockCreateTodoDto = (): CreateTodoDto => ({
    content: 'Be Happy.',
    status: 'NOT_DONE',
  });
  ```

- `makeMockTodo`: `todo entity`의 mocking 데이터를 반환  

   (함수 내부에서 시간 선언을 하면 시간 차이 때문에 테스트를 실패하는 경우가 있기 때문에 함수 밖으로 밀어냅니다.)

  ```typescript
  /* src/todos/specs/mock-todo.ts */
  export const makeMockTodo = (now: Date): Todo => ({
    id: 1,
    content: 'Be Happy.',
    status: 'NOT_DONE',
    createdAt: now,
    deletedAt: now,
  });
  ```




## <strong>#</strong> 컨트롤러 테스트

Toto를 생성하는 API 핸들러에 대한 테스트입니다.

`nest g ~~~`  커맨드를 통해 컨트롤러를 생성하셨다면, `~~controller.spec.ts` 라는 파일이 함께 생성됩니다. 테스트 전에 컨트롤러의 로직에만 집중할 수 있도록 우선 module과 service를 Mocking해 의존성을 끊어주도록 하겠습니다.

### <strong>##</strong> 모듈 및 서비스 Mockup

```typescript
/* todos.controller.spec.ts */

// todoService의 Mocking class
class MockTodoService {
  createTodo = jest.fn();
}

describe('TodosController', () => {
  let todosController: TodosController;
  let todosService: TodosService;

  beforeEach(async () => {
    const module: TestingModule = await Test.createTestingModule({
      controllers: [TodosController],
      // todoService의 class를 위에 선언한 mocking class로 대체
      providers: [
        {
          provide: TodosService,
          useClass: MockTodoService,
        },
      ],
    }).compile();

    todosController = module.get<TodosController>(TodosController);
    todosService = module.get<TodosService>(TodosService);
  });

// ...
```

내장된 `@nest/testing`을 통해 테스트 코드에서 사용될 module을 mocking합니다.

꽤 헤맸던 부분은 서비스를 mocking하는 방법이었습니다. 먼저 실제 실제 `@Module` 데코레이터의 옵션과 같이 `controllers`와 `providers` 를 작성합니다. 이때, 서비스는 축약형으로 작성하지 않고 상단에 선언한 `MockTodoService` 를 적어주도록 합니다. jest.fn()을 통해 일일히 실제 `TodoService`의 메소드를 mocking해야 한다는 단점이 있습니다. 흑흑

### <strong>##</strong> 컨트롤러 테스트 코드

`createTodo()`에 대한 테스트 케이스는 다음과 같이 정리하겠습니다.

- 정확한 값과 함께 Todoservice의 createTodo를 호출해야한다.
- TodoService가 예외를 던지면, 별도의 처리없이 처리없이 던져야 한다.
- 성공 시, 생성된 Todo를 반환한다.


```typescript
/* todos.controller.spec.ts */
class MockTodoService {
  createTodo = jest.fn();
}

describe('TodosController', () => {
  let todosController: TodosController;
  let todosService: TodosService;

  beforeEach(async () => {
    const module: TestingModule = await Test.createTestingModule({
      controllers: [TodosController],
      providers: [
        {
          provide: TodosService,
          useClass: MockTodoService,
        },
      ],
    }).compile();

    todosController = module.get<TodosController>(TodosController);
    todosService = module.get<TodosService>(TodosService);
  });

  it('should be defined', () => {
    expect(todosController).toBeDefined();
  });
  describe('createTodo', () => {
    it('정확한 값과 함께 TodoService의 createTodo를 호출해야 한다.', async () => {
      const createSpy = jest.spyOn(todosService, 'createTodo');
      const mockParam = makeMockCreateTodoDto();
      await todosController.createTodo(mockParam);
      expect(createSpy).toHaveBeenCalled();
    });

    it('TodoService가 예외를 던지면, 별도의 처리없이 처리없이 던져야 한다.', async () => {
      const createSpy = jest
        .spyOn(todosService, 'createTodo')
        .mockRejectedValueOnce(new Error());
      await expect(
        todosController.createTodo(makeMockCreateTodoDto()),
      ).rejects.toThrow(new Error());
    });

    it('성공 시, 생성된 Todo를 반환한다.', async () => {
      const mockReturned = makeMockTodo();
      jest
        .spyOn(todosService, 'createTodo')
        .mockResolvedValueOnce(mockReturned);
      const response = await todosController.createTodo(
        makeMockCreateTodoDto(),
      );
      expect(response).toStrictEqual(mockReturned);
    });
  });
});
```

jest의 `spyOn()`을 통해 fn()으로만 선언해두었던 서비스의 `createTodo()`를 케이스 별로 mocking합니다. spyOn()은  몰래 숨어서 실제 호출을 추적할 수 있어, 해당 메소드가 실제 호출되었는지 판단하는  `toHaveBeenCalled()` 등의 함수를 사용할 수 있습니다.

그리고 Nest.js 각 레이어들의 메소드의 다수가 `Promise<>` 객체를 반환하는 일이 잦습니다. 잊지말고 `expect()` 함수 뒤에 `rejects`, `resolves` 을 붙여주세요.

![controller_test_with_error](/assets/images/posts_img/nest/controller_test_with_error.png)
열심히 달려왔지만 속상하게도 위와 같이 수많은 에러가 발생합니다. 이 에러는 대부분 우리가 아직 구현하지 않아 발생한 에러들이기 때문에 구현을 하며 이 에러를 처리하도록 하겠습니다.

### <strong>##</strong> 컨트롤러 구현

```typescript
/* todos.controller.ts */
@Controller('todos')
export class TodosController {
  constructor(private readonly todoService: TodosService) {}

  @Post('')
  async createTodo(createTodoDto: CreateTodoDto) {
    return this.todoService.createTodo(createTodoDto);
  }
}
```

먼저 컨트롤러를 구현합니다. 생성자 함수를 통해 `todoService`의 의존성을 주입받고, `@Post` 데코레이터를 달고 있는 `createTodo()` 메소드를 구현합니다.

인터셉터도, 가드도, 파이프도 없어 상당히 왜소해보입니다.

그리고  `todoService.createTodo` 를 구현하지 않아, 다음과 같이  여전히 에러가 발생합니다. 테스트 모듈을 생성할 때, `TodoService`를  참조하고, 이 서비스를 우리가 선언한 `MockTodoService`로 대체하게 됩니다. 우리는 `MockTodoService` 를 작성했고, 내부에 `createTodo()` mocking 함수를 작성했지만, 참조하는 실제 서비스에는 이 메소드가 구현되어 있지 발생하는 에러입니다.

![controller_test_with_error_02](/assets/images/posts_img/nest/controller_test_without_error_02.png)

이를 해결하기 위해 `todos.service.ts` 를 생성하고 내부에 `createTodo()` 메소드를 구현합니다.

### <strong>##</strong> 서비스 구현 (껍데기만 구현)

컨트롤러 테스트에 집중하기 위해 createTodo() 내부 로직은 현재 단계에서 구현하지 않겠습니다.

```typescript
/* todos.service.ts */
@Injectable()
export class TodosService {
  async createTodo(createTodoDto: CreateTodoDto) {
    return await Promise.resolve(); // 프로미스 객체 리턴
  }
}
```

껍데기만 갖고 있는 `TodoService` 를 만들었습니다. 생성자 함수도 없고, `createTodo()` 또한 내부를 구현하지 않았습니다. 컨트롤러 테스트에서 우리는 이 서비스를  mocking 해두었기 때문에 현재 상태로 컨트롤러의 유닛 테스트가 정상적으로 돌아갈 것이라 믿습니다.

`todos.controller.spec.ts` 의 에러가 사라진 것을 확인할 수 있습니다 .드디어 컨트롤러 테스트 코드를 실행시킬 준비를 마쳤습니다.

![controller_test_without_error](/assets/images/posts_img/nest/controller_test_without_error.png)

### <strong>##</strong> 컨트롤러 테스트 코드 실행

테스트 코드를 실행시키는 방법은 크게 아래와 같이 2가지로 분류할 수 있습니다.

#### 1. npm run test

`nest new ~~~` 를 통해 nest.js 프로젝트를 생성했다면,  `package.json` 에 다음과 같은 script를 확인할 수 있습니다.

```json
// package.json
"scripts": {
	// ...
  "test": "jest",
  "test:watch": "jest --watch",
  "test:cov": "jest --coverage",
  "test:debug": "node --inspect-brk -r tsconfig-paths/register -r ts-node/register node_modules/.bin/jest --runInBand",
  "test:e2e": "jest --config ./test/jest-e2e.json"
},
```

테스트 목적에 맞춰서 다양한 스크립트를 사용할 수 있습니다. 우리는 상세 정보를 볼 수 있는  `test:watch` 스크립트를 사용하도록 하겠습니다. 터미널을 열고 스크립트를 실행합니다.

```sh
$ npm run test:watch
```

테스트가 정상적으로 진행됐고, 너무나도 다행히 결과도 모두 성공한 것을 확인할 수 있습니다.

![controller_test_result_script](/assets/images/posts_img/nest/controller_test_result_script.png)

#### 2. 에디터 내장 테스트

저는 webstorm 을 사용하고 있기 때문에 webstorm 기준으로 작성하겠습니다.

`todos.controller.spec.ts` 파일을 열고 메뉴의 `Run` > `Run 'todos.controller.spec.ts'` 를 클릭합니다. 단축키는 `⌃` + `r` 라고 알려줍니다.

![controller_test_run_editor](/assets/images/posts_img/nest/controller_test_run_editor.png)

또 테스트 케이스 별로 실행 시키기 위해서는 line 표시 우측에 실행 버튼을 클릭하셔도 됩니다. 이 단축키는 커서가 해당 테스트 케이스에 존재한 상태에서 `⌃` + `⇧` +  `r` 를 입력하시면 됩니다.

하단의 `Run` 탭에 테스트 결과가 나타납니다. 마찬가지로 테스트가 정상적으로 진행됐고, 결과도 모두 성공인 것을 알 수 있습니다.

![controller_test_result_editor](/assets/images/posts_img/nest/controller_test_result_editor.png)



## <strong>#</strong> 마무리

지금까지 nest.js에서 컨트롤러 테스트를 진행하는 방법을 알아봤습니다. TDD의 3단계인 리팩토링은 진행하지 못했지만, 실패하는 테스트 코드를 먼저 작성하고, 이를 통과할 수 있도록 컨트롤러를 구현했습니다.

다음 포스트에서는 서비스의 테스트 케이스를 작성하고, 이를 구현해보도록 하겠습니다.





---

## 📚 참조

[https://github.com/marcosvcorsi/nestjs-tdd](https://github.com/marcosvcorsi/nestjs-tdd)

[https://github.com/JHyeok/nestjs-api-example](https://github.com/JHyeok/nestjs-api-example)

[https://jhyeok.com/nestjs-unit-test/](https://jhyeok.com/nestjs-unit-test/)

[https://devkly.com/nodejs/simple-tdd-with-nestjs/](https://devkly.com/nodejs/simple-tdd-with-nestjs/)
