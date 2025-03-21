# 5. E2E 테스트란?

## 5.1 E2E 테스트란 무엇일까?
- 실제로 앱을 실행해 전체 소프트웨어 시스템의 전체 흐름을 검증하는 것이다.
- 사용자 입장에서 앱을 사용하면서 발생할 수 있는 시나리오가 실제 환경에서 정상적으로 작동하는 확인한다.
- 직접 앱을 구동하기에 다른 테스트들에 비해 시간이 오래 걸린다.

### E2E 테스트의 장점
- 사용자 관점에서 시나리오를 완벽하게 테스트할 수 있다.
- 프런트엔드부터 백엔드까지 앱의 전반적인 상태를 확인할 수 있다.
- 변경 사항이 전체 시스템에 미치는 영향을 확인할 수 있다.

### E2E 테스트 작성시 중요한 점
- 유관 부서와의 협력이 필요하다.
- 가능한 한 API는 모킹하지 않는다.
- 도입을 위한 일정이 확보되어야 한다.

## 5.2 Cypress로 E2E 테스트 시작하기
- Cypress는 자바스크립트를 사용하여 실제 웹앱에서 다양한 테스트를 작성할 수 있는 오픈 소스 자동화 도구이다.
- https://www.cypress.io/

### Head 모드와 Headless 모드
- Head 모드
  - cypress open
  - 브라우저 UI까지 모두 구동하여 시작적으로 확인할 수 있는 환경에서 테스트를 실행한다.
  - 주로 실행 과정을 확인하거나 디버깅 시에 사용한다.
- Headless 모드
  - cypress run
  - UI없이 브라우저 엔진을 명령어 인터페이스로 제어하여 테스트를 실행한다. 
  - 구동속도가 상대적으로 빨라 CI 또는 클라우드 환경에서 사용한다.

## 5.3 Cypress로 E2E 테스트 작성하기
> 단위, 통합 테스에서 작성한 테스트를 E2E에서도 작성해야하나요?
> - 컴포넌트 일부가 아닌 앱 전체가 구동되었을 때도 동일하게 기능이 동작하는 검증하기 위해서다.
> - 통합테스트에서 특정 기능을 검증했는지 파악하는 노력을 한 후에 E2E테스트에서 다시 해당 기능을 제외하고 작성하는 과정을 추가하는 것이 DX의 큰장점이 없다고 생각하기 때문이다.
> - 이런 이유로 중복되더라도 E2E 테스트에서도 워크플로우에 필요한 기능을 검증하는것이 안정성과 DX 측면에서 좋다.

### Cypress의 단언
- 내부적으로 Chai, Chai-jQuery, Sinon-Chai의 문법을 사용할수 있도록 확장되어 있다.
- Chai-jQuery는 주로 DOM 객체에 대한 단언이다.
- Sinon-Chai는 주로 스파이, 스텁, 목 객체에 대한 단언이다.

### Cypress 테스트 작성
```javascript
beforeEach(() => {
  // baseUrl을 기준으로 웹 페이지로 접속할 수 있도록 도와주는 함수
  cy.visit('/login');
});

it('이메일을 입력하지 않고 로그인 버튼을 클릭할 경우 "이메일을 입력하세요" 경고 메세지가 노출된다', () => {
  // Cypress test library -> 사용자가 앱을 사용하는 방식과 유사하게 요소 탐색 -> 신뢰성 있는 테스트 코드
  // Cypress test library -> findBy*, findAllBy*를 사용해야함 -> retry-ability 기능 사용
  cy.findByLabelText('로그인').click();

  cy.findByText('이메일을 입력하세요').should('exist');
});

it('비밀번호를 입력하지 않고 로그인 버튼을 클릭할 경우 "비밀번호를 입력하세요" 경고 메세지가 노출된다', () => {
  cy.findByLabelText('로그인').click();
  cy.findByText('비밀번호를 입력하세요').should('exist');
});

it('잘못된 양식의 이메일을 입력한 뒤 로그인 버튼을 클릭할 경우 "이메일 양식이 올바르지 않습니다" 경고 메세지가 노출된다', () => {
  cy.findByLabelText('이메일').type('joy');
  cy.findByLabelText('로그인').click();
  cy.findByText('이메일 양식이 올바르지 않습니다').should('exist');
});

it('회원 가입 클릭 시 회원 가입 페이지로 이동한다', () => {
  // E2E에서는 단위, 통합 테스트처럼 라우터 라이브러리를 모킹하는 작업이 없음
  // -> 실제로 페이지가 이동하는지 확인할 수 있기 때문에 실제 앱의 동작과 완전히 동일하게 검증할수있다.
  cy.findByText('회원가입').click();

  cy.url().should('eq', `${Cypress.env('baseUrl')}/register`);
});

it('성공적으로 로그인 되었을 경우 메인 홈 페이지로 이동하며, 사용자 이름 "Maria"와 장바구니 아이콘이 노출된다', () => {
  // 실제 로그인 API를 호출하여 백엔드까지 연동했을때 로그인의 성공 여부를 확인할 수 있다.
  // 메인 페이지로 이동하여 사용자 계정 정보가 나타냐는지 확인이 가능하다.
  const username = 'maria@mail.com';
  const password = '12345';

  cy.findAllByLabelText('이메일').type(username);
  cy.findByLabelText('비밀번호').type(password);
  cy.findByLabelText('로그인').click();

  cy.url().should('eq', `${Cypress.env('baseUrl')}/`);
  cy.findByText('Maria').should('exist');
  cy.findByTestId('cart-icon').should('exist');
});
```

