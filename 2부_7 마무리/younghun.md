# 섹션 7 마무리

## 어떻게 프런트엔드 테스트를 작성할 것인가?

### 단위 테스트
- 단일 함수의 결괏값 또는 단일 컴포넌트(클래스)의 상태(UI)나 행위를 검증
- 대상: 공통 컴포넌트
  - 다른 컴포넌트와 상호작용이 없음
- 단위 테스트 전략
  - 별도 로직없이 UI만 그리는 컴포넌트는 검증하지 않음(스토리북)
  - 간단한 로직 처리만 하는 컴포넌트는 상위 컴포넌트의 통합테스트에서 검증
- 한계
  - 여러 모듈이 조합 되었을 때 발생하는 이슈는 찾을 수 없음
  - 앱의 전반적인 기능 동작 보장 X

### 통합 테스트
- 두 개 이상의 모듈의 상호작용 또는 컴포넌트가 조합 되었을 때 비즈니스 로직이 올바른지 검증하는 테스트
- 대상: 상태나 데이터를 관리하는 컴포넌트
  - 특정 상태를 기준으로 동작하는 컴포넌트 조합
  - API와 함께 상호작용하는 컴포넌트 조합
- 추가로, 단순 UI 렌더링 및 간단한 로직을 실행하는 컴포넌트까지 한 번에 효율적으로 검증 가능
- 통합 테스트 전략
  - 가능한 모킹하지 않음
  - 비즈니스 로직을 처리하는 상태 관리나 API 호출은 상위 컴포넌트로 응집
  - 변경 가능성을 고려해 여러 도메인의 기능이 조합된 비즈니스 로직은 나눠 작성하는 것이 좋음
- 한계
  - 전체 워크플로우를 검증하기 어려우며, 검증하더라도 모킹에 의존하게 됨

### 시각적 회귀 테스트
- 실제 렌더링된 UI 결과 이미지를 스냅샷으로 저장해 비교
- 스토리북과 같은 컴포넌트 UI 개발 도구를 연동 가능
- 실제 UI만 렌더링하는 컴포넌트를 대상으로 실행
- 크로스 브라우징 또는 스타일 변경으로 UI가 자주 틀어지는 컴포넌트 검증
- 한계
    - 관리 공수 및 비용 부담
    - 디버깅 어려움
    - 실행 시간 오래걸림(CI 연동 필수, TDD 사이클 불가능)
- 예시: percy

### E2E 테스트
- 실제 앱을 구동해 전체 소프트웨어 시스템 전반의 흐름을 검증
- 앱을 사용하는 다양한 시나리오 검증
- E2E 테스트 전략
  - 가능한 모킹하지 않음
  - API 호출이 어렵거나 외부 앱 사용하는 경우 스터빙
- 한계
  - 실행 시간 오래 걸려 개발 생산성 저하
  - 통제하기 힘든 외부 환경요소 
  - 디버깅 시간 오래걸림

### 개발 프로세스와 테스트
- 개발 시작 
  - 단위, 통합 테스트
  - 스토리북을 통한 UI 검증
- 개발 마무리 
  - E2E 테스트 도입
  - 시각적 회귀 테스트를 통한 UI 검증 (운영에서 UI 이슈가 발생하는 부분)

## Best Practices 분석

### Organizing Tests, Logging In, Controlling State
- 안티 패턴: page 객체 공유 패턴, 로그인 시 UI 사용, shortcuts를 사용하지 않는 것
- 베스트 프랙티스 : 테스트 케이스를 독립화, 

### Selecting Elements
- 안티 패턴: 깨지기 쉬운 선택자를 사용하는 것
- 베스트 프랙티스: data-* 속성을 활용하여 테스트에서 사용하는 요소를 선택할 때, UI의 변화에 유연한 선택자를 사용하는 것이 중요

### Assigning Return Values
- 안티 패턴: 커맨드의 리턴 값을 할당하는 것
- 베스트 프랙티스: aliases를 사용하거나 클로저를 통해 저장된 것에 접근

### Visiting External Sites
- 안티 패턴: 통제불가능한 외부 사이트와 상호작용하는 것
- 베스트 프랙티스: cy.request를 통해 서드파티 서버 API 호출, cy.session을 통해 캐싱으로 반복된 방문 방지하기 

### Having Tests Rely On The State Of Previous Tests
- 안티 패턴: 여러가지 테스트들을 연결
- 베스트 프랙티스: 독립적인 테스트 작성, 반복적인 부분 before & beforeEach 훅 사용

### Using after Or AfterEach Hooks
- 안티 패턴: 상태를 clean up 하기위해 after&afterEach 훅 사용하는 것
- 베스트 프랙티스: 테스트 실행전 상태 clean up 하기

### Unnecessary Waiting
- 안티 패턴: cy.wait(Number)를 통해 임의의 시간을 대기하는 것
- 베스트 프랙티스: route aliases 사용 혹은 Cypress guard를 통해 특정 케이스들이 충족되기 전까지 대기하는 것

### Setting a Global baseUrl
- 안티 패턴: cy.visit에 url 직접 명시
- 베스트 프랙티스: Cypress config에 baseUrl 설정
