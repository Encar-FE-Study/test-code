# Cypress의 기본 동작

Cypress는 기본적으로 명령(Command)을 체이닝해서 실행

- 모든 명령은 Command Queue에 들어가서 순차적으로 실행됨 (Promise처럼 생겼지만 내부는 다름)
  -> Cypress는 테스트 코드를 작성하면, 그걸 즉시 실행하지 않고 내부적으로 Command Queue에 차곡차곡 쌓아놓고, 한 줄씩 순차적으로 실행
- 그리고 Cypress는 자동 재시도(Retry) 기능이 있는 명령과 없는 명령을 구분됨
- Retry-ability 지원 -> 커스텀 쿼리 /동기로 동작, subject 결과를 받아 내부적으로 체이닝 코드를 재시도
- Retry-ability 미지원 -> 비동기로 동작, 특별한 설정이 없으면 subject를 이어받지 않음

### 동기 / 비동기 비교

| 용어                 | 동기적(Synchronous)                           | 비동기적(Asynchronous)                                                                   |
| -------------------- | --------------------------------------------- | ---------------------------------------------------------------------------------------- |
| Cypress기준에서 의미 | Cypress 명령이 즉시 값을 반환한다 (return 값) | Cypress 명령이 즉시 값을 반환하지 않고, 나중에 Cypress가 실행할 수 있게 큐에 등록만 한다 |

```
커스텀 쿼리는 Cypress가 내부에서 DOM 요소를 찾는 쿼리 명령처럼 동작하게 함 → .get(), .find() 같은 느낌
Cypress는 이 쿼리를 실패하면 알아서 재시도 함 (ex: DOM 아직 안 나왔을 때)
커스텀 커맨드는 Cypress 명령 큐에 "행동"을 넣는 명령어를 추가함
단, 이 명령 자체는 실패해도 Cypress가 재시도하지 않음

```

```
쿼리는 값을 조회해서 넘겨주기 위한 함수
커맨드는 일련의 행동을 실행하는 함수
커스텀 쿼리는 내부적으로 값이 나올 때까지 Cypress가 계속 재시도함
커스텀 커맨드는 한 번 실행되고 끝남, 재시도 로직을 넣고 싶으면 내부에서 넣어줘야 함
```

# 커스텀 쿼리 vs 커스텀 커맨드

### 공통 목적

- 반복 로직을 Cypress 명령으로 분리해 재사용성/유지보수성 높이기
- 로직을 묶어서 추상화할 수 있음
```
커맨드
Cypress.Commands.addQuery('getProductCardByIndex', (index) => {
  return () => {
    return Cypress.$('.product-card').eq(index);
  };
});

cy.get('.product-list .product-card').eq(3)를 매번 작성하려면 귀찮음
-> cy.getProductCardByIndex(3)로 처리 가능
-> getProductCardByIndex(1) 이 명령은 DOM에서 해당 요소를 찾아서 즉시 리턴 / 자동 재시도 됨

쿼리

Cypress.Commands.add('login', (username, password) => {
  cy.get('#username').type(username);
  cy.get('#password').type(password);
  cy.get('form').submit();
});

cy.login(어쩌구,저쩌구)
자동 재시도 안 됨 → cy.get() 자체는 재시도되지만, 전체 흐름 자체는 한 번 실행

```



| 항목                   | 커스텀 쿼리 (Custom Query)    | 커스텀 커맨드 (Custom Command) |
| ---------------------- | ----------------------------- | ------------------------------ |
| 정의 방법              | `Cypress.Commands.addQuery()` | `Cypress.Commands.add()`       |
| Retry-ability (재시도) | 지원                          | 미지원                         |
| 동기/비동기            | 동기적 (즉시 리턴)            | 비동기 가능 (Promise 사용)     |
| subject 체이닝         | subject 받아 체이닝 가능      | 기본적으로 subject 안 받음     |
| 실행 시점              | 즉시 실행 (결과 리턴)         | 커맨드 큐에 쌓여 순차 실행     |
| 사용 목적              | DOM 요소 조회 쿼리            | 행동 또는 프로세스 추상화      |
| 사용 예                | `getProductCardByIndex`       | `login`, `addToCart` 등        |

---

## 다른 테스트 환경에서는
### RTL
```
Cypress처럼 “명령어 등록”은 없지만, 유틸 함수로 추상화하는 방식으로 처리

test('로그인할 수 있다', async () => {
  render(<LoginForm />);

  await userEvent.type(screen.getByLabelText(/아이디/i), 'testuser');
  await userEvent.type(screen.getByLabelText(/비밀번호/i), '1234');
  await userEvent.click(screen.getByRole('button', { name: /로그인/i }));

  expect(screen.getByText(/환영합니다/i)).toBeInTheDocument();
});
=> 반복되는 로직이 생기면 Cypress처럼 커맨드를 등록할 순 없지만

유틸함수로 추상화
// test-utils/loginHelper.ts
import { screen } from '@testing-library/react';
import userEvent from '@testing-library/user-event';

export async function login(username: string, password: string) {
  await userEvent.type(screen.getByLabelText(/아이디/i), username);
  await userEvent.type(screen.getByLabelText(/비밀번호/i), password);
  await userEvent.click(screen.getByRole('button', { name: /로그인/i }));
}

및 재사용 가능
import { login } from '../test-utils/loginHelper';

test('유저 로그인 시 대시보드 이동', async () => {
  render(<LoginPage />);
  await login('tester', 'pass');
  expect(screen.getByText(/대시보드/i)).toBeInTheDocument();
});

```

# 테스트 더블

### 실제 의존 객체를 테스트를 위해 가짜로 대체한 것

→ 복잡한 의존성을 제어하고, 예외 케이스나 행동 검증을 쉽게 수행

- 실제 의존 객체를 "가짜로 바꿔서" 테스트 시 예외 상황이나 조건을 쉽게 제어할 수 있도록 만든 것

자주쓰는 것
Stub /Spy /Mock
-> 비동기 로직 테스트 + 외부 API 의존 제거 + 예상 시나리오 제어에 가장 자주 필요하기 때문
