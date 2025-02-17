# 섹션 4 단위테스트 작성하기

## 단위 테스트 대상 선정하기
- 공통 컴포넌트, 리액트 훅, 공통 유틸
    - 다른 모듈과의 의존성이 없다.
    - 안정성을 높인다.

## 모듈 모킹
- 모의 모듈, 객체를 만드는 것
- 테스트를 실행 후에는 모킹 초기화를 하자.

## 리액트 훅 테스트(feat. act 함수)
- 단위 테스트 작성에 적합
- act 함수
    - 상호 작용(렌더링, 이펙트 등)을 함께 그룹화하고 실행하여 렌더링과 업데이트가 실제 앱과 유사하게 동작하도록 함
    - 렌더링 후 코드의 결과를 검증하고 싶을 때 사용
    - react-testing-library의 useEvent, Render

## 타이머 테스트
- 타이머 모킹 후 모킹 초기화 필요
    - useFakeTimers(), advanceTimersByTime(), setSystemTime(), useRealTimers()

## userEvent를 사용한 사용자 상호작용 테스트
- fireEvent
    - 특정 요소에서 원하는 이벤트만 쉽게 발생
    - 실제 상황과 거리가 있음
    - scroll 이벤트
- userEvent
    - 다양한 상호 작용 시뮬레이션 가능
    - disabled, display 상태 고려

## 단위 테스트의 한계
- 