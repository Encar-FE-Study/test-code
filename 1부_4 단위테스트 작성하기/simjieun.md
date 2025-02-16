# 1. 단위테스트 대상 선정하기

### 쇼핑몰 예제 프로젝트의 단위 테스트 전략

- state나 로직처리 없이 UI만 그리는 컴포넌트는 검증하지 않는다.
- 간단한 로직 처리만 하는 컴포넌트는 상위 컴포넌트의 통합 테스트에서 검증한다.
- 공통 유틸 함수, 훅은 단위 테스트로 검증한다.
- 상위에서 다른 컴포넌트들과 조합되지 않고 독립적으로 동작하는 컴포넌트는 단위테스트 하기 좋다

# 2. 모듈 모킹(Mocking)

- git checkout shopping-mall-unit-test

## 모킹

- 모킹이란? 실제 모듈.객체와 동일한 동작을 하도록 만든 모의 모듈.객체로 실제를 대체하는 것이다.
- useNavigate()를 검증 해야할까? react, react-router-dom 같은 범용적인 라이브러리는 이미 단위테스트로 검증된 상태이다.

```javascript
import { screen } from "@testing-library/react";
import React from "react";

import ErrorPage from "@/pages/error/components/ErrorPage";
import render from "@/utils/test/render";

const navigateFn = vi.fn(); // useNavigate()의 스파이 함수

// react-router-dom의 모킹
vi.mock("react-router-dom", async () => {
  const original = await vi.importActual("react-router-dom");
  return { ...original, useNavigate: () => navigateFn };
});

it('"뒤로 이동" 버튼 클릭시 뒤로 이동하는 navigate(-1) 함수가 호출된다', async () => {
  const { user } = await render(<ErrorPage />);

  const button = await screen.getByRole("button", { name: "뒤로 이동" });
  await user.click(button);

  expect(navigateFn).toHaveBeenNthCalledWith(1, -1);
});
```

## 모킹 초기화

- 모든 테스트가 끝날 때마다 mocking 초기화를 하기 위해 teardown 작업을 전역에서 동작하도록 setup파일에 작성함

```javascript
...
afterEach(() => {
  server.resetHandlers();
  // 모킹된 모의 객체 호출에 대한 히스토리 초기화
  // 모킹된 모듈의 구현을 초기화 하지 않는다. -> 모킹된 상태로 유지됨
  // -> 모킹 모듈 기반으로 작성한 테스트가 올바르게 실행
  // 반면, 모킹 히스토리가 계속 쌓임(호출 횟수나 인자가 계속 생김) -> 다른 테스트에 영향을 줄수 잇음
  vi.clearAllMocks();
});

afterAll(() => {
  // 모킹 모듈에 대한 모든 구현을 초기화
  vi.resetAllMocks();
  server.close();
});
...
```

- 각 테스트의 독립성과 안정성을 보장하기 위해 teardown에서 모킹을 초기화 하자

# 3. 리액트 훅 테스트

- React Hook은 독립적인 단위 테스트를 작성하기 좋은 케이스이다.
- 외부 모듈에 대한 의존성이 없고 독립적으로 동작하는 함수 일수록 단위 테스트를 작성하기 좋다.
- React Testing Library에 renderHook API를 사용하면 React 컴포넌트 없이도 React Hook의 기능을 편리하게 검증할수 있다.

```javascript
import { renderHook, act } from "@testing-library/react";

import useConfirmModal from "./useConfirmModal";
// 리액트 훅은 반드시 리액트 컴포넌트내에서만 호출되어야 정상적으로 실행된다.
it("호출 시 initialValue 인자를 지정하지 않는 경우 isModalOpened 상태가 false로 설정된다.", () => {
  // result : 훅을 호출하여 얻은 결과 값을 반환 -> result.current 값의 참조를 통해 최신 상태를 추적할 수 있다.
  // rerender : 훅을 원하는 인자와 함께 새로 호출하여 상태를 갱신한다.
  const { result, rerender } = renderHook(useConfirmModal);

  expect(result.current.isModalOpened).toBe(false);
});

it("호출 시 initialValue 인자를 boolean 값으로 지정하는 경우 해당 값으로 isModalOpened 상태가 설정된다.", () => {
  const { result } = renderHook(() => useConfirmModal(true));

  expect(result.current.isModalOpened).toBe(true);
});

it("훅의 toggleIsModalOpened()를 호출하면 isModalOpened 상태가 toggle된다.", () => {
  const { result } = renderHook(useConfirmModal);

  act(() => {
    result.current.toggleIsModalOpened();
  });

  expect(result.current.isModalOpened).toBe(true);
});
```

- Act 함수는 렌더링, 이펙트와 같은 상호작용을 함께 그룹화하고 실행하여 실제 앱에서 동작하는 것처럼 렌더링과 업데이트 상태를 반영하도록 도와준다.
- 테스트 환경에서는 컴포넌트의 렌더링, 업데이트 결과를 JS Dom에 반영할때 반드시 사용해야 한다.
- React Testing Library의 render함수와 user-event는 내부적으로 act 함수를 호출하기 때
  문에 편리하게 테스트 코드 작성이 가능했다.
