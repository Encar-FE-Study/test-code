# 단위테스트란

## 2.1 단위 테스트란?

- 상세한 테스트 디스크립션으로 가독성 향상
- 내부 DOM 구조나 로직에 영향을 받지않게 테스팅 라이브러리 API를 통해 적절한 요소를 검증하자
- 최종 렌더링 결과물인 DOM 구조가 올바르게 변경되었는지 검증하자

## 2.2 테스트 환경과 매처

it, test, jsdom

- 단언 : 테스트가 통과하기 위한 조건ㄴ을 기술하여 검증을 실행한다

## 2.3 setup, teardown

setup: 테스트를 실행하기 전 수행해야 하는 작업
teardown: 테스트를 실행한 뒤 수행해야 하는 작업

- 테스트 실행 전, 후 실행되어야할 반복 작업을 깔끔하게 관리할 수 있다.
- 별도의 스코프로 동작하기 때문에 독립적인 테스트를 구성하는데 도움이 될 수 있다.
- 전역 변수를 사용한 조건 처리는 독립성을 보장하지 못하고, 신뢰성이 낮아지므로 지양하자

1. beforeAll - 테스트 실행전 한번만 실행 (주로 스코프내에서 전역으로 공유할 환경이나 상태를 설정할 때 사용 유용)
2. beforeEach - 테스트가 실행될 때 마다 실행
3. afterEach - 테스트가 완료될 때 마다 실행
4. afterAll - 테스트 실행 후 한번만 실행 (테스트에 의해 생성된 상태를 초기화하는데 사용)

블록으로 묶기 vs beforeEach

## 2.4 React Testing Library와 컴포넌트 테스트

UI 구성 요소를 사용자가 사용하는 방식으로 테스트는 테스트의 신뢰성은 향상된다.
=> 인터페이스를 기준으로 작성하자

- query에서도 우선순위가 있음(role, labelText, placeHolder, text 권장)

- vi.fn - 스파이 함수 - 테스트 코드에서 특정 함수가 호출되었는지, 함수의 인자로 어떤 것이 넘어왓는지 어떤 값을 반환하는지 저장

```jest
const spy = vi.fn();
const { user } = await render(<TextField onChange={spy} />);

await user.type(textInput, 'test');
expect(spy).toHaveBeenCalledWith('test');

await user.type(textInput, 'test{Enter}'); // enter 키 입력
```

spy 함수는

- 함수의 호출 여부, 인자, 반환 값을 등 함수 호출에 관련된 다양한 값을 저장
- 콜백 함수나 이벤트 핸들러가 올바르게 호출 되었는지 검증할 수 있음
  - toHaveBeenCalledWith, toHaveBeenCalled 매처
