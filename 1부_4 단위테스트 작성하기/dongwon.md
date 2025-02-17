# 단위테스트 작성하기

## 3.1 단위 테스트 대상 선정하기

단위 테스트란? 앱에서 테스트 가능한 가장 작은 소프트웨어를 실행해 예상대로 동작하는지 확인하는 테스트

=> 결괏값 또는 상태, 행위를 검증

empty, error, notfound만 단위테스트 할거임. 나머지는 통합테스트

util, hook은 단위테스트 적합

- ui만 그리는 컴포넌트는 검증하지 않는다.
- 간단한 로직 처리만 하는 컴포넌트는 상위 컴포넌트의 통합 테스트에서 검증
- 공통 유틸 함수는 단위 테스트로 검증
  - 다른 모듈과의 의존성이 없다
  - 여러 곳에서 사용되기 때문에 검증을 통해 안정성을 높인다

## 3.2 모듈모킹

모킹이란 실제 모듈 객체와 동일한 동작을 하도록 만든 모의 모듈 객체로 실제를 대체하는 것

장단점

- 외부 모듈과 의존성을 제외한 필요한 부분만 검증이 가능
- 실제 모듈과 완전히 동일한 모의 객체를 구현하는 것은 큰 비용
- 모의 객체를 남용하는 것은 테스트 신뢰성을 낮춤

- toHaveBeenNthCalledWith

- vi.mock을 사용해 특정 모듈을 모킹할 수 있다.
- 각 테스트의 독립성과 안정성을 보장하기 위해 teardown에서 모킹을 초기화하자
- viest의 resetAllMocks, clearAllMocks, resotreAllMocks를 활용해 초기화하자

## 3.3 hook 테스트(feat.act함수)

비즈니스 로직을 분리하여 컴포넌트와 비즈니스의 결합도를 낮춤

- result: 훅을 호출하여 얻은 결과 값을 반환 => result.current 값의 참조를 통해 최신 상태를 추적한다.
- renderHook: 훅을 원하는 인자와 함께 새로 호출하여 상태를 갱신한다.
- rerender

act 함수:

- 상호 작용을 함께 그룹화하고 실행해 렌더링과 업데이트가 실제 앱이 동작하는 것과 유사한 방식으로 동작함
- 컴포넌트를 렌더링한 뒤 업데이트하는 코드의 결과를 검증하고 싶을 때 사용

=> 테스트 환경에서 컴포넌트 렌더링 결과를 jsdom에 반영하기 위해 act 함수를 반드시 호출해야 함.

```js
it('호출 시 initialValue 인자를 지정하지 않는 경우 isModalOpened 상태가 false로 설정된다.', () => {
  const { result } = renderHook(useConfirmModal);
  expect(result.current.isModalOpened).toBe(false);
});

it('호출 시 initialValue 인자를 boolean 값으로 지정하는 경우 해당 값으로 isModalOpened 상태가 설정된다.', () => {
  const { result } = renderHook(() => useConfirmModal(true));
  expect(result.current.isModalOpened).toBe(true);
});

it('훅의 toggleIsModalOpened()를 호출하면 isModalOpened 상태가 toggle된다.', () => {
  const { result } = renderHook(useConfirmModal);
  act(() => {
    result.current.toggleIsModalOpened();
  });
  expect(result.current.isModalOpened).toBe(true);
});
```

## 3.4 타이머 테스트

타이머기반 비동기 테스트

타이머 모킹 필요

- useFakeTimers
- advanceTimersByTime
- setSystemTime
- useRealTimers

- debounce

```js
it('특정시간이 지난 후 함수가 호출된다', () => {
  beforeEach(() => {
    vi.useFakeTimers();
  });
  vi.useFakeTimers();
  const spy = vi.fn();
  const debounceFn = debounce(spy, 300);
  debounceFn();
  vi.advanceTimersByTime(300);
  expect(spy).toHaveBeenCalled();
});
```

## 3.5 userEvent를 사용한 사용자 상호 작용 테스트

fire 이벤트 모듈이 존재함!!

fireEvent vs userEvent

fireEvent는 별도의 설치없이 가능, 단순하게 해당 이벤트만 디스패치. 실제 사용자 처럼 다양하게 활용이 안됨

fireEvent는 실제 상황과 거리가 있다!!

userEvent는 disabled된 텍스트 필드는 입력이 불가 => 테스트의 신뢰성 향상

## 3.6 단위 테스트의 한계

효율적으로 모듈의 핵심 기능을 검증할 수 있다!!

=> 여러 모듈이 조합되었을 때 비즈니스 요구사항에 맞게 동작하는지 보장할 수 없다.
=> 이를 위해 통합, e2e, 시작적 테스트로 보강해야 한다

앞으로 통합 테스트는 비즈니스 로직을 검증함
