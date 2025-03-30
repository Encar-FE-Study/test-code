# 6. cypress 를 활용한 E2E 테스트 작성하기

## 5.1 커스텀 커맨드와 쿼리

## 5.2 E2E 테스트 작성하기 - 메인 홈페이지

- setup

  - cy.visit('/')을 통해 메인 페이지로 이동

- findByText('장바구니')
- assertUrl('/login')
- getProductCardByIndex(0)
- getFn()
- cy.getCartButton().should('have.text', '');
- should('exist')

로그인 command에서 session

쿠키, local storage, session storage 를 캐싱 가능함 : session

cy.session(username, () => {

})

session 을 사용하지 않으면 login을 하게되어서 비효율적

## 5.3 서버 요청 가로채기

회원가입 워크플로우 테스트 같은 경우는 가입과 탈퇴를 한 프로세스에서 검증하는 것이 좋음

e2e 테스트에서는 모킹을 하지 않는것이 좋지만 사용할 수 밖에 없는 경우가 있음.

=> cy.intercept 라는 api를 사용함

(spy, stubbing을 한다 표현함)

stubbing: 특정 네트워크 요청에 대해 미리 정해진 응답을 반환하는 것
spying: 요청과 응답에 대한 호출 정보를 기록해 두는 것

e2e 테스트에서 모킹이 유용한 경우

- 워크플로우 실행 중 실패 케이스를 검증하는 경우
- 워크플로우 내에서 서드파티 API나 외부 앱을 사용하는 경우

## 5.4 E2E 테스트 작성하기 - 구매 페이지

사전작업

- 로그인
- 상품을 장바구니에 담기
- 구매하기 페이지로 이동

=> 커스텀 커맨드와 쿼리로 모두 자동화하여 테스트

```jest
beforeEach(() => {
    cy.findAllByRole('table').eq(1).findAllByRole('row').as('itemList')
})
it('구매하려는 상품의 정보가 나타난다', () => {
    cy.get('@itemList')
        .eq(0)
        .findByText('Handmade Cotton Fish')
        .should('exist')
    cy.get('@itemList').eq(0).findByText('1개').should('exist');
})
```

- alias 는 테스트 실행 전에 해당 블록 내에서 항상 초기화됨

## 5.5 E2E 테스트 작성하기 - 권한체크

권한을 가진 계쩡만 준비되어있다면 E2E 테스트 환경에서 빠르게 검증할 수 있음

E2E 테스트로 자동화하면 좋다

layout, Auth.cy.js

```js
const { isLogin } = useUserStore(
  (state) => pick(state, 'setIsLogin', 'isLogin'),
  shallow
);
if (authStatus === authStatusType.NEED_LOGIN && !isLogin) {
  return <Navigate to={pageRoutes.login} />;
}
```

## 5.6 E2E 테스트의 한계

- 단위, 통합 테스트에 비해 느린 속도 => 개발 생산성 저하
- 외부 환경 요소로 인해 테스트가 깨질 수 있음 => 온전한 테스트 실행을 위한 관리 비용이 많이 들어감
- 디버깅 시간이 오래 걸림

## 5.7 테스트 더블

영화 촬영 시 위험한 역할을 대신하는 `스턴트더블`에서 착안

=> 테스트더블

- Dummy
- Stub
- Spy
- Mock
- Fake

### Dummy

테스트 환경에서 특정 모듈 필요하지만 해당 모듈의 구현이나 기능 실행까지는 필요없는 경우 사용

### Stub

더미 객체와 다르게 모듈이 호출될 때 정해진 값을 반환하도록 함 => 에러처리나 다양한 응답의 케이스를 고려하여 구현된 것은 아님

### Spy

스텁을 조금 더 고도화. 구현된 객체의 호출 정보까지 기록

vi.fn으로 생성된 함수 => 스텁이자 스파이

### Mock

실제 모듈과 유사한 행동을 하도록 만들어진 모의 객체

모의 객체는 스텁처럼 정해진 값만 반환하는 것이 아니라 기대되는 예상 동작에 대한 구현을 완료한 것

vi.mock

행동기반으로 검증

### Fake

특정 모듈을 테스트 전용으로 만든 단순한 모듈로 대체하는 것

실제 프로덕션에선 사용해선 안됨

### 테스트 더블의 장점

- 실제 구현체를 사용하지 못할 떄 적절한 테스트 더블로 대체해 검증할 수 있다
- 테스트 코드와 외부 의존성 모듈을 분리할 수 있다
- 예외 처리에 대한 상황을 쉽게 재현하여 검증할 수 있다
