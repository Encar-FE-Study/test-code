# Cypress를 활용한 E2E 테스트 작성하기

> # 커스텀 커맨드와 쿼리

- 반복되는 로직을 커스텀 커맨드와 커스텀 쿼리로 정의하여 사용할 수 있음
- 커스텀 쿼리
  - Retry-ability 지원
  - 동기로 동작하며, subject 결과를 받아 내부적으로 체이닝 코드를 일정시간 재시도
  - Cypress.Commands.addQuery()와 Cypress.Commands.overwriteQuery()로 정의
- 커스텀 커맨드
  - Retry-ability 미지원
  - 비동기로 동작할 수 있으며, 특별한 설정이 없다면 subject를 이어받지 않는다.
  - Cypress.Commands.add()와 Cypress.Commands.overwrite()로 정의

> # E2E 테스트 작성하기 - 메인 홈 페이지

-

> # 서버 요청 가로채기

> # E2E 테스트 작성하기 - 구매 페이지

> # E2E 테스트 작성하기 - 권한 체크
>
> # E2E 테스트의 한계
>
> # 테스트 더블
