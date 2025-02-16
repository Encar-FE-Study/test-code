## 2.1 단위테스트란?

- 앱에서 테스트 가능한 가장 작은 sw(단일함수 또는 단일컴포넌트)를 실해해 예상대로 동작(결괏값 도는 상태, 행위를 검증)하는지 확인하는 테스트
- 아토믹 컴포넌트들이 테스트코드로 작성하기 좋음
- 공통 컴포넌트는 단위테스트에 적합함 (범용적으로 사용되기 때문에 단위테스트를 통해 안정성을 높여야함)

### Arange-Act-Assert 테스트 작성 패턴

- Arrange: 테스트를 위한 환경
- Act: 테스트할 동작 발생
- Assert: 올바른 동작이 실행되었는지 또는 변경사항 검증

### 테스트 프레임워크

- Jest, Vitest, Cypress 등등
- RTL의 screen.debug()를 통해서 dom구조를 확인할 수 있음
- describe > it(should~~) === test(if~~)

### 매처

- 기대 결과를 검증하기 위해 사용되는 API 집합
- 기본 vitest, jest의 매처에는 DOOM과 관련된 매처는 없음
- TL의 jest-dom을 활용해 DOM과 관련된 매처를 import
- https://vitest.dev/api/expect.html
- https://github.com/testing-library/jest-dom#custom-matchers

### setup과 teardown

- setup: 테스트 실행 전 수행해야 하는 작업
  - beforeEach, beforeAll
- teardown: 테스트를 실행한 뒤 수행해야 하는 작업
  - afterEach, afterAll

## React Testing Library와 컴포넌트 테스트

- 사용자 동작과정에 대한 테스트를 작성

- toHaveStyle에서 borderWidth는 인식을 못하는 문제

```code
it('focus가 발생 시 border 스타일이 추가된다.', async () => {
    const { user } = await render(<TextField />);

    const textInput = screen.getByPlaceholderText('텍스트를 입력해 주세요.');

    await user.click(textInput);

    expect(textInput).toHaveStyle({
      borderWidth: 10,
      borderColor: 'rgb(25, 118, 210)',
    });
  });
```

- borderWidth는 '2px'과 같이 정확한 표현을 사용해야 정확한 검증이 가능함
  - vitest, jest의 toHaveStyle은 borderWidth에 정확한 px값을 이용하지 않으면 속성이 있는지 여부만 확인해서 테스트를 통과시킴
- Spy 함수를 통해 함수의 호출여부, 인자, 반환값 등 함수 호출에 관한 다양한 값을 저장
  - toHaveBeenCalledWith, toHaveBeenCalled
