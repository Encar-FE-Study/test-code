# 섹션 5 통합테스트란?

## 단위 테스트의 한계
- 단위테스트에 비해 모킹의 비중이 적으며, 모듈 간에 발생하는 에러를 검증 가능
- 실제 앱이 동작하는 로직과 유사하게 기능을 검증 가능
- 단위 테스트 보다 넓은 범위의 테스트
- 좋은 통합테스트를 위해서는 좋은 설계가 기반이 되어야 함

## 통합 테스트 대상 선정하기
- 비즈니스 로직을 기준으로 통합테스트 작성 권장
- 비즈니스 로직 분리 시 참고 사항
    - 가능한 모킹 줄이고 최대한 실제와 유사하게 검증
    - 비즈니스 로직을 처리하는 상태관리나 API 호출은 상위 컴포넌트로 응집해 관리
    (ex. 네비게이션 바에서 사용자 정보 호출을 하여 로그인/로그아웃 버튼으로 전달)
    - 도메인 기능이 조합된 비즈니스 로직은 나누어 통합 테스트 작성

## 상태 관리 모킹하기

## 통합 테스트 작성하기 - 상태 관리 모킹
- queryBy~ : 요소의 존재하지 않아도 에러 X

## msw로 API 모킹하기

## RTL 비동기 유틸 함수를 활용한 테스트 작성
- findBy : 비동기 처리로 인한 변화 감지에 사용, await 사용 필요
- 다른 페이지의 로직의 경우 모킹 처리

## 테스팅 할 때 중첩은 피하세요
- [avoid nesting when you're testing](https://jaehyeon48.github.io/testing/avoid-nesting-when-youre-testing)
    - 간단한 컴포넌트의 경우 추상화를 최대한 제거하는 것(Over-abstraction 경계)
    - 단순히 코드 재사용을 위해 befareAll, afterall 같은 유틸리티를 사용하지 말 것
    - 테스트를 실패해도 클린업 할 수 있도록 afterEach 사용할 것
- [AHA (Avoid Hasty Abstractions) 적용하기](https://kentcdodds.com/blog/aha-testing)
    - 잘못된 추상화보단 중복을 허용하고, 변경에 대한 최적화를 먼저 해야 함 (prefer duplication over the wrong abstraction and optimize for change first)
    - ANA(Absolutely No Abstraction)
    - AHA(Avoid Hasty Abstraction)
    - DRY(Don't Repeat Yourself)
        - describe, it nesting + beforeEach의 남용
- 그룹핑 테스트
    - describe 보다 파일 분리를 통해 셋업 분리, 인지 부하 감소시킴
- 클린업
    - 테스트 실패 유무와 cleanup 함수의 의존성 분리를 통해 테스트 안정성 확보
    - 코드의 재사용을 위해 before*, after* 유틸 함수 사용하지 말 것
- [test isolation with react](https://kentcdodds.com/blog/test-isolation-with-react)

