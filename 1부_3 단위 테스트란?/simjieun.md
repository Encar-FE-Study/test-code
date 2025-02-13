# 1. 단위 테스트란 무엇인가?
- 단일 함수 또는 단일 컴포넌트(클래스)를 예상대로 동작(결괏값 또는 상태(UI), 행위를 검증)하는지 확인하는 테스트 이다.
- 단위 테스트의 대상 : 공통 컴포넌트(텍스트 인풋, 버튼, 캐러셀, 아코디언등등) 영역

## 테스트를 작성하는 패턴
### Arrange, Act, Assert 패턴으로 테스트 코드를 작성
	1. Arrange: 테스트를 위한 환경 만들기
    2. Act: 테스트할 동작 발생
	3. Assert: 올바른 동작이 실행 되었는지 또는 변경사항 검증하기

- Arrange에 해당하는 테스트 환경 만드는건 React Testing Library는 컴포넌트 테스트를 손쉽게 할 수 있도록 도와주는 라이브러리이다.
- render API를 호출 -> 테스트 환경의 jsDOM에 리액트 컴포넌트가 렌더링되는 DOM구조가 반영된다.
- jsDOM: Node.js에서 사용하기 위해 많은 웹 표준을 순수 자바스크립트로 구현한 것이다.
  - 요소를 검색하는 다양한 쿼리 API와 렌더링된 React 엘리먼트의 DOM node를 의미하는 컨테이너, 컴포넌트를 리렌더링 하는 리렌더링 API등 다양한 정보를 반환한다.

# 2. 테스트 환경과 매처
- 테스트 실행 및 검증에 필요한 다양한 환경을 제공하는 도구를 테스트 프레임워크라고 한다.
- 프론트엔드 테스트 프레임워크로는 자스민, Jest, Vitest, cypress, playlight 다양한 프레임워크가 존재한다.
- 해당 강의에서는 Vitest 테스트 프레임워크를 사용한다.
  - 테스트의 별도 설정없이 같은 환경에서 빠르게 테스트를 실행할수 있고 수많은 장점이 있다.
- [Vitest 공식홈페이지](https://vitest.dev/)
- vitest.config.js
```
import path from 'path';

import react from '@vitejs/plugin-react';
import eslint from 'vite-plugin-eslint';
import { defineConfig } from 'vitest/config';

// https://vitejs.dev/config/
export default defineConfig({
  plugins: [react(), eslint({ exclude: ['/virtual:/**', 'node_modules/**'] })],
  test: {
    globals: true, // 별도의 import 없이 vitest에서 제공하는 api를 사용할수있는 옵션
    environment: 'jsdom', // 
    setupFiles: './src/utils/test/setupTests.js',
  },
  resolve: {
    alias: [{ find: '@', replacement: path.resolve(__dirname, 'src') }],
  },
});

```
- Node 환경에서는 DOM이 존재하지 않기 때문에 DOM이 제대로 렌더링되는지 확인하기 위해 jsdom을 사용하는데
- jsdom(https://github.com/jsdom/jsdom)은 whatwg의 DOM과 HTML표준 스펙을 Node.js 환경에서 사용할 수 있도록 순수 Javascript로 구현한 것이다.
- React Testing Library에서 제공하는 **screen.debug()** 라는 함수를 이용하여 DOM구조를 확인할 수 있다.
- it 함수는 test 함수의 alias로서 동일한 역할 수행
- describe 함수는 테스트를 그룹핑해서 새로운 블럭, 즉 컨텍스트를 만드는 역할을 한다.
- Vitest에서는 DOM과 관련된 매처가 없기에 Testing Library의 jest-dom을 추가하여 매처를 확장해야한다.
- expect().toHaveClass(), expect().toBeInTheDocument() : 단언 부분

# 3. setup과 teardown
모든 테스트는 독립적으로 실행되어야 한다. 순서가 바뀌어도 다른 테스트에 영향이 없어야 한다!!

## setup: 테스트를 실행하기 전 수행해야 하는 작업
    1. beforeEach : 모든 테스트가 실행되기 전에 beforeEach함수 내에 작성된 코드가 실행된다.
    2. beforAll : 테스트 전에 실행되는건 beforeEach와 동일하지만, 파일과 스코프내에 실행 전에 단 한번만 호출된다.
## teardown: 테스트를 실행한 뒤 수행해야 하는 작업
    1. afterEach : 테스트가 완료될때 마다 실행된다.
    2. afterAll : 테스트가 완료될때 실행되는건 마찬가지지만 파일과 스코프내에 테스트 실행이 모두 완료된 후에 한번만 호출된다.

- setup, teardown에서 전역 변수를 사용한 조건 처리(아래예제 참고)는 독립성을 보장하지 못하고, 신뢰성이 낮아지므로 지양하자!
```
...
let someCondition = true;

beforeEach(async() => {
  if(someCondition) {
    await render(<TextField className="my-class" />);
  } else {
    ...
  }
});
```

# 4. React Testing Library와 컴포넌트 테스트
- 테스팅 라이브러리는 컴포넌트의 테스트를 손쉽게 할 수 있도록 도와주는 라이브러리이다.
- Testing Library의 핵심 철학은 **"UI 컴포넌트를 사용자가 사용하는 방식으로 테스트 하자"** 이다.
- 여기서 말하는 사용자가 사용하는 방식은 DOM 노드를 쿼리(조회)하고 사용자와 비슷한 방식으로 이벤트를 발생시키자의 의미이다.
```
import { render } from '@testing-library/react';
import userEvent from '@testing-library/user-event';

export default async component => {
  const user = userEvent.setup();

  return {
    user,
    ...render(component),
  };
};
```
- userEvent : 클릭, 키보드 이벤트등 다양한 이벤트를 실제 브라우저에서의 동작과 유사하게 시뮬레이션 할 수 있는 라이브러리이다.
- Spy 함수 : 테스트 코드에서 특정 함수가 호출되었는지, 함수의 인자로 어떤 것이 넘어왔는지 어떤값을 반환하는지 등 다양한 값들을 저장하고 있다.
