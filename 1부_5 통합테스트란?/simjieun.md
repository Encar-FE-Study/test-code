# 1. 통합 테스트란 무엇일까?

## 단위 테스트의 한계

- 독립된 일부 모듈만을 대상으로 검증하기때문에 여러 모듈이 조합되었을 때의 동작은 검증할 수 없다.

## 통합 테스트는

- 두 개 이상의 모듈이 상호 작용하여 발생하는 상태를 검증 한다.
- 실제 앱의 비지니스 로직과 가깝게 기능을 검증할 수 있다.

## 통합 테스트 항목

- 특정 상태를 기준으로 동작하는 컴포넌트 조합
- API와 함께 상호작용하는 컴포넌트 조합
- **단순 UI 렌더링 및 간단한 로직을 실행하는 컴포넌트까지 한번에 효율적으로 검증이 가능하다**

## 우리가 작성할 통합 테스트는

- 통합테스트를 효율적으로 작성하고 운영하기 위해서는 컴포넌트의 조합을 어떤 범위로 나눌지가 매우 중요하다.
- 프론트엔드에서 주로 작성할 통합 테스트는 상태나 데이터를 관리하는 특정 컴포넌트를 기준으로 하위 컴포넌트들이 제대로 렌더링 되는지 검증하는 테스트이다.
- 그렇기 때문에 컴포넌트의 상태 및 데이터를 어디서 어떻게 관리하고 변경할지 구조적인 설계를 잘 해야 통합 테스트를 작성할 수있다.
- **즉, 통합 테스트를 효율적으로 작성하고 관리하기 위해서는 좋은 설계가 기반이 되어야한다.**

# 2. 통합 테스트 대상 선정하기

## 통합 테스트는 무엇을 검증할까?

- API,상태 관리 스토어,리액트 컨텍스트 등 다양한 요소들이 결합된 컴포넌트가 특정 비지니스 로직을 올바르게 수행하는지 검증하는 것이다.
- 이 과정에서 컴포넌트 간의 상호작용이나 API 호출 및 상태 변경에 따른 UI 변경 사항을 검증하게 된다.
- 그렇기 때문에 거대한 책임으로 구성된 비즈니스 로직을 나누어 컴포넌트 집합을 구성하고 이 컴포넌트 집합을 하나의 통합 테스트 단위로 선정하여 테스트 코드를 작성하게 된다.
- 다양한 기능 검증을 하기 위해 API, 상태 관리 스토어 모킹이 필요하다.

## 모킹(Mocking)

- 모킹은 테스트를 독립적으로 분리하여 효과적으로 검증할 수 있게 도와주지만 지나치게 많은 모킹은 테스트 신뢰성을 저하시키며 변경에 취약하다.
- 지나친 모킹으로 테스트 신뢰성을 저하시킬 수 있다.

## 비즈니스 로직 이란?

- 도메인 로직이라고도 불리는 비즈니스 로직은 프로그램의 핵심 기능을 구현하는 코드를 의미한다.
- 즉, 사용자가 원하는 결과를 얻기 위한 계산,처리,의사 결정을 수행하는 코드가 비즈니스 로직이다.
- 비즈니스 로직을 통해 서비스의 정책, 절차, 규칙 등을 코드로 구현하고 데이터를 조회하거나 수정, 삭제한다.

## 비즈니스 로직을 기준으로 통합 테스트를 작성한다면

- 서비스의 핵심 비즈니스 로직을 독립적인 기능 관점에서 효율적으로 검증할 수 있다.
- 도메인 단위로 잘 나누어진 통합 테스트 코드는 앱의 명세 자체를 이해할 수 있어 앱 전반을 이해하는데 큰 도움이 된다.
- 불필요한 단위 테스트 없이 효율적으로 기능을 검증할 수 있어 유지보수 측면에서도 좋다.

## 비즈니스 로직을 기준으로 통합 테스트를 나눌때는

