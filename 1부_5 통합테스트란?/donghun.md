# 섹션 5 통합 테스트란?

> # 통합 테스트란 무엇일까?

- 두 개 이상의 모듈이 상호 작용하여 발생하는 상태를 검증
- 실제 서비스의 로직과 비슷하게 검증이 가능하다
- 단위 테스트에 비해 모킹의 비율이 적으며 모듈 간 발생하는 에러를 검증할 수 있다.
- 하위 모듈 단위의 단위 테스트로 검증할 수 있는 부분까지 한 번에 효율적으로 검증할 수 있다.(단위테스트가 통합테스트의 부분집합)
- 통합 테스트 항목
  - 특정 상태를 기준으로 동작하는 컴포넌트 조합
  - API와 상호 작용하는 컴포넌트 조합
  - 단순 UI 렌더링 및 간단한 로직을 실행하는 컴포넌트까지 한 번에 효율적으로 검증이 가능하다.
- 상태나 데이터를 관리하는 특정 컴포넌트를 기준으로 하위 컴포넌트가 제대로 렌더링 되는지 검증하는 테스트
  - 이 때, 데이터를 관리하는 로직이 산재되어 있다면 통합 테스트 작성이 어렵다.
  - 따라서 앱의 상태를 어디서 어떻게 관리하고 변경할 지 구조적인 설계가 중요하다.
  - ? 엔카의 경우 컨테이너-프리젠테이션 구조이기 때문에 컨테이너를 기준으로 하위 프레젠테이션이 제대로 렌더링 되는지 검증해야 되려나?
- 즉 좋은 설계가 기반이 되어야한다.

> # 통합 테스트 대상 선정하기

- 통합테스트가 검증하는 것들
  - API, 상태, 컨텍스트, 등 다양한 요소들이 결합된 컴포넌트가 특정 비지니스 로직을 올바르게 수행하는 지
  - 주로 컴포넌트간 상호작용, API 호출 및 상태 변경에 따른 UI 변경 사항 검증
- 여러 시나리오를 검증하기 위한 모킹이 필요하다.
  - 모킹은 테스트를 독립적으로 분리하여 효과적으로 검증할 수 있게 도와주지만, 과한 모킹의 사용은 신뢰도를 떨어트린다.
- 통합 테스트도 쪼개어 관리하는 것이 좋다
- 비지니스 로직을 기준으로 통합테스트를 작성하면 얻는 이점
  - 비지니스 로직을 독립적인 기능 관점에서 검증이 가능하다
  - 테스트 자체를 명세서로 볼 수 있기에 서비스를 이해하는 것에 도움이 된다.
  - 불필요한 단위 테스트를 줄여 유지 보수 측면에서 이점을 가진다.
- 통합 테스트는 구성된 비지니스 로직을 적절한 단위로 나누어야 한다.
- 비지니스 로직을 기준으로 통합 테스트를 실행할 때 유의할 점
  - 가능한 한 모킹하지 않고 실제와 유사하게 검증한다
  - 비지니스 로직을 처리하는 상태관리나 API 호출은 상위 컨테이너로 응집한다
  - 변경 가능성을 고려하여 여러 도메인의 기능이 결합된 비지니스 로직은 나눠 검증한다.

> # 상태 관리 모킹하기

- \_mocks\_ 하위에 위치한 파일들은 jest나 vitest에서 특정 모듈을 자동으로 모킹한다. ( 잘 이해 안감.. 내부적으로 해당 디렉토리를 바라보게 만들어져있는건감? )
  - 예제에선 setupTest.js 파일 내에 `vi.mock('zustand')` 을 호출하여 zustand 모듈로 모킹하였다.
- mockZustandStore에 작성되어있는 setState 관련 함수를 실행할 때 act를 안해도 되는건가 .. ?

- 앱의 전역 상태를 모킹해 테스트 전/후에 값을 변경하고 초기화해야한다.

  - mocks/zustand.js 를 통해 자동모킹을 적용해 스토어를 초기화
  - mockZustandStore의 유틸 함수를 통한 zustand의 스토어 상태 변경
  - 다른 상태관리 라이브러리도 모킹 가이드를 제공한다.

- 해당 파트는 zustand에 대해 잘 몰라서 이해가 안가는건가 ?\_?

> # 통합 테스트 작성하기 - 상태 관리 모킹

- ProductInfoTable 통합테스트
  - cart, user store + ProductInfoTableRow가 결합된 컴포넌트
  - state와 API에 대한 제어코드를 응집하는 것이 로직 파악, 유지 보수에 좋다.
  - 비즈니스 로직을 실제 사용성과 유사하도록 사용자 인터랙션을 통해 컴포넌트 통합 테스트로 검증할 수 있다.

```js
it("장바구니에 포함된 아이템들의 이름, 수량, 합계가 제대로 노출된다", async () => {
  await render(<ProductInfoTable />);

  const [firstItem, secondItem] = screen.getAllByRole("row");
  const firstItemWithin = within(firstItem);
  const secondItemWithin = within(secondItem);

  expect(firstItemWithin.getByText("Handmade Cotton Fish")).toBeInTheDocument();
  expect(firstItemWithin.getByRole("textbox")).toHaveValue("3");
  expect(firstItemWithin.getByText("$2,427.00")).toBeInTheDocument();

  expect(
    secondItemWithin.getByText("Awesome Concrete Shirt")
  ).toBeInTheDocument();
  expect(secondItemWithin.getByRole("textbox")).toHaveValue("4");
  expect(secondItemWithin.getByText("$1,768.00")).toBeInTheDocument();
});

it("특정 아이템의 수량이 변경되었을 때 값이 재계산되어 올바르게 업데이트 된다", async () => {
  const { user } = await render(<ProductInfoTable />);
  const [firstItem] = screen.getAllByRole("row");
  const input = within(firstItem).getByRole("textbox");

  await user.clear(input);
  await user.type(input, "5");

  expect(screen.getByText("$4,045.00")).toBeInTheDocument();
});

it('특정 아이템의 수량이 1000개로 변경될 경우 "최대 999개 까지 가능합니다!"라고 경고 문구가 노출된다', async () => {
  const { user } = await render(<ProductInfoTable />);
  const [firstItem] = screen.getAllByRole("row");
  const input = within(firstItem).getByRole("textbox");
  const alertSpy = vi.fn();
  vi.stubGlobal("alert", alertSpy);

  await user.clear(input);
  await user.type(input, "1000");

  expect(alertSpy).toHaveBeenNthCalledWith(1, "최대 999개 까지 가능합니다!");
});

it("특정 아이템의 삭제 버튼을 클릭할 경우 해당 아이템이 사라진다", async () => {
  const { user } = await render(<ProductInfoTable />);
  const [, secondItem] = screen.getAllByRole("row");
  const deleteButton = within(secondItem).getByRole("button");

  expect(screen.getByText("Awesome Concrete Shirt")).toBeInTheDocument();

  await user.click(deleteButton);
  // queryBy~~ 은 getBy~~ 과 다르게 요소의 존재 유무를 판단한다. getBy는 요소가 없다면 에러가 난다.
  expect(screen.queryByText("Awesome Concrete Shirt")).not.toBeInTheDocument();
});
```

> # MSW로 API 모킹하기

> # RTL 비동기 유틸 함수를 통한 노출 테스트 작성
