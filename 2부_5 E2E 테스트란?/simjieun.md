# 5. E2E 테스트란?

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
