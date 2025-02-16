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

# 4. 타이머 테스트

```javascript
// 테스트 코드는 비동기 타이머와 무관하게 동기적으로 실행
// -> 비동기 함수가 실행되기 전에 단언이 실행됨
// 타이머 모킹!
describe("debounce", () => {
  // 타이머 모킹 -> 0.3초 흐른것으로 타이머 조작 -> spy 함수 호출 확인
  beforeEach(() => {
    // teardown에서 모킹 초기화 -> 다른 테스트에 영향이 없어야함

    // 타이머 모킹도 초기화 필수!
    // 3rd 파티 라이브러리, 전역의 teardown에서 타이머에 의존하는 로직 -> fakeTimer로 인해 제대로 동작하지 않을 수 있음
    vi.useFakeTimers();

    // 시간은 흐리기 때문에 매일 달라짐
    // -> 테스트 당시의 시간에 의존하는 테스트의 경우 시간을 고정하지 않으면 테스트가 깨질 수 있다.
    // -> setSystemTime으로 시간을 고정하면 일관된 환경에서 테스트 가능
    vi.setSystemTime(new Date("2025-02-17"));
  });

  afterEach(() => {
    vi.useRealTimers();
  });

  it("특정 시간이 지난 후 함수가 호출된다.", () => {
    const spy = vi.fn();

    const debounceFn = debounce(spy, 300);

    debounceFn();

    vi.advanceTimersByTime(300);

    expect(spy).toHaveBeenCalled();
  });

  it("연이어 호출해도 마지막 호출 기준으로 지정된 타이머 시간이 지난 경우에만 함수가 호출된다.", () => {
    const spy = vi.fn();

    const debounceFn = debounce(spy, 300);

    // 최초 호출
    debounceFn();

    // 최초 호출 후 0.2초 후 호출
    vi.advanceTimersByTime(200);
    debounceFn();

    // 두번째 호출 후 0.1초 후 호출
    vi.advanceTimersByTime(100);
    debounceFn();

    // 세번째 호출 후 0.2초 후 호출
    vi.advanceTimersByTime(200);
    debounceFn();

    // 네번째 호출 후 0.3초 후 호출
    vi.advanceTimersByTime(300);
    debounceFn();

    expect(spy).toHaveBeenCalledTimes(1);
  });
});
```

- 타이머를 원하는대로 제어하기 위해서는 타이머 모킹이 필요하다.
- useFakeTimers()를 통해 타이머를 모킹할 수 있으며, advanceTimersByTime()을 통해 시간이 흐른것 처럼 제어할 수 있다.
- setSystemTime()을 통해 테스트가 구동되는 현재 시간을 정의할 수 있다.

# 5. userEvent를 사용한 사용자 상호작용 테스트

## fireEvent

- @testing-library/react 모듈에 내장되어 제공
- 특정 요소에서 원하는 이벤트만 쉽게 발생시킬 수 있음

## fireEvent vs userEvent

- fireEvent는 DOM이벤트만 발생시키는 반면, userEvent는 다양한 상호 작용을 시뮬레이션 할 수 있음
  - 클릭 이벤트가 발생한다면.. pointerdown, mousedown, pointerup, mouseup, click,
    focus가 연쇄적으로 발생
  - 실제 상황처럼 disabled된 버튼이나 인풋 입력이 불가능함
- 테스트 코드 작성 시에는 userEvent를 활용해 실제 상황과 유사한 코드로 테스트의 신뢰성을 높이자
- userEvent에서 지원하지 않는 부분이 있을 때, fireEvent활용을 고민하자.

```javascript
fireEvent.scroll(container, {
  target: {
    scrollTop: 10000,
  },
});
```

# 5. 단위테스트의 한계

- 단위테스트는 컴포넌트, 커스텀 훅, 공통 유틸처럼 다른 모듈에 대한
  의존성이 거의 없을 때, 모듈 자체만으로 작지만 독립적인 역할을 할 때 효율적으로 검증할 수 있다.
- 단위 테스트에서 검증하지 못하는 부분을 통합·E2E·시각적 테스트 등 다양한 테스트로 보강해야 한다.
- 통합 테스트에서는 여러 모듈이 조합되었을 때 비즈니스 로직을 검증할 수 있고, 비즈니스 로직 기준으로 여러 컴포넌트들 간의 상호 작용을 한번에 검증할 수 있다.
