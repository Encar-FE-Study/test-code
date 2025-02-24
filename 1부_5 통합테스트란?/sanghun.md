## 4.1 통합 테스트란 무엇일까?

- 통합테스트는 두개 이상의 모듈이 상호작용하여 발생하는 상태를 검증
- 실제 앱의 비즈니스 로직과 가깝게 기능을 검증할 수 있음

- **상태나 데이터**를 관리하는 특정 컴포넌트를 기준으로 하위 컴포넌트가 제대로 렌더링 되는지 검증하는 테스트
- 통합 테스트를 잘 작성하기 위해서는 **테스트가 가능한 좋은 설계**가 기반이 되어야함

### 테스트 가능한 좋은 설계란?

1. 비즈니스 로직을 UI에서 분리
   - UI 컴포넌트는 화면을 렌더링하고 사용자 입력을 처리하는 역할 수행
   - 비즈니스 로직은 별도의 훅이나 서비스 함수로 분리 후 따로 테스트

```javascript
// ui와 비즈니스 로직 결합으로 테스트 하기 어려운 코드
// 테스트 시 api mocking도 필요
import { useState, useEffect } from 'react';

function UserProfile({ userId }) {
  const [user, setUser] = useState(null);

  useEffect(() => {
    fetch(`https://api.example.com/users/${userId}`)
      .then((res) => res.json())
      .then((data) => setUser(data));
  }, [userId]);

  if (!user) return <p>Loading...</p>;

  return <div>{user.name}</div>;
}

// api mocking없이 쉽게 테스트 가능
function UserProfile({ userId }) {
  const user = useUser(userId); // API 호출 로직을 분리하여 가져옴

  if (!user) return <p>Loading...</p>;

  return <div>{user.name}</div>;
}

export default UserProfile;
```

2. 상태를 최소화하고 props를 활용한 설계
   - 컴포넌트 내부적으로 useState, useRef 상태를 활용하면 state 의존적인 컴포넌트가 될 수 있어 재사용이 어려워짐
   - 꼭 필요한 상태가 아니라면 props로 전달하도록 설계

```javascript
// 컴포넌트 내부에 state 사용으로 테스트 어려움
function Counter() {
  const [count, setCount] = useState(0);
  return (
    <div>
      <p>Count: {count}</p>
      <button onClick={() => setCount(count + 1)}>Increment</button>
    </div>
  );
}

// props로 전달하여 컴포넌트 테스트가 쉬워짐
function Counter({ count, onIncrement }) {
  return (
    <div>
      <p>Count: {count}</p>
      <button onClick={onIncrement}>Increment</button>
    </div>
  );
}
```

### 통합 테스트의 장점

- 단위 테스트에 비해 모킹의 비중이 적고, 모듈 간에 발생하는 에러를 검증할 수 있음
- 하위 모듈의 단위 테스트에서 검증할 수 있는 부분까지 한번에 효율적으로 검증 가능
- 테스트 가능한 코드를 작성하기 위해 좋은 구조를 설계하도록 생각하게 도와줌

## 4.2 통합테스트 대상 선정하기

- **모킹**은 테스트를 독립적으로 분리하여 효과적으로 검증할 수 있게 도와주지만, 모킹의 과한 사용은 테스트 신뢰성을 저하시키며 변경에 취약함
- 거대한 통합테스트는 모킹 코드가 증가하고 컴포넌트의 수정으로 인해 테스트가 깨짐

### 비즈니스 로직 기준으로 통합테스트 작성

- 서비스의 핵심 비즈니스 로직을 독립적인 기능 관점에서 효율적으로 검증
- 테스트를 명세 자체로 볼 수 있어 앱을 이해하는데 도움
- 불필요한 단위테스트를 줄여 유지보수에 도움
- **비즈니스 로직을 적절히 분리하여 통합테스트도 분리하여 작성**
- 통합테스트는 가능한 한 모킹을 하지 않고 최대한 앱의 실제 기능과 유사하게 검증

## 4.3 상태관리 모킹하기

- 원하는 상태로 통합 테스트를 진행하기 위해서는 zustand, redux 등 사용하는 상태관리 스토어의 모킹이 필요함
- 앱의 전역 상태를 모킹해 테스트 전, 후에 값을 변경하고 초기화해야 함
  - **mocks** 폴더 하위에 모킹할 상태관리 파일을 등록 해서 초기화 시키면 됨

## 4.4 통합테스트 작성하기 - 상태 관리 모킹

> 테스트 파일 생성 시 **spec**을 사용하는 이유는 해당 테스트 파일이 요구사항 명세로 사용할 수 있다는 의미로 사용함. (specification)
> BDD(Behavior-Driven Development)와 연관

- state, api에 대한 제어 코드를 통합 테스트 대상 컴포넌트(최상위 컴포넌트)로 응집

  - 유지보수성 향상, 테스트의 단위 나누기에 용이

- alert도 spy함수를 통해서 캐치 가능

```javascript
const alertSpy = vi.fn();
vi.stubGlobal('alert', alertSpy);
```

- 요소가 dom에 존재하지 않는지에 대해서 단언할 때는 **getByText 대신 queryByText**를 사용한다.

## 4.5 msw로 API 모킹하기

- 테스트에서 api 호출 -> 실행시간 증가, 서버 이슈로 인한 테스트 실패
  - API 응답 모킹 -> 일관된 테스트 환경 구성 가능

### Mock Service Worker (MSW)

- node.js 및 브라우저 환경을 위한 api 모킹 라이브러리
- BE의 api 규격정도만 받아서 fe에서 api 테스트 가능

## 4.6 RTL 비동기 유틸 함수를 활용한 테스트 작성

- findBy 쿼리는 비동기적으로 동작하는 코드로 인한 요소를 쿼리할 때 사용
  - api 호출과 같은 비동기 처리로 인한 변화를 감지할 때 사용
  - 반환값은 Promise이기 때문에 await, then 사용
- waitFor를 사용해도 작성 가능하지만, findBy- 쿼리 역시 내부적으로 waitFor 사용

```javascript
// findBy- 쿼리 사용
const productCards = await screen.findAllByTestId('product-card');
// waitFor 사용
const productCards = await waitFor(() => screen.getAllByTestId('product-card'));
```

- handlers.js
