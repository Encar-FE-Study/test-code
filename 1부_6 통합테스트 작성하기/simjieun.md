# 1. 통합 테스트  작성하기 - ProductFilter
- beforeEach 에서 store 모킹
- store 액션은 스파이 함수를 이용함
- API 호출 후에 검증하기 위해선 findBy~ 쿼리를 사용함

# 2. 통합 테스트  작성하기 - NavigationBar
- API에 다른 모킹을 하기 위해 server.use()를 사용하여 커스텀
- afterEach에서 server.resetHandlers() 호출로 API 모킹 초기화

# 3. 통합 테스트의 한계 - 구매 페이지
- 테스트의 범위가 "페이지"로 커져 테스트를 위해 많은 모킹이 필요함.
- 통합 테스트의 비지니스 로직 검증 자체가 지나치게 모킹에 의존하게 됨.
- 통합 테스트는 비지니스 로직을 나누어 컴포넌트의 상호 작용을 검증하기 좋으나, 전체 워크 플로우를 검증하기에는 한계가 있음
- 이런 한계는 E2E 테스트로 해결할 수 있다.

# 4. GitHub Actions를 통한 테스트 자동화
```yml
name: 'vitest test'
on: pull_request
jobs:
  Component-test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
        with:
          fetch-depth: 0
      - uses: actions/setup-node@v3
        with:
          node-version: 19
      - name: Install dependencies
        run: npm ci
      - name: run vitest
        run: npm run test
```

# 5. 1부 마무리 하며
- 테스트 코드의 효과 : 리팩토링, 문서, 좋은 설계(테스트 단위에 대한 고민은 좋은 설계에 대한 사로 이어짐)
- 테스트 작성 시 중요한 규칙
  - 인터페이스 기준으로 테스트를 작성하자
  - 의미있는 테스트인지 고민하자
  - 테스트 코드의 가독성을 생각하자
- 결국 완벽한 테스트는 없으며 모든 테스트에는 한계가 존재한다.
- 이러한 각 테스트의 한계를 명확하게 인지하고 서로의 한계점을 보완하여 앱에 테스트를 작성하면 신뢰성을 높이면서 장기적으로 유지보수가 가능한 테스트를 작성할 수 있다.


