# 1. 통합 테스트  작성하기 - ProductFilter
- beforeEach 에서 store 모킹
- store 액션은 스파이 함수를 이용함
- API 호출 후에 검증하기 위해선 findBy~ 쿼리를 사용함

# 2. 통합 테스트  작성하기 - NavigationBar
- API에 다른 모킹을 하기 위해 server.use()를 사용하여 커스텀
- afterEach에서 server.resetHandlers() 호출로 API 모킹 초기화