- 가능한 모킹을 하지 않고 최대한 앱의 실제 기능과 유사하게 검증한다.
- 비즈니스 로직을 처리하는 상태 관리나 API로직은 상위 컴포넌트로 응집해 관리한다.
- 변경 가능성을 고려해 여러 도메인 기능이 조합된 비즈니스 로직은 나누어 통합 테스트를 작성한다.

# 3. 상태 관리 모킹

- zustand를 사용하여 상태관리를 테스트 한다.
- 원하는 상태로 통합 테스트를 하기 위해 zustand 모킹 필요하다.
- __mocks__/zustand.js를 통해 자동 모킹을 적용해 스토어를 초기화한다.

# 4. 통합 테스트 작성하기 - 상태 관리 모킹
```javascript
import { screen, within } from '@testing-library/react';
import React from 'react';

import ProductInfoTable from '@/pages/cart/components/ProductInfoTable';
import {
  mockUseCartStore,
  mockUseUserStore,
} from '@/utils/test/mockZustandStore';
import render from '@/utils/test/render';

beforeEach(() => {
  mockUseUserStore({ user: { id: 10 } });
  mockUseCartStore({
    cart: {
      6: {
        id: 6,
        title: 'Handmade Cotton Fish',
        price: 809,
        description:
          'The slim & simple Maple Gaming Keyboard from Dev Byte comes with a sleek body and 7- Color RGB LED Back-lighting for smart functionality',
        images: [
          'https://user-images.githubusercontent.com/35371660/230712070-afa23da8-1bda-4cc4-9a59-50a263ee629f.png',
          'https://user-images.githubusercontent.com/35371660/230711992-01a1a621-cb3d-44a7-b499-20e8d0e1a4bc.png',
          'https://user-images.githubusercontent.com/35371660/230712056-2c468ef4-45c9-4bad-b379-a9a19d9b79a9.png',
        ],
        count: 3,
      },
      7: {
        id: 7,
        title: 'Awesome Concrete Shirt',
        price: 442,
        description:
          'The Nagasaki Lander is the trademarked name of several series of Nagasaki sport bikes, that started with the 1984 ABC800J',
        images: [
          'https://user-images.githubusercontent.com/35371660/230762100-b119d836-3c5b-4980-9846-b7d32ea4a08f.png',
          'https://user-images.githubusercontent.com/35371660/230762118-46d965ab-7ea8-4e8a-9c0f-3ed90f96e1cd.png',
          'https://user-images.githubusercontent.com/35371660/230762139-002578da-092d-4f34-8cae-2cf3b0dfabe9.png',
        ],
        count: 4,
      },
    },
  });
});

it('장바구니에 포함된 아이템들의 이름, 수량, 합계가 제대로 노출된다', async () => {
  await render(<ProductInfoTable />);

  const [firstItem, secondItem] = screen.getAllByRole('row');

  expect(
    within(firstItem).getByText('Handmade Cotton Fish'),
  ).toBeInTheDocument();
  expect(within(firstItem).getByRole('textbox')).toHaveValue('3');
  expect(within(firstItem).getByText('$2,427.00')).toBeInTheDocument();

  expect(
    within(secondItem).getByText('Awesome Concrete Shirt'),
  ).toBeInTheDocument();
  expect(within(secondItem).getByRole('textbox')).toHaveValue('4');
  expect(within(secondItem).getByText('$1,768.00')).toBeInTheDocument();
});

it('특정 아이템의 수량이 변경되었을 때 값이 재계산되어 올바르게 업데이트 된다', async () => {
  const { user } = await render(<ProductInfoTable />);
  const [firstItem] = screen.getAllByRole('row');

  const input = within(firstItem).getByRole('textbox');

  await user.clear(input);
  await user.type(input, '5');

  // 2427 + 809 * 2 = 4045
  expect(screen.getByText('$4,045.00')).toBeInTheDocument();
});

it('특정 아이템의 수량이 1000개로 변경될 경우 "최대 999개 까지 가능합니다!"라고 경고 문구가 노출된다', async () => {
  const alertSpy = vi.fn();

  // window.alert -> alertSpy로 대체
  vi.stubGlobal('alert', alertSpy);

  const { user } = await render(<ProductInfoTable />);
  const [firstItem] = screen.getAllByRole('row');

  const input = within(firstItem).getByRole('textbox');

  await user.clear(input);
  await user.type(input, '1000');

  expect(alertSpy).toHaveBeenNthCalledWith(1, '최대 999개 까지 가능합니다!');
});

it('특정 아이템의 삭제 버튼을 클릭할 경우 해당 아이템이 사라진다', async () => {
  const { user } = await render(<ProductInfoTable />);

  const [, secondItem] = screen.getAllByRole('row');
  const deleteButton = within(secondItem).getByRole('button');

  expect(screen.getByText('Awesome Concrete Shirt')).toBeInTheDocument();

  await user.click(deleteButton);

  expect(screen.queryByText('Awesome Concrete Shirt')).not.toBeInTheDocument();
});

```

