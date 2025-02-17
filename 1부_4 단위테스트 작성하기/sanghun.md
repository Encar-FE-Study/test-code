## 3.1 단위테스트 대상 선정하기

- 간단한 로직 혹은 ui만 존재하는 컴포넌트는 테스트대상 X
  - 테스트 커버리지를 위한 테스트는 의미없음
  - **ui만 그리는 컴포넌트**는 스토리북을 통해 검증
- 로직은 있어도 중요하지 않은 컴포넌트는 통합테스트를 통해서 테스트
  - ex) navigation bar에서 로그인, 장바구니 버튼 분기 테스트 등
  - 다른 컴포넌트들과의 상호작용이 필요함 (로그인 페이지에서 로그인 후 네비게이션 바 변경확인)
- 작은 로직이 있는 유틸함수들도 단위테스트의 대상
  **다른 컴포넌트와 의존성이 없는 독립적 컴포넌트를 대상으로 테스트를 작성**

## 3.2 모듈 모킹

- useNavigate()에 대한 추가 검증은 불필요
- 단위테스트 내에서의 외부 모듈검증은 완전히 분리하고 모듈의 특정기능의 **호출여부**만 검증함

  - 단, 외부 모듈 역시 별도로 검증이 되어야 함

- vi.mock() 사용
- 각 테스트의 독립성과 안정성을 보장하기 위해 teardown으로 모킹을 초기화해야함

  - resetAllMocks, clearAllMocks, restoreAllMocks

- api를 목킹하는 예제

```code
import * as api from '../services/api';

vi.mock('../services/api', () => {
  fetchUser: vi.fn(),
})

beforeEach(() => {vi.resetAllMocks()});

it('should return a user profile with uppercase name', async () => {
  vi.spyOn(api, 'fetchUser').mockResolvedValue({ id: '123', name: 'sanghun'});

  const user = await getUserProfile('123');
  expect(user).toEqual({id: '123', name: 'SANGHUN'});

  expect(api.fetchUser).toHaveBeenCalledTimes(1);
  expect(api.fetchUser).toHaveBeenCalledWith('123');
})
```

## 3.3 리액트 훅 테스트
- 리액트 훅은 단위테스트를 작성하기 적합함
- renderHook api를 사용하면 hook의 최신상태가 원하는대로 변경되는지 result 프로퍼티를 사용하여 검증이 가능
```javascript
import { renderHook, act } from '@testing-library/react';

it('호출 시 initialValue 인자를 지정하지 않는 경우 isModalOpened 상태가 false로 설정된다.', () => {
  const { result } = renderHook(useConfirmModal);

  expect(result.current.isModalOpened).toBe(false);
});
```
### act 함수의 역할
- 컴포넌트를 렌더링한 뒤 업데이트 하는 코드의 결과를 검증하고 싶을 때 사용
- 테스트 환경에서 컴포넌트의 렌더링, 업데이트 결과를 jsdom에 반영할 때 사용해야 함
```javascript
import { renderHook, act } from '@testing-library/react';
it('훅의 toggleIsModalOpened()를 호출하면 isModalOpened 상태가 toggle된다.', () => {
  const { result, rerender } = renderHook(useConfirmModal);
  act(() => {
    result.current.toggleIsModalOpened();
  });
  // rerender(); => rerender도 가능
  expect(result.current.isModalOpened).toBe(true);
});
```
- act대신 rerender를 사용할 시 react의 비동기 업데이트 흐름에 따라 즉시 반영되지 않을 수도 있기에 위험성이 존재함.
- rerender함수는 훅에 새로운 props를 전달할 때 사용

## 타이머 테스트
- 테스트 코드는 비동기 타이머와 무관하게 동기적으로 실행됨
  - 비동기 함수가 실행되기 전 단언이 실행
- vitest에서 지원하는 api 사용
  - vi.useFakeTimers(), vi.advanceTimersByTime(time)
  - teardown을 통해 timer초기화 (vi.useRealTimers())
  - setSystemTime()을 통해 구동되는 현재 시간 정의 가능

## userEvent를 사용한 사용자 상호작용 테스트
- fireEvent는 실제 상황과 거리가 있음
- userEvent 사용을 권장
- 스크롤 이벤트 등을 사용한다면 fireEvent 사용

## 단위테스트의 한계
- 여러 모듈이 조합되었을 때 발생하는 이슈는 찾을 수 없음
  - 앱의 전반적인 기능이 비즈니스 요구사항에 맞게 동작하는지 보장하지 않음
- 따라서 통합테스트가 필요함