## 4.1 시각적 회귀 테스트를 위한 스토리북

#### 스토리북

- 오픈소스 UI 개발도구
- **스토리를 기준으로 시각적 회귀 테스트**를 실행할 수 있음
- 명확한 성공 실패에 대한 피드백이 없어 TDD에 적합하지 않음

#### 시각적 회귀 테스트

- UI 변경사항이 발견됐을 때 예상치 못한 상황을 방지하기 위한 테스트임
- 실제 렌더링 된 ui 결과의 이미지를 스냅샷으로 저장하여 비교 검증함
- 스토리북 공통 설정은 .storkbook 하위에서 정의 가능

## 4.2 스토리 작성하기

#### 스토리 작성대상

- props를 전달받아 UI만 렌더링하는 컴포넌트
- 복잡한 비즈니스 로직이 있는 컴포넌트를 스토리로 작성하면 과한 모킹 필요

#### 스토리

- CSF 포맷으로 작성
  - 스토리를 함수기반으로 작성하여 json 및 다른 형식보다 가독성이 좋고 유지보수가 쉬움
- 메타데이터는 default export, 스토리는 named export로 정의

#### Play

- 스토리를 렌더링한 후 사용자 상호 작용을 시뮬레이션 할 수 있음
  - testcode를 작성해서 동작을 컨트롤할 수 있음

## 4.3 크로마틱을 통한 UI 테스트 자동화

- 전문적인 도구를 사용해 시각적 회귀 테스트를 실행
  > 시각적 회귀테스트란, UI 변경사항이 기존 디자인과 비교하여 의도치 않게 변경되지 않았는지 픽셀단위로 자동감지하는 테스트 방식
- 변경 이력을 모두 저장하기 때문에 수정사항 히스토리 파악에 이점

#### 크로마틱

- 스토리북 메인테이너들이 만든 클라우드 기반 시각적 회귀 테스트 도구
- UI 컴포넌트가 변경될 때 이전 버전(스냅샷)과 비교하여 픽셀단위 차이를 자동으로 감지해줌

#### 시각적 회귀테스트가 필요한 이유는?

- 일반적인 단위테스트 or 스냅샷 테스트는 UI가 실제로 어떻게 변했는지 확인하기 어려움
- 크로마틱 등 시각적 회귀테스트 도구를 활용하면 UI 변경사항을 실제 눈으로 확인하고 승인, 거절이 가능함
- CI/CD와 연동하여 자동화도 가능

## 4.4 크로마틱을 활용한 시각적 회귀 테스트 워크플로우 만들기

#### 디자이너와 개발자간 협업 프로세스로 활용

- UI 변경사항에 대해 크로마틱에 배포
- 디자이너 or 협업 개발자에게 검증 요청 (Accept or Deny)
  - Deny 시 개발자가 다시 코드 수정 후 배포
  - Deny + comment 활용
- CI와 연동하여 deny된 UI 수정사항이 있을 경우 배포 금지

> 현실적으로 엔카 프로세스에 넣기 가능할까? 일정의 비효율?
> 디자인 서브잡에서 해보면 좋을듯

## 4.5 시각적 회귀 테스트의 한계

- 비용...
- BackstopJS 무료도구
  - 도커 사용 직접 환경 구축
- 이슈의 원인 파악이 어려움
  - 글로벌 css 변경으로 인한 UI 변화일 경우 등

> 모든 서비스보다 UI에 비중을 많이 두는 서비스에 적용해야 효율적일듯
