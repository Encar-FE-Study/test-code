# 1. 통합 테스트란 무엇일까?

## 프론트엔드 통합 테스트에서 적절한 범위 설정

- 기능 중심 범위 설정

사용자 동작 기반: "사용자가 상품을 장바구니에 추가하고 결제하는 과정"과 같은 기능 중심으로 범위 설정
비즈니스 도메인 기반: "장바구니 관리", "회원 가입", "상품 검색" 같은 도메인 기능별로 구분
페이지 또는 섹션 단위: 하나의 페이지나 UI의 주요 섹션을 기준으로 범위 설정

### 1.findBy 중복 beforeEach로 변경

```javascript
변경 전

it('로딩이 완료된 경우 상품 리스트가 제대로 모두 노출된다', async () => {
  await render(<ProductList limit={PRODUCT_PAGE_LIMIT} />);

  // 각 테스트마다 findAllByTestId를 별도로 호출
  const productCards = await screen.findAllByTestId('product-card');

  expect(productCards).toHaveLength(PRODUCT_PAGE_LIMIT);
});

it('보여줄 상품 리스트가 더 있는 경우 show more 버튼이 노출되며, 버튼을 누르면 상품 리스트를 더 가져온다.', async () => {
  const { user } = await render(<ProductList limit={PRODUCT_PAGE_LIMIT} />);

  // 여기서도 findAllByTestId를 호출
  await screen.findAllByTestId('product-card');

  expect(screen.getByText('Show more')).toBeInTheDocument();
});

```

```javascript
변경 후
beforeEach(async () => {
  await screen.findAllByTestId('product-card'); // 한 번만 실행되도록 변경
});

it('로딩이 완료된 경우 상품 리스트가 제대로 모두 노출된다', async () => {
  await render(<ProductList limit={PRODUCT_PAGE_LIMIT} />);

  const productCards = screen.getAllByTestId('product-card'); // 중복 호출 제거
  expect(productCards).toHaveLength(PRODUCT_PAGE_LIMIT);
});

it('보여줄 상품 리스트가 더 있는 경우 show more 버튼이 노출되며, 버튼을 누르면 상품 리스트를 더 가져온다.', async () => {
  const { user } = await render(<ProductList limit={PRODUCT_PAGE_LIMIT} />);

  expect(screen.getByText('Show more')).toBeInTheDocument();
});

findAllByTestId -> 비동기 함수로, 요소가 렌더링될 때까지 기다린 후 테스트를 진행
getAllByTestId는 -> 동기적으로 실행, 요소가 존재하지 않으면 즉시 오류를 던짐


```

### 2. alert 모킹 방식 개선

```javascript
변경 전
it('특정 아이템의 수량이 1000개로 변경될 경우 "최대 999개 까지 가능합니다!"라고 경고 문구가 노출된다', async () => {
  const alertSpy = vi.fn();

  // window.alert를 alertSpy로 대체
  // 단점: window.alert가 테스트 후에도 돌아오지 않는다 => vi.resetAllMocks()같은 코드로 일일히 복원해야 한다.
  // 다른 테스트에서 alert쓴다면 영향을 미칠 수 있다.
  vi.stubGlobal('alert', alertSpy);

  const { user } = await render(<ProductInfoTable />);
  const [firstItem] = screen.getAllByRole('row');

  const input = within(firstItem).getByRole('textbox');

  await user.clear(input);
  await user.type(input, '1000');

  expect(alertSpy).toHaveBeenNthCalledWith(1, '최대 999개 까지 가능합니다!');
});

```

```javascript
변경 후
it('특정 아이템의 수량이 1000개로 변경될 경우 "최대 999개 까지 가능합니다!"라고 경고 문구가 노출된다', async () => {
  const alertSpy = vi.spyOn(window, 'alert').mockImplementation(() => {});
  // vi.spyOn() -> 테스트 끝나면 자동으로 window.alert는 mock함수 대신 원래 함수로 돌아옴
  // vi.resetAllMocks()따로 호출할 필요 XX
  // spyOn을 사용하면 함수 호출을 추적하면서도 원래 window.alert을 유지가능

  const { user } = await render(<ProductInfoTable />);
  const [firstItem] = screen.getAllByRole('row');

  const input = within(firstItem).getByRole('textbox');

  await user.clear(input);
  await user.type(input, '1000');

  expect(alertSpy).toHaveBeenNthCalledWith(1, '최대 999개 까지 가능합니다!');
});

```

### 3.계산 방식 변경

```javascript
// 기존 코드
it('특정 아이템의 수량이 변경되었을 때 값이 재계산되어 올바르게 업데이트 된다', async () => {
  const { user } = await render(<ProductInfoTable />);
  const [firstItem] = screen.getAllByRole('row');

  const input = within(firstItem).getByRole('textbox');

  await user.clear(input);
  await user.type(input, '5');

  // 2427 + 809 * 2 = 4045
  expect(screen.getByText('$4,045.00')).toBeInTheDocument();
});

==============

it('특정 아이템의 수량이 변경되었을 때 값이 재계산되어 올바르게 업데이트 된다', async () => {
  const { user } = await render(<ProductInfoTable />);
  const [firstItem] = screen.getAllByRole('row');

  const input = within(firstItem).getByRole('textbox');
  const originalPrice = 809; // 상수로 분리
  const newQuantity = 5;

  await user.clear(input);
  await user.type(input, String(newQuantity));

  // 계산은 여기서
  const expectedTotal = originalPrice * newQuantity;
  expect(screen.getByText(`$${expectedTotal.toLocaleString()}.00`)).toBeInTheDocument();
});

```
