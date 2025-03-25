# Cypress를 활용한 E2E 테스트 작성하기

## 1. 커스텀 커맨드와 쿼리
- 반복 로직을 명령으로 커맨드와 쿼리를 활용한다.
- 커스텀 쿼리
  - Retry-ability 지원
  - 동기로 동작하며, subject 결과를 받아 내부적으로 체이닝 코드를 재시도한다
  - Cypress.Commands.addQuery()와 Cypress.Commands.overwriteQuery()로 정의
- 커스텀 커맨드
  - Retry-ability 미지원
  - 비동기로 동작할 수 있으며, 특별한 설정이 없다면 subject를 이어받지 않는다.
  - Cypress.Commands.add()와 Cypress.Commands.overwrite()로 정의

## 2. E2E 테스트 작성하기 - 메인 홈 페이지
```javascript
// 통합 테스트에서는 msw를 사용한 API 모킹
// E2E 테스트에서는 실제 API까지 호출하여 검증 -> 앱 사용 시나리오를 완벽하게 검증
it('초기 상품은 20개가 노출된다', () => {
  assertProductCardLength(20);
});

it('show more 버튼을 클릭할 경우 상품이 20개 더 노출된다', () => {
  // Show more 란 버튼 요소를 찾아 클릭
  // 20개의 상품 카드가 추가되어 총 40개 상품 카드 렌더링
  cy.findByText('Show more').click();

  assertProductCardLength(40);
});

describe('장바구니 / 구매 버튼', () => {
  // 통합 테스트의 jsDOM 환경 : 실제로 로그인 페이지로 이동하는지 확인 불가
  // E2E 테스트 : 브라우저의 URL 주소가 로그인 페이지로 이동했는지 검증
  describe('로그인을 하지 않은 경우', () => {
    // 첫번째 상품 코드의 장바구니, 구매 버튼 클릭
    it('장바구니 버튼 클릭 시 로그인 페이지로 이동한다', () => {
      cy.getProductCardByIndex(0).findByText('장바구니').click();

      cy.assertUrl('/login');
    });

    it('구매 버튼 클릭 시 로그인 페이지로 이동한다', () => {
      cy.getProductCardByIndex(0).findByText('구매').click();

      cy.assertUrl('/login');
    });
  });

  describe('로그인 시', () => {
    beforeEach(() => {
      cy.login();
    });

    it('장바구니에 아무것도 추가하지 않은 경우 장바구니 아이콘 뱃지에 숫자가 노출되지 않는다', () => {
      // 카트 버튼을 조회
      // 버튼 하위 뱃지에 빈 텍스트가 렌더링 확인
      cy.getCartButton().should('have.text', '');
    });

    it('장바구니 버튼 클릭 시 "장바구니 추가 완료!" 알림 메세지가 노출되며, 장바구니에 담긴 수량도 증가한다', () => {
      // 첫번째, 두번째 상품 카드 클릭
      // 장바구니 추가 완료 문구 노출 단언
      // 카트 버튼의 백시에 수량이 증가하는 것 단언
      cy.getProductCardByIndex(0).findByText('장바구니').click();

      cy.findByText('Handmade Cotton Fish 장바구니 추가 완료!').should('exist');
      cy.getCartButton().should('have.text', '1');

      cy.getProductCardByIndex(1).findByText('장바구니').click();

      cy.findByText('Awesome Concrete Shirt 장바구니 추가 완료!').should('exist');
      cy.getCartButton().should('have.text', '2');
    });

    it('구매 버튼 클릭시 해당 아이템이 장바구니에 추가되며, 장바구니 페이지로 이동한다', () => {
      // 첫번째 상품 카드의 구매 버튼 클릭
      cy.getProductCardByIndex(0).findByText('구매').click();

      cy.assertUrl('/cart');
      cy.getCartButton().should('have.text', '1');
      cy.findByText('Handmade Cotton Fish').should('exist');
    });
  });
})
```
- cy.session()
  - 여러 테스트에서 공유할 수 있는 쿠키, local storage, session storage에 있는 정보를 캐싱하여 활용
  - E2E 테스트 실행 시간이 훨씬 감소
```javascript
import '@testing-library/cypress/add-commands';

Cypress.Commands.add('login', () => {
  const username = 'maria@mail.com';
  const password = '12345';

  // 쿠키, local storage, session storage에 있는 정보들을 캐싱
  // 콜백 함수 실행 전 -> 모든 도메인의 쿠키, 로컬 스토리지, 세션 스토리지 초기화
  // 초기화 진행 후 로그인 완료 -> 세션 정보 설정
  // 메인 홈페이지로 이동
  cy.session(username, () => {
    cy.visit('/login');

    cy.findByLabelText('이메일').type(username);
    cy.findByLabelText('비밀번호').type(password);
    cy.findByLabelText('로그인').click();

    cy.location('pathname').should('eq', '/');
  });

  // 로그인 이후 메인 홈페이지로 이동
  cy.visit('/');
});
```
