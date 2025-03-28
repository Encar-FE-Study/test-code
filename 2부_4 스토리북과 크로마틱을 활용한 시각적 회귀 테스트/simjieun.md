# 4. 스토리북과 크로마틱을 활용한 시각적 회귀 테스트

## 4.1 시각적 회귀 테스트를 위한 스토리북
- 스토리북을 이용하여 스토리 기준으로 시각적 회귀 테스트를 실행할 수 있다.
- 스토리북은 TDD에 적합한 도구인가?
  - 각 테스트 케이스에 대한 정의가 어려움
  - 명확한 성공.실패에 대한 피드백이 없음
  - 테스트코드를 작성하고 코드 짜고 리팩토링하는 그런 과정을 스토리북으로는 할 수 없다고 생각한다.
- 스토리북이 TDD에 적합하지 않지만, 컴포넌트 개발의 효율성과 정확성은 높여준다.
- 시각적 회귀 테스트란?
  - UI 변경 사항이 발생했을 때 기존과 다른 점이 있는지 비교하여 검증하고, 예상치 못한 문제가 있는지 반복적으로 검증하는 테스트이다.
  - 한자어는 돌아올 회, 돌아갈 귀로 되돌아간다는 뜻이다.

## 4.2 스토리 작성하기
- 일반적으로 .stories.(tsx|ts|jsx|js) 로 끝나는 파일에 작성한다.
- ES6 모듈 기반의 CSF(Component Story Format)으로 작성한다.

#### CSF의 핵심 구성요소
1. 메타 데이터 : 이름이나 동작에 대한 정보를 설명
2. 스토리 : 각각의 시나리오를 나타냄

#### Play
- 스토리를 렌더링한 후 사용자 상호 작용을 시뮬레이션 할 수 있도록 도와줌

#### 스토리 작성 대상
- 단순하게 UI만 렌더링 하는 컴포넌트의 시나리오를 작성하자.
- 비즈니스 로직이 응집되어 있는 컴포넌트의 경우 별도 준비 과정이 필요해 작성이 어려울 수 있다.

## 4.3 크로마틱(Chromatic)을 통한 UI 테스트 자동화
- 시각적 회귀 테스트는 전문적인 도구를 사용해 스냅샷을 만들고 검증하는 것이 좋다 (ex. Chromatic, Applitools, Percy, BackstopJS)

#### 크로마틱
- 스토리북 메인테이너이 만든 시각적 회귀 테스트 도구이다.
- 스토리북, CI와의 연동이 매우 편하다.
- 함께 작업하기 좋은 워크플로우 환경을 제공한다.

#### 크로마틱 사용법
1. 깃헙계정으로 가입
2. Add project -> Create a project -> 토큰을 발급해줌
3. 발급받은 토큰으로 테스트코드 깃헙에서 package.json에서 복붙
4. run chromatic해서 완료되면 크로마틱사이트에서 결과물을 확인할수있음(짱신기)

## 4.4 크로마틱을 활용한 시각적 회귀 테스트 워크플로우 만들기

#### 시각적 회귀 테스트 워크플로우
1. 스토리북 작성
2. Chromatic CI 연동
3. UI 회귀 테스트 실행
4. PR 승인 및 머지
5. 스토리북 배포

## 4.5 시각적 회귀 테스트의 한계

1. 비용과 관리에 대한 부담 : 월5000장만 무료
2. 이슈의 원인을 파악하는데 오랜 시간이 걸린다 : UI결과를 검증하기 때문에 어떤 부분이 문제인지 파악하기 어렵다.
3. 피드백이 느림 : 프로젝트 개발단계에서 도입할수없고, 모두 개발이 어느정도 완료된 상태에서 도입해야 효과가 있다.