# 5. msw로 API 모킹하기
- MSW : Node.js 및 브라우저 환경을 위한 API 모킹 라이브러리이다. 
- 브라우저에서는 서비스 워커를 사용하여 모킹한다.
- Node.js 환경에서는 내부적으로 XHR, fetch 등의 요청을 가로채는 인터셉터를 사용한다.
- setup과 teardown을 사용해 테스트 실행 전, 후에 API를 모킹한다.

# 6. RTL 비동기 유틸 함수를 통한 노출 테스트 작성
- findBy
  - 쿼리가 통과하거나 시간 초과될 때 까지 재시도한다.(1초 동안 50ms간격으로)
  - API 호출과 같은 비동기 처리로 인한 변화를 감지해야 할 때 사용하기 좋다.
  - RTL에서는 findBy같은 비동기 메서드의 반환값은 Promise이기에, 해당 요소를 사용하려면 await이나 then을 사용해야 한다.
```javascript
import { screen, within } from '@testing-library/react';
import React from 'react';

import data from '@/__mocks__/response/products.json';
import ProductList from '@/pages/home/components/ProductList';
import { formatPrice } from '@/utils/formatter';
import {
  mockUseUserStore,
  mockUseCartStore,
} from '@/utils/test/mockZustandStore';
import render from '@/utils/test/render';

const PRODUCT_PAGE_LIMIT = 5;

const navigateFn = vi.fn();

vi.mock('react-router-dom', async () => {
  const original = await vi.importActual('react-router-dom');
  return {
    ...original,
    useNavigate: () => navigateFn,
    useLocation: () => ({
      state: {
        prevPath: 'prevPath',
      },
    }),
  };
});

it('로딩이 완료된 경우 상품 리스트가 제대로 모두 노출된다', async () => {
  await render(<ProductList limit={PRODUCT_PAGE_LIMIT} />);

  // 1초 동안 50ms 마다 요소가 있는지 조회
  const productCards = await screen.findAllByTestId('product-card');

  expect(productCards).toHaveLength(PRODUCT_PAGE_LIMIT);

  productCards.forEach((el, index) => {
    const productCard = within(el);
    const product = data.products[index];

    expect(productCard.getByText(product.title)).toBeInTheDocument();
    expect(productCard.getByText(product.category.name)).toBeInTheDocument();
    expect(
      productCard.getByText(formatPrice(product.price)),
    ).toBeInTheDocument();
    expect(
      productCard.getByRole('button', { name: '장바구니' }),
    ).toBeInTheDocument();
    expect(
      productCard.getByRole('button', { name: '구매' }),
    ).toBeInTheDocument();
  });
});

it('보여줄 상품 리스트가 더 있는 경우 show more 버튼이 노출되며, 버튼을 누르면 상품 리스트를 더 가져온다.', async () => {
  const { user } = await render(<ProductList limit={PRODUCT_PAGE_LIMIT} />);

  // show more 버튼의 노출 여부를 정확하게 판단하기 위해
    // findBy 쿼리를 사용하여 먼저 첫 페이지에 해당하는 상품 목록이 렌더링되는 것을 기다려야 함.
  await screen.findAllByTestId('product-card');

  expect(screen.getByText('Show more')).toBeInTheDocument();

  const moreBtn = screen.getByText('Show more');
  await user.click(moreBtn);

  expect(await screen.findAllByTestId('product-card')).toHaveLength(
    PRODUCT_PAGE_LIMIT * 2,
  );
});

it('보여줄 상품 리스트가 없는 경우 show more 버튼이 노출되지 않는다.', async () => {
  await render(<ProductList limit={20} />);

  await screen.findAllByTestId('product-card');

  expect(screen.queryByText('Show more')).not.toBeInTheDocument();
});

describe('로그인 상태일 경우', () => {
  beforeEach(() => {
    mockUseUserStore({ isLogin: true, user: { id: 10 } });
  });

  it('구매 버튼 클릭시 addCartItem 메서드가 호출되며, "/cart" 경로로 navigate 함수가 호출된다.', async () => {
    const addCartItemFn = vi.fn();
    mockUseCartStore({ addCartItem: addCartItemFn });

    const { user } = await render(<ProductList limit={PRODUCT_PAGE_LIMIT} />);

    await screen.findAllByTestId('product-card');

    // 첫번째 상품을 대상으로 검증한다.
    const productIndex = 0;
    await user.click(
      screen.getAllByRole('button', { name: '구매' })[productIndex],
    );

    expect(addCartItemFn).toHaveBeenNthCalledWith(
      1,
      data.products[productIndex],
      10,
      1,
    );
    expect(navigateFn).toHaveBeenNthCalledWith(1, '/cart');
  });

  it('장바구니 버튼 클릭시 "장바구니 추가 완료!" toast를 노출하며, addCartItem 메서드가 호출된다.', async () => {
    const addCartItemFn = vi.fn();
    mockUseCartStore({ addCartItem: addCartItemFn });

    const { user } = await render(<ProductList limit={PRODUCT_PAGE_LIMIT} />);

    await screen.findAllByTestId('product-card');

    // 첫번째 상품을 대상으로 검증한다.
    const productIndex = 0;
    const product = data.products[productIndex];
    await user.click(
      screen.getAllByRole('button', { name: '장바구니' })[productIndex],
    );

    expect(addCartItemFn).toHaveBeenNthCalledWith(1, product, 10, 1);
    expect(
      screen.getByText(`${product.title} 장바구니 추가 완료!`),
    ).toBeInTheDocument();
  });
});

describe('로그인이 되어 있지 않은 경우', () => {
  it('구매 버튼 클릭시 "/login" 경로로 navigate 함수가 호출된다.', async () => {
    const { user } = await render(<ProductList limit={PRODUCT_PAGE_LIMIT} />);

    await screen.findAllByTestId('product-card');

    // 첫번째 상품을 대상으로 검증한다.
    const productIndex = 0;
    await user.click(
      screen.getAllByRole('button', { name: '구매' })[productIndex],
    );

    expect(navigateFn).toHaveBeenNthCalledWith(1, '/login');
  });

  it('장바구니 버튼 클릭시 "/login" 경로로 navigate 함수가 호출된다.', async () => {
    const { user } = await render(<ProductList limit={PRODUCT_PAGE_LIMIT} />);

    await screen.findAllByTestId('product-card');

    // 첫번째 상품을 대상으로 검증한다.
    const productIndex = 0;
    await user.click(
      screen.getAllByRole('button', { name: '장바구니' })[productIndex],
    );

    expect(navigateFn).toHaveBeenNthCalledWith(1, '/login');
  });
});

it('상품 클릭시 "/product/:productId" 경로로 navigate 함수가 호출된다.', async () => {
  const { user } = await render(<ProductList limit={PRODUCT_PAGE_LIMIT} />);

  const [firstProduct] = await screen.findAllByTestId('product-card');

  // 첫번째 상품을 대상으로 검증한다.
  await user.click(firstProduct);

  expect(navigateFn).toHaveBeenNthCalledWith(1, '/product/6');
});

```
