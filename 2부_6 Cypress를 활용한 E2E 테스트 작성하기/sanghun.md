# 6.1 커스텀 커맨드와 쿼리

#### 커스텀 쿼리

- Retry-ability 지원
- 동기로 동작, subject 결과를 받아 내부적으로 체이닝 코드를 재시도

#### 커스텀 커맨드

- Retry-ability 미지원
- 비동기로 동작, 특별한 설정이 없으면 subject를 이어받지 않음

## 6.2 E2E 테스트 작성하기 - 홈페이지

- cy.session()을 사용해 쿠키, local storage, session storage에 있는 정보를 캐싱
- E2E 테스트에서 사용되는 로그인 시간을 단축시켜줌

```javascript
describe('필터', () => {
  it('상품명을 "Handmade Cotton"로 입력하면 해당 상품명을 포함한 상품만 나타난다', () => {
    cy.findByLabelText('상품명').type('Handmade Cotton');

    assertProductCardLength(2);
    assertProductCardText('Handmade Cotton Fish', 0);
    assertProductCardText('Handmade Cotton Keyboard', 1);
  });

  it('카테고리를 "Shoes"로 선택할 경우 해당 카테고리 상품만 나타난다', () => {
    cy.findByRole('radio', { name: 'Shoes' }).click();

    cy.findAllByTestId('product-card').each(($el) => {
      cy.wrap($el).findByText('Shoes').should('exist');
    });
  });

  it('최소 가격을 "15", 최대 가격을 "20"로 입력한 경우 해당 금액 사이에 있는 상품이 노출된다', () => {
    cy.findByPlaceholderText('최소 금액').type('15');
    cy.findByPlaceholderText('최대 금액').type('20');

    assertProductCardLength(1);
    assertProductCardText('$19.00');
  });

  it('상품명 "Handmade", 카테고리 "Shoes", 최소 금액 "750", 최대 금액 "800"로 입력하면 모든 조건을 충족하는 상품만 노출된다', () => {
    cy.findByLabelText('상품명').type('Handmade');
    cy.findByLabelText('Shoes').click();
    cy.findByPlaceholderText('최소 금액').type('750');
    cy.findByPlaceholderText('최대 금액').type('800');

    assertProductCardLength(1);
    assertProductCardText('Handmade Soft Chicken');
    assertProductCardText('Shoes');
    assertProductCardText('$769.00');
  });
});
```

## 6.3 서버 요청 가로채기

- 예외적으로 목킹이 필요한 api는 cy.intercept()를 사용한다.
- 특정 데이터를 등록하는 경우, 삭제과정까지 같이 검증하는 것이 좋음
  - 불필요한 데이터 쌓임 방지
    > 비교견적 예시로 차량번호 검색 및 신청 후 임시저장 데이터 혹은 신청데이터를 삭제하는 api를 afterEach로 적용
- 데이터 삭제가 어려울 경우 API 모킹을 통해 실제 데이터가 생성되지 않도록 하는것이 좋음

## 6.4 E2E 테스트 작성하기 - 구매페이지

- alias를 사용하면 편리하게 테이블 요소 조회 가능

```javascript
cy.findAllByRole('table').eq(2).findAllByRole('row').as('paymentList');
cy.findAllByRole('table').eq(0).findAllByRole('row').as('shippingList');
```

- 결제처럼 외부 앱을 사용해 테스트 환경을 만들기 어려운 경우 API 스터빙 사용
  > response가 있는 api의 경우 스터빙 방법은?

## 6.5 E2E 테스트 작성하기 - 권한 체크

- 권한체크를 통해 페이지 접근제한하는 코드가 있다면 E2E로 검증하기 좋음
  - 간단한 방법으로 체크 가능

## 6.6 E2E 테스트의 한계

- E2E테스트는 다른 테스트보다 많은것을 검증해야해서 시간이 오래걸림
- 디버깅... 지원환경... 등의 문제

## 6.7 테스트 더블

- 더미
- 스텁(Stub) : 이미 정해진 응답값을 반환하도록 구현
  - 고려된 케이스 이외에는 대응이 불가능
- 스파이(Spy) : 스텁의 고도화, **구현된 객체의 호출정보**까지 기록 (상태검증)
- 목(Mock) : 실제모듈과 같은 행동을 하도록 정의된 모의 객체 (행동검증)
- 페이크(Fake) : 특정 모듈을 아주 단순한 테스트 전용 모듈로 대체

### 테스트 더블의 장점

- 예외 처리에대한 상황을 가짜로 만들어 검증 가능함
