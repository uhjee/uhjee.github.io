---
title: '[Nest.js, Jest] Nest.js with TDD(2) - Service'
excerpt: 'TDD를 통해 todolist 로직 생성하기 - 서비스 편'

categories:
- nest
tags:
- [Typescript, node, test, unit test, jest, ts, nest, nest.js, TDD, spyOn, mocking, controller, service, mock, module]

permalink: /nest/nestjs-with-tdd-service/

toc: true
toc_sticky: true

date: 2022-08-28
last_modified_at: 2022-08-28 
---



이전 포스트에서 컨트롤러 테스트코드를 작성하고, 이를 바탕으로 구현을 진행했습니다.

이번 포스트에서는 서비스단의 테스트 코드를 작성하고, 실제 서비스 코드를 구현해보도록 하겠습니다. 전체적인 흐름은 컨트롤러와 다르지 않습니다. 

하단의 코드는 모두 [여기](https://github.com/uhjee/TIL/tree/master/nest_js/nest-tdd)에서 보실 수 있습니다.


## <strong>#</strong> 서비스 테스트 

`nest g service todos`를 통해 서비스를 생성하신 분들은 `todos.service.spec.ts`라는 파일이 이미 생성되어 있겠지만, 그렇지 않으신 분들은 `todos.service.spec.ts` 파일을 생성합니다.

서비스에서는 레포지토리 레이어에 대한 의존성을 주입받기 때문에 이 관계를 끊고자, 모듈 mocking을 하며 동시에 레포지토리도 mocking하도록 하겠습니다.

### <strong>##</strong> 모듈 및 레포지토리 Mockup

```typescript
/* todos.controller.spec.ts */

// todoRepository의 Mocking class
class MockTodoRepository {
  createTodo = jest.fn();
}

describe('TodosService', () => {
  let todoService: TodosService;
  let todoRepository: TodoRepository;

  beforeEach(async () => {
    const module: TestingModule = await Test.createTestingModule({
      providers: [
        TodosService,
        {
          provide: TodoRepository,
          useClass: MockTodoRepository,
        },
      ],
    }).compile();

    todoService = module.get<TodosService>(TodosService);
    todoRepository = module.get<TodoRepository>(TodoRepository);
  });

// ...
```

내장된 `@nest/testing`을 통해 테스트 코드에서 사용될 mock module을 생성합니다.

테스트 대상이 아닌 레포지토리는 Mock up하도록 하겠습니다. providers의 `TodoRepository`를 상단에 작성한 `MockTodoRepository`로 대체합니다.  `MockTodoRepository` 는 `createTodo` 메소드의 mocking 함수를 갖고 있도록 작성했습니다.

### <strong>##</strong> 서비스 테스트 코드

`createTodo()`에 대한 테스트 케이스는 다음과 같이 정리하겠습니다.

- 정확한 값과 함께 `Todoservice`의 `createTodo`를 호출해야한다.
- `TodoService`가 예외를 던지면, 별도의 처리없이 처리없이 던져야 한다.
- 파라미터로 받는 `createTodoDto`의 속성에 값이 빈 문자열일 경우, '[key[, key]]가 없습니다.'라는 메세지를 가진 에러를 발생시켜야 한다.
- 성공 시, 생성된 `Todo`를 반환한다.


```typescript
/* todos.service.spec.ts */
describe('TodosService', () => {
  let todoService: TodosService;
  let todoRepository: TodoRepository;

  // ...

  describe('createTodo()', () => {
    it('파라미터 정상 입력 시, 생성된 투두를 반환해야 함', () => {
      const mockTodo = makeMockTodo();
      jest.spyOn(todoRepository, 'createTodo').mockResolvedValue(mockTodo);

      expect(todoService.createTodo(makeMockCreateTodoDto())).resolves.toEqual(
        mockTodo,
      );
    });

    it('TodoRepository가 예외를 던지면, 별도의 처리없이 처리없이 던져야 한다.', async () => {
      jest.spyOn(todoRepository, 'createTodo').mockRejectedValue(new Error());

      await expect(
        todoService.createTodo(makeMockCreateTodoDto()),
      ).rejects.toThrow(new Error());
    });

    it("파라미터로 받는 createTodoDto의 속성에 값이 빈 문자열일 경우, '[key[, key]]가 없습니다.'라는 메세지를 가진 에러를 발생시켜야 한다.", async () => {
      const invalidCreateTodoDto = {
        content: '',
        status: '',
      };
      jest.spyOn(todoRepository, 'createTodo').mockRejectedValue(new Error());
      await expect(
        todoService.createTodo(invalidCreateTodoDto),
      ).rejects.toThrowError('content, status가 없습니다.');
    });

    it('정상적으로 todo를 생성하면, todo를 반환해야 한다.', async () => {
      const mockReturned = makeMockTodo();
      jest.spyOn(todoRepository, 'createTodo').mockResolvedValue(mockReturned);

      const response = await todoService.createTodo(makeMockCreateTodoDto());

      expect(response).toEqual(mockReturned);
    });
  });
});

```


내부에서 사용하는  `makeMockTodo()`, `makeMockCreateTodoDto()`는 컨트롤러 테스트 진행 시 만들어 놓은 함수입니다. [이 포스트](https://uhjee.github.io/nest/nestjs-with-tdd-controller/)에서 확인하실 수 있습니다.

서비스 테스트 코드를 작성했습니다. 컨트롤러 테스트 코드와 마찬가지로 jest의 `spyOn()`을 통해 fn()으로만 선언해두었던 서비스의 `createTodo()`를 케이스 별로 mocking했습니다. 실제 `createTodo()`는  promise를 반환하기 때문에 이에 유의하며 코드를 작성합니다.

에러 메세지를 검증하기 위해 `try...catch`문이 아닌 expect에서 제공하는 `toThrowError()`를 사용했습니다. 

![service_test_with_error](/assets/images/posts_img/nest/service_test_with_error.png)
아직 서비스를 구현하지 않아, 위와 같이 오류가 발생합니다. 에러가 발생하지 않도록, 테스트 코드를 정상적으로 실행시킬 수 있도록 서비스를 구현해봅시다.

### <strong>##</strong> 서비스 구현

이전 포스트에서 아래와 같이 `todos.service.ts` 의 코드를 작성했습니다. 실제 로직은 없고, 단순히 파라미터와 Promise 리턴 값만 갖는 껍데기입니다.

```typescript
* todos.service.ts */
@Injectable()
export class TodosService {
  async createTodo(createTodoDto: CreateTodoDto) {
    return await Promise.resolve(); // 프로미스 객체 리턴
  }
}
```

이전에 작성해둔 껍데기에 repository의 의존성을 갖도록 `constructor()`를 작성합니다. 

이후 우리가 테스트할 대상인 `createTodo()` 메소드를 작성합니다. 통상적으로 dto의 validation은 nest의 pipe를 통해 처리하지만, 우리는 로직을 좀 더 복잡하게 작성하기 위해 서비스 내부에서 validation을 진행하도록 하겠습니다.

```typescript
@Injectable()
export class TodosService {
  constructor(
    @Inject(TodoRepository)
    private readonly todoRepository: TodoRepository,
  ) {}

  async createTodo(createTodoDto: CreateTodoDto) {
    // dto validation
    const invalidProps = [];
    Object.entries(createTodoDto).forEach(([key, value]) => {
      if (!value) invalidProps.push(key);
    });
    if (invalidProps.length > 0) {
      const invalidPropsString = invalidProps.reduce((result, value, index) => {
        let text = result + value;
        if (index !== invalidProps.length - 1) {
          text += ', ';
        }
        return text;
      }, '');
      throw new BadRequestException(`${invalidPropsString}가 없습니다.`);
    }
    return await this.todoRepository.createTodo(createTodoDto);
  }
}
```

서비스는 구현을 했지만, `TodoRepository`를 구현하지 않아 아직 에러가 발생합니다. TodoRepository를 작성해봅시다.

```typescript
/* todos.repository.ts */
/**
 * [CUSTOM REPOSITORY]
 */

@Injectable()
export class TodoRepository extends Repository<Todo> {
  constructor(private dataSource: DataSource) {
    super(Todo, dataSource.createEntityManager());
  }

  async createTodo(createTodoDto: CreateTodoDto) {
    return await this.save(createTodoDto);
  }
}

```

레포지토리는 평소대로 custom repository로 작성했습니다. 트랜잭션으로 묶어줘야 하는 경우가 아니라면 최대한 service에서 직접 DB접근을 하지 않기 위함입니다.

이제 다시 확인을 해보면 `todos.service.spec.ts` 의 에러가 사라졌습니다. 모든 에러를 없앴으므로 서비스 테스트 코드를 실행시켜봅니다.

### <strong>##</strong> 서비스 테스트 코드 실행

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

이 커맨드는 jest가 관리하는 프로젝트 전체의 테스트 코드를 실행하므로, 이번에는 `todos.service.spec.ts` 파일만 실행시켜보도록 하겠습니다.

```sh
$ npx jest --watch ./src/todos/specs/todos.service.spec.ts    
```

모든 테스트가 정상적으로 실행된 것을 알 수 있습니다.

![service_test_result_script](/assets/images/posts_img/nest/service_test_result_script.png)

#### 2. 에디터 내장 테스트

저는 webstorm 을 사용하고 있기 때문에 webstorm 기준으로 작성하겠습니다.

`todos.service.spec.ts` 파일을 열고 메뉴의 `Run` > `Run 'todos.service.spec.ts'` 를 클릭합니다. 단축키는 `⌃` + `r` 라고 알려줍니다.

![service_test_run_editor](/assets/images/posts_img/nest/service_test_run_editor.png)

또 테스트 케이스 별로 실행 시키기 위해서는 line 표시 우측에 실행 버튼을 클릭하셔도 됩니다. 이 단축키는 커서가 해당 테스트 케이스에 존재한 상태에서 `⌃` + `⇧` +  `r` 를 입력하시면 됩니다.

하단의 `Run` 탭에 테스트 결과가 나타납니다. 마찬가지로 테스트가 정상적으로 진행됐고, 결과도 모두 성공인 것을 알 수 있습니다.

![service_test_result_editor](/assets/images/posts_img/nest/service_test_result_editor.png)



## <strong>#</strong> 마무리

지금까지 nest.js에서 서비스 테스트를 진행해보았습니다. 예제로 작성한 코드가 복잡한 코드가 아니기 때문에, 실무에서는 mocking 작업도 테스트 코드도 훨씬 복잡해지겠지만, 테스트 코드를 미리 작성하면서 개발하고자 하는 로직에 대해 더 깊고 넓게 생각할 수 있는 경험을 얻었습니다. 

TDD가 익숙해지면 개발해야 할 기능에 대해 명확하게 이해할 수 있고, 각 함수들의 역할과 책임에 대해 효율적으로 분리할 수 있을 것이라 생각됩니다.

앞으로 사이드 프로젝트에서 다양한 테스트 코드를 작성해보고, 나아가 실무에서도 작성할 수 있도록 해보겠습니다.

긴 글 읽어주셔서 감사합니다.





---

## 📚 참조

[https://github.com/marcosvcorsi/nestjs-tdd](https://github.com/marcosvcorsi/nestjs-tdd)

[https://github.com/JHyeok/nestjs-api-example](https://github.com/JHyeok/nestjs-api-example)

[https://jhyeok.com/nestjs-unit-test/](https://jhyeok.com/nestjs-unit-test/)

[https://devkly.com/nodejs/simple-tdd-with-nestjs/](https://devkly.com/nodejs/simple-tdd-with-nestjs/)

[https://jojoldu.tistory.com/656](https://jojoldu.tistory.com/656)
