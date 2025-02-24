# 섹션 3 단위 테스트란?

> # 단위 테스트란 무엇일까?

- 테스트 가능한 단일 함수의 결과값 또는 단일 를 실행하여 예상대로 동작하는지 확인하는 테스트
- 공통 컴포넌트들은 단위 테스트로 검증하기 적합하다
- Arrange, Act, Assert 패턴으로 테스트 코드를 작성

1. Arrange: 테스트를 위한 환경 만들기 (컴포넌트 렌더링)
2. Act: 테스트할 동작 발생 (사용자 인터랙션)
3. Assert: 올바른 동작이 실행 되었는지 또는 변경사항 검증하기

<<<<<<< HEAD
=======
![AAA에 따른 TextField 컴포넌트 테스트](/images/donghun/image.png)

>>>>>>> d9773f5 (1부_ 3 단위테스트란?)
> # 테스트 환경과 매처(Machter)

### vitest

- 별다른 설정없이 vite 기반에서 테스트 구동, ESM,TS,JSX 사용 가능
- Jest 호환
- 기본적으로 vitest의 실행환경은 vite의 설정을 그대로 따른다 (vite.config.ts)
- 별도의 설정은 test field에 정의할 수 있다.
- 정의해야할 내용이 많다면 vitest.config.js와 같이 따로 파일로 관리할 수 있다.

### jsDOM

- nodeJS에서 DOM이 제대로 렌더링이 되었는지 확인하기 위한 환경

### it 함수과 test 함수

- 테스트의 단위로서 내가 바라는 기대결과를 정의한다
- it 대신 test 함수를 사용할 수 있으며, it 함수는 test 함수의 alias이다.
- it 와 test의 차이는 영문으로 디스크립션을 작성할 때의 차이가 있다.
- it으로 시작하는 경우 "should ~~ " , test로 시작하는 경우 "if ~~~ " 꼴로 작성한다.
- it과 test의 기능상의 차이점은 없다.

### describe

- 테스트를 그룹화하는 함수로서 컨택스트를 만드는 역할을 수행한다

> # setup과 teardown

- 모든 테스트는 독립적으로 실행되어야한다.
- 이러한 테스트의 독립성을 보장하기 위해 setup과 teardown을 활용할 수 있다.
- setup : 테스트 실행 전 수행해야 하는 작업 (beforeEach, beforeAll)
- teardown : 테스트 실행 후 수행해야 하는 작업 (afterEach, afterAll)
- describe마다 독립적으로 setup과 teardown을 수행할 수 있다.
- all 함수는 each와는 다르게 단 한번만 호출되며, 같은 레벨 안에 each와 함께 존재한다면 all이 먼저 실행된다.
- beforeAll은 전역에서 설정해야할 것들을 설정하는 데에 유용함
- setup과 teardown을 전역으로 설정해주고 싶다면 vite config파일에서 설정해줄 수 있다.
- setup과 teardown에서 전역 변수를 사용한 조건 처리는 독립성을 보장하지 못하고 신뢰성을 헤치기 때문에 지양 해야한다.

> # React Testing Library와 컴포넌트 테스트

- UI 컴포넌트 테스트를 도와주는 라이브러리

```js
import { screen } from "@testing-library/react";
import React from "react";

import TextField from "@/components/TextField";
import render from "@/utils/test/render";

// Arrange - 테스트를 위한 환경 만들기
// Act - 테스트 할 동작 발생
// Assert - 올바른 동작이 실행되었는지 검증

describe("TextField", () => {
  it("className Props로 설정한 css class가 적용된다.", async () => {
    await render(<TextField className="my-class" />);
    // 특정 플레이스폴더를 지닌 요소를 조회한다.
    // 이렇게 특정 컴포넌트만을 조회하여 테스트한다면 내부 돔 구조와는 무관하게 원하는 테스트 요소만 테스트할 수 있다.
    expect(screen.getByPlaceholderText("텍스트를 입력해 주세요.")).toHaveClass(
      "my-class"
    );
  });

  it('기본 placeholder "텍스트를 입력해 주세요."가 노출된다', async () => {
    await render(<TextField />);
    const textField = screen.getByPlaceholderText("텍스트를 입력해 주세요.");
    // jest-dom lib를 사용하여 매처를 확장
    expect(textField).toBeInTheDocument();
  });

  it("placeholder prop에 따라 placeholder가 변경된다.", async () => {
    await render(<TextField placeholder={"플레이스홀더 테스트"} />);
    const textField = screen.getByPlaceholderText("플레이스홀더 테스트");
    expect(textField).toBeInTheDocument();
  });

  it("텍스트를 입력하면 onChange props로 등록한 함수가 실행된다", async () => {
    // 스파이 함수 - 테스트 코드에서 특정 함수가 호출되었는지, 함수의 인자로 어떤 것이 넘어왔는지, 어떤 값을 반환하는지에 대한 정보를 저장한다.
    const spy = vi.fn(); // 스파이 함수
    const { user } = await render(<TextField onChange={spy} />);
    const textField = screen.getByPlaceholderText("텍스트를 입력해 주세요.");
    await user.type(textField, "test");
    // 스파이가 올바르게 호출되었는지 확인하기 위하여 expect, toHaveBeenCalledWith 실행
    // 내가 원하는 테스트라는 문자열과 함께 올바르게 실행되었는지 알 수 있다.
    expect(spy).toHaveBeenCalledWith("test");
  });

  it("엔터키를 입력하면 onEnter prop으로 등록한 함수가 호출된다", async () => {
    const spy = vi.fn();
    const { user } = await render(<TextField onEnter={spy} />);
    const textField = screen.getByPlaceholderText("텍스트를 입력해 주세요.");
    await user.type(textField, "test{Enter}");
    expect(spy).toHaveBeenCalledWith("test");
  });
});
```
