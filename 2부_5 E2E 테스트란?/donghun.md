# 섹션 5 E2E 테스트란?

> # E2E 테스트란 무엇일까?

- E2E 테스트란 End to End 테스트의 줄임말
- 실제로 앱을 실행해 전체 소프트웨어 시스템의 전체 흐름을 검증
  - JSDom을 사용하지 않고, 실제 브라우저를 통해 테스트
- 사용자 입장에서 앱을 사용하면서 발생할 수 있는 시나리오가 실제 환경에서 정상적으로 작동하는 확인
- 직접 앱을 구동하기에 다른 테스트들에 비해 시간이 오래 걸림
- E2E 테스트의 장점
  - 사용자 관점에서 시나리오를 완벽하게 테스트할 수 있다.
  - 프런트엔드부터 백엔드까지 앱의 전반적인 상태를 확인할 수 있다.
  - 변경 사항이 전체 시스템에 미치는 영향을 확인할 수 있다.
- E2E 테스트 작성시 중요한 점

  - 유관부서 ( 특히 BE )의 협력이 필요하다.
    - 실제 서비스 및 디비에 영향을 주어선 안된다.
    - 검증용 계정이 필요할 수도 있다.
  - 가능한 API 모킹을 하지 않는다.
    - 백엔드 측과 협의가 어렵거나 실패 케이스 등의 검증이 필요하다면 일부 모킹을 통해 워크 플로우를 검증하는 것도 충분한 의미가 있다.
  - 도입을 위한 일정 확보
    - E2E 테스트는 FE와 BE 모두 전반적인 개발이 마무리 되었을 때 도입이 가능하다.

- 아마 BE의 협력이 없을 것으로 예상되는데, 모킹을 통해서 테스트한다면 정말 유효한 테스트일까?
- 모킹에 의존적이게 된다면 E2E 테스트의 장점이 희석되는 것이 아닐까?
- 데이터 환경이나 서버 세팅을 미리 준비를 해놔야하는데 괜찮을까 ?
- 위 제약사항들을 극복할 수 있는 방법은 무엇이 있을까 .... ?

> # Cypress로 E2E 테스트 시작하기

- Cypress
  - 자바스크립트를 사용하여 실제 웹앱에서 다양한 테스트를 작성할 수 있는 오픈소스 자동화 도구
- Head 모드
  - 브라우저 UI까지 모두 구동하여 시작적으로 확인할 수 있는 환경에서 테스트를 실행
  - 시각적으로 실행 과정을 확인하거나 디버깅 시에 이용
- Headless 모드
  - 브라우저 UI 없이 브라우저 엔진을 명령어 인터페이스로 제어
  - 구동속도가 상대적으로 빨라 CI 또는 클라우드 환경에서 구동하기 적합하다.

> # Cypress로 E2E 테스트 작성하기

- cypress에 필요한 기본 설정은 cypress.confing.js 파일에서 설정한다.

  ```js
  beforeEach(() => {
    // visit 함수는 url을 기준으로 웹에 접속할 수 있도록 도와주는 함수
    cy.visit("/login");
  });

  it('이메일을 입력하지 않고 로그인 버튼을 클릭할 경우 "이메일을 입력하세요" 경고 메세지가 노출된다', () => {
    cy.findByLabelText("로그인").click();
    // should('exist') => 존재하는지 단언하는 문법
    cy.findByText("이메일을 입력하세요").should("exist");
  });

  it('잘못된 양식의 이메일을 입력한 뒤 로그인 버튼을 클릭할 경우 "이메일 양식이 올바르지 않습니다" 경고 메세지가 노출된다', () => {
    // type => 이메일 입력
    cy.findByLabelText("이메일").type("qq");
    cy.findByLabelText("로그인").click();
    cy.findByText("이메일 양식이 올바르지 않습니다").should("exist");
  });

  it("회원 가입 클릭 시 회원 가입 페이지로 이동한다", () => {
    cy.findByText("회원가입").click();
    // cy.url 을 통해 현재 URL을 가져올 수 있고, should eq 단언을 통해 2번째 인자의 주소와 일치하는지 확인
    cy.url().should("eq", `${Cypress.env("baseUrl")}/register`);
  });

  it('성공적으로 로그인 되었을 경우 메인 홈 페이지로 이동하며, 사용자 이름 "Maria"와 장바구니 아이콘이 노출된다', () => {
    // 단위테스트와의 차이점
    // 1) 실제 로그인 API를 호출하여 로그인 성공 여부를 확인한다
    // 2) 페이지 이동을 검증한다
    cy.findByLabelText("이메일").type(Cypress.env("test-email"));
    cy.findByLabelText("비밀번호").type(Cypress.env("test-pw"));
    cy.findByLabelText("로그인").click();

    cy.url().should("eq", `${Cypress.env("baseUrl")}/`);
    cy.findByText("Maria").should("exist");
    cy.findByTestId("cart-icon").should("exist");
  });
  ```

> # Cypress와 쿼리

- https://docs.cypress.io/api/table-of-contents#Queries
- get 쿼리
  - cypress에서 자체적으로 제공하는 get 함수를 통해 DOM에 접근할 수도 있지만 DOM 구조 변경에 유연하게 대응이 불가함
  - 그러나 get으로 alias처럼 사용이 가능하다.
- cypress는 내부적으로 Promise 체이닝을 통하여 관리되며 커맨드가 시작될 때 각 subject는 비동기 대기열에서 대기하다가 실행이 된다.
- 따라서 변수에 담아서 체이닝하는 것은 불가능하며 체이닝 형태로 사용하거나 then을 사용하거나 해야한다.
- then 내부에서 반환하는 값은 새로운 subject가 되어서 다음 커맨드에서 사용된다.
- then 내부에서 반환하지 않았다면 기존의 subject가 다음 커맨드에서 사용된다.
- Retry-Ability
  - 잠재적으로 업데이트 가능성이 있다고 판단하여 특정 시간 재시도한다.
    - ex) API 응답 대기, 계산이 오래 걸리는 로직, DOM 미반영 , 등
  - 디폴트는 4초동안 재시도하며 {timeout:n} 을 인자로 넘겨주어 대기 시간 조절도 가능하다.
- Retry-Ability 여부에 따른 구분
  - Query : 전체 체이닝 로직을 재시도하며 수행
    - ex) get, cypress-testing-library, find등
  - Assertion : 단언을 수행하는 특별한 쿼리
    - ex) should 등
  - Non-Query : 재시도를 하지 않음
    - ex) visit, click, then 등
