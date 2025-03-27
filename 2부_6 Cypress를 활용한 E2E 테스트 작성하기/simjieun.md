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

## 3. 서버 요청 가로채기
- cy.intercept()를 사용해 네트워크 쵸엉을 가로채 응답을 조작할수 있다.
- E2E 테스트에서 모킹이 유용한 경우
  - 워크플로우 실행 중 실패 케이스를 검증하는 경우
  - 워크플로우 내에서 서드파티 API나 외부 앱을 사용하는 경우
```javascript
it('성공적으로 회원 가입이 완료되었을 경우 "가입 성공!"문구가 노출되며 로그인 페이지로 이동한다', () => {
  // 회원 삭제 불가
  // -> 회원 데이터가 계속 쌓이고, E2E 테스트에서도 중복되지 않는 계정 정보로 업데이트 필요
  // -> 이슈 해결을 위해 등록 API를 스터빙하여 테스트 진행

  // stubbing: 특정 네트워크 요청에 대해 미리 정해진 응답을 반환하는 것
  // intercept API는 요청과 응답에 대한 호출도 기록 -> stubbing과 spying을 모두 실행할 수 있음
  cy.intercept('POST', 'http://localhost:3000/users', { statusCode: 200 });

  cy.findAllByLabelText('이름').type('Maria');
  cy.findAllByLabelText('이메일').type('maria@mail.com');
  cy.findAllByLabelText('비밀번호').type('12345');
  cy.findByText('가입').click();

  cy.findByText('가입 성공!').should('exist');
  cy.assertUrl('/login');
});

it('회원 가입이 실패했을 경우 "잠시 문제가 발생했습니다! 다시 시도해 주세요." 문구가 노출된다', () => {
  cy.intercept('POST','http://localhost:3000/users', { statusCode: 401 });

  cy.findAllByLabelText('이름').type('Maria');
  cy.findAllByLabelText('이메일').type('maria@mail.com');
  cy.findAllByLabelText('비밀번호').type('12345');
  cy.findByText('가입').click();

  cy.findByText('잠시 문제가 발생했습니다! 다시 시도해 주시요.').should('exist');
});
```

## 4. E2E 테스트 작성하기 - 구매 페이지
```javascript
beforeEach(() => {
  cy.login();
  cy.getProductCardByIndex(0).findByText('장바구니').click();
  cy.getProductCardByIndex(1).findByText('장바구니').click();

  cy.visit('/purchase');
});

describe('배송 정보', () => {
  // 1. 첫번째 테이블 요소 조회(배송 정보 영역)
  // 2. 배송 정보 테이블의 행을 shippingList alias로 지정 -> 모든 테이블 행 요소를 shippingList로 접근할 수 있음
  beforeEach(()=>{
    cy.findAllByRole('table').eq(0).findAllByRole('row').as('shippingList');
  })

  it('로그인한 사용자의 이름 "Maria"가 입력되어 있다.', () => {
    // 3. shippingList 요소의 첫번째 테이블 행의 텍스트 입력 필드 조회
    // 4. 텍스트 필드 요소에 Maria가 입력되었는지 단언
    cy.get('@shippingList').eq(0).findByRole('textbox').should('have.value', 'Maria');
  });

  it('할인 쿠폰을 선택하지 않은 경우 "없음"이 노출되며, 특정 할인 쿠폰을 선택하면 해당 쿠폰의 이름("가입 기념! $5 할인 쿠폰")이 노출된다', () => {
    // 1. 없음 버튼 클릭 -> 드롭다운 열기
    // 2. 가입기념! $5 할인 쿠폰 클릭
    cy.get('@shippingList').eq(0).findByRole('button', { name: '없음' }).click();
    cy.findByText('가입 기념! $5 할인 쿠폰').click();

    // 3. 배송 정보 테이블에 가입 기념! $5 할인 쿠폰이 노출 단언
    cy.get('@shippingList').eq(2).findByText('가입 기념! $5 할인 쿠폰').should('exist');
  });
});

// 구매 -> 카드나 페이 정보 등록하여 결제
// 결제 정보를 모두 등록하여 외부 PG사와 연동 -> 테스트 환경에서는 어려움
// 테스트 종료 후 결제 내역 환불 or 실결제가 되지 않는 별도의 테스트용 플로우 필요 -> 현실적으로 무리
// 구매하기 API 응답을 스터빙하여 구매 프로세스 테스트 진행
it('구매하기 성공시 장바구니는 초기화 되고 메인홈 페이지로 이동하며, "구매 성공!" 메세지가 노출된다', () => {
  // 스터빙
  cy.intercept('POST', 'http://localhost:3000/purchases', { statusCode: 200 });

  cy.findAllByRole('table').eq(0).findAllByRole('row').as('shippingList');

  cy.get('@shippingList').eq(1).findByPlaceholderText('주소를 입력하세요').type('Seoul');
  cy.get('@shippingList').eq(3).findByPlaceholderText('휴대폰 번호를 입력하에쇼').type('010-1234-5678');

  cy.findByText('구매하기').click();

  cy.assertUrl('/');
  cy.findByText('구매 성공').should('exist');
  cy.getCartButton().should('have.text', '');
});

it('구매 실패시 "잠시 문제가 발생했습니다! 다시 시도해 주세요."라고 경고 문구가 노출된다', () => {
  cy.intercept('POST', 'http://localhost:3000/purchases', { statusCode: 500 });

  cy.findAllByRole('table').eq(0).findAllByRole('row').as('shippingList');

  cy.get('@shippingList').eq(1).findByPlaceholderText('주소를 입력하세요').type('Seoul');
  cy.get('@shippingList').eq(3).findByPlaceholderText('휴대폰 번호를 입력하에쇼').type('010-1234-5678');

  cy.findByText('구매하기').click();
  cy.findByText('잠시 문제가 발생했습니다! 다시 시도해 주세요.').should('exist');
});
```

## 5. E2E 테스트 작성하기 - 권한 체크
- 적을만한게 없음....또르르.....