## 5.4 Cypress와 쿼리

### subject
- Cypress에서는 커맨드를 Promise 체이닝을 통해 관리하며, 각 커맨드는 체인이 종료되거나 오류가 발생할 때 까지 멈춘다.
- 이런 동작을 제어하기 위해 내부적으로 다음 커맨드의 대상을 subject로 관리한다.
- 이 Subject 객체는 테스트의 시작 지점 또는 대상이 되는 요소를 의미한다.

### Retry-ability
- Cypress의 핵심기능으로 테스트에서 타이밍제어를 편리하게 할수 있도록 도와주는 함수  
- 시간변경하기 위해서는 cy.findByLabelText('이메일', { timeout: 10000 }) 이렇게 선언하면 된다.
- 기본 대기시간은 4초이며 defaultCommandTimeout으로 전역 재시도 시간을 변경할 수도 있다.
- 선행 동작이 실패하면 이후의 동작은 실행되지 않는다.

#### Retry-ability 기능 여부에 따른 API 구분
- Query : 전체 체이닝 로직을 재시도하며 수행 ex) get, cypress-testing-library, find등
- Assertion : 단언을 수행하는 특별한 쿼리의 유형 ex) should 등
- Non-Query : 재시도를 하지 않으며, 단한번만 실행되는 커맨드 ex) visit, click, then 등

```javascript
beforeEach(() => {
  cy.visit('/login');
  cy.findByLabelText('로그인').as('loginBtn');
});

it('이메일을 입력하지 않고 로그인 버튼을 클릭할 경우 "이메일을 입력하세요" 경고 메세지가 노출된다', () => {
  // get API -> Cypress에서 지정한 별칭(alias)로 선언한 요소에 접근 가능
  // 체이닝 형태로 테스트 코드 작성 -> 테스트의 과정을 이해하기 쉬움, 코드를 간결하게 작성할 수 있음
  cy.get('@loginBtn').click();
  cy.findByText('이메일을 입력하세요').should('exist');
});

it('비밀번호를 입력하지 않고 로그인 버튼을 클릭할 경우 "비밀번호를 입력하세요" 경고 메세지가 노출된다', () => {
  cy.findByLabelText('로그인').click();
  cy.findByText('비밀번호를 입력하세요').should('exist');
});

it('잘못된 양식의 이메일을 입력한 뒤 로그인 버튼을 클릭할 경우 "이메일 양식이 올바르지 않습니다" 경고 메세지가 노출된다', () => {
  // 1. 이메일 요소 조회 커맨드 실행 -> 완료되어야 type 커맨드로 subject가 넘어가 실행됨
  // 2. type 커맨드로 텍스트 입력
  cy.findByLabelText('이메일').type('joy');
  
  // 커맨드가 실행될때 각 subject는 내부적으로 비동기 대기열에서 대기하다가 실행
  // get API나 테스팅 라이브러리의 쿼리 실행이 완료되는 타이밍
  // -> subject 체이닝형태로 연속해서 커맨드를 실행 or then() API를 사용
    
  // then() API -> 이전 커맨드의 subject 실행 결과를 전달받아 사용
  // 체이닝 형태로만 subject의 결과를 전달받아 사용 가능 -> 실행 결과 전달을 yield 라고 표현
  cy.findByLabelText('로그인').click();
  
  // then 내부에서 반환하는 값은 새로운 subject가 되어 다음 커맨드에서 사용
  // 아무것도 반환하지 않을 경우 다음 커맨드에서는 이전 subject를 그대로 사용
  cy.findByLabelText('이메일')
      .then($email => {
          const cls = $email.attr('class');
          // ...
      })
      .click();
  
  cy.findByText('이메일 양식이 올바르지 않습니다').should('exist');
});

it('회원 가입 클릭 시 회원 가입 페이지로 이동한다', () => {
  cy.findByText('회원가입').click();

  cy.url().should('eq', `${Cypress.env('baseUrl')}/register`);
});

it('성공적으로 로그인 되었을 경우 메인 홈 페이지로 이동하며, 사용자 이름 "Maria"와 장바구니 아이콘이 노출된다', () => {
  const username = 'maria@mail.com';
  const password = '12345';

  cy.findAllByLabelText('이메일').type(username);
  cy.findByLabelText('비밀번호').type(password);
  cy.findByLabelText('로그인').click();

  cy.url().should('eq', `${Cypress.env('baseUrl')}/`);
  cy.findByText('Maria').should('exist');
  cy.findByTestId('cart-icon').should('exist');
});
```
