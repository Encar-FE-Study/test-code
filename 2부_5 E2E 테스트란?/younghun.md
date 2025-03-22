# 섹션 5 E2E 테스트란?

## E2E 테스트란 무엇일까?
- 가능한 API 모킹 최소화
  - 모든 워크 플로우를 검증하는 것이 핵심

## Cypress로 E2E 테스트 시작하기
- head 모드
  - 브라우저 UI까지 모두 구동
  - 디버깅 시 용이
- headless 모드
  - 브라우저 엔진을 명령어로 제어
  - 구동속도가 빨라 CI 또는 클라우드 환경에 적합
 
## Cypress로 E2E 테스트 작성하기
- Cypress는 내부적으로 chai, chai-jQuery, sinon-chai를 확장하여 사용

## Cypress와 쿼리
- get
  - css selector로 DOM에 접근 
  - alias로 선언한 요소에 접근 가능
- then
  - subject 객체는 시작 지점 또는 대상이 되는 요소를 의미
  - 이전 커맨드의 subject의 실행 결과를 전달받아 사용
  - 아무것도 반환하지 않을 경우 기존 subject 그대로 사용
- Retry-ability
  - subject 덕분에 타이밍 문제 없이 재시도하여 얻은 결과를 순차적으로 사용할 수 있음
  - 일관된 환경을 유지하여 테스트하기 위함
  - Query: 전체 체이닝 로직을 재시도하여 실행 (ex. get, cypress-testing-library, find)
  - Assertion: 단언을 수행하는 특별한 쿼리의 유형 (ex. should)
  - Non-Query: 재시도를 하지 않는 쿼리 (ex. visit, click, then)

