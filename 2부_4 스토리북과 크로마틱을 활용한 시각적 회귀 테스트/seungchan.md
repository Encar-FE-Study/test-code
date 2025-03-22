## 1. 시각적 회귀 테스트란?

개념:

- UI에 변경이 생겼을 때 기존 스냅샷과 비교해서 예상치 못한 변화나 버그가 발생했는지 자동으로 검증하는 테스트 방법

원리:

- 컴포넌트가 렌더링된 결과를 이미지(스냅샷)로 저장 이후 UI 변경 시 이전 이미지와 비교하여 차이가 있으면 알림
- 장점: 수동 리뷰 없이 빠른 피드백 / UI 변경 이력을 관리할 수 있음
- 한계: 비용 및 관리 부담(무료 플랜의 경우 월 5000장 제한)
- 스냅샷 차이만으로 문제의 원인을 정확히 파악하기 어려움

## 2. 스토리북을 활용한 시각적 회귀 테스트

#### 1. 스토리북의 역할

UI 개발 도구:
스토리북은 단순 테스트 라이브러리가 아니라, 컴포넌트를 격리된 환경에서 렌더링해보며 개발할 수 있게 도와주는 도구
주로 브라우저 내의 독립된 영역(iframe)을 활용

#### 효율성 증대:

- 컴포넌트를 실제 서비스와 분리해 다양한 상황(스토리)별로 테스트할 수 있음
- MSW(Mock Service Worker)를 활용해 실제 API 호출 없이도 테스트 가능
  구성 파일:
  preview.jsx: 스토리 렌더링 시 공통 설정(글로벌 데코레이터, 파라미터 등)을 지정
  .storybook 폴더: 스토리북의 전체 환경 설정을 관리

#### 2. CSF(Component Story Format)로 스토리 작성하기

CSF의 핵심 구성요소는 메타데이터와 스토리, 그리고 선택적으로 play 함수

```
  메타데이터: 컴포넌트 이름, 컴포넌트 자체, 스토리의 기본 설정 등을 담음
  스토리: 각 상황(시나리오)에 따른 컴포넌트 렌더링 예시 => UI 모습 확인 용도
  play 함수: 스토리 렌더링 후 실제 사용자 상호작용을 시뮬레이션 할 수 있는 함수
=> 스토리가 화면에 렌더링 된 후 자동으로 실행됨
```

작성 대상: 스토리북은 주로 UI 검증에 초점을 맞추고 있기 때문에, 복잡한 로직이 포함되면 테스트 작성과 유지보수가 어려워질 수 있음

- 하위 컴포넌트를 별도로 작성하는 것을 추천 /UI 렌더링만 집중해서 스토리를 작성하는 것이 더 효율

```jsx
// Button.jsx
import React from "react";

const Button = ({ label, onClick, variant = "primary" }) => {
  return (
    <button className={`btn btn-${variant}`} onClick={onClick}>
      {label}
    </button>
  );
};

export default Button;
```

```jsx
// Button.stories.jsx
import React from "react";
import Button from "./Button";

export default {
  title: "Example/Button",
  component: Button,
};

// 컴포넌트 렌더링함수
const Template = (args) => <Button {...args} />;

// 기본 스토리
export const Primary = Template.bind({});
// 왜 bind를 사용하는가
// Template 함수의 복사본을 만들어 각각의 스토리에서 독립적으로 args(즉, props)를 설정할 수 있도록 하기 위해 사용
// 여기서는 this값 조작이 필요없으므로 {}만 전달해서 Template 함수의 복사본을 생성

Primary.args = {
  label: "Primary Button",
  variant: "primary",
};
// Primary.args에 값을 할당하면, Storybook 내부에서 Primary 스토리를 렌더링할 때 이 값을 Template 함수의 인자로 전달
// Primary 스토리가 실행시 Storybook은 내부적으로 Primary(Primary.args) 형태로 호출해서, Template 함수에 { label: 'Primary Button', variant: 'primary' }라는 값을 인자로 전달

// 두 번째 스토리
export const Secondary = Template.bind({});
Secondary.args = {
  label: "Secondary Button",
  variant: "secondary",
};

// play 함수를 이용해 클릭 이벤트를 시뮬레이션한 예시
export const WithClick = Template.bind({});
WithClick.args = {
  label: "Clickable Button",
  variant: "primary",
};
WithClick.play = async ({ canvasElement }) => {
  // canvasElement : story가 렌더링된 dom의 최상위 요소(보통 ifrmae의 root 요소)
  // Testing Library를 활용해 클릭 이벤트를 시뮬레이션 할 수 있음
  const canvas = within(canvasElement);
  const button = canvas.getByRole("button");
  userEvent.click(button);
  await waitFor(() => {
    expect(button).toHaveTextContent("Clicked");
  });
};
```

## 3.워크플로우 요약

- 스토리북 작성:
  CSF 방식으로 단순 UI 컴포넌트에 대해 스토리를 작성
  필요한 경우 play 함수를 활용해 사용자 상호작용 시뮬레이션
- Chromatic 연동:
  CI/CD 파이프라인에 크로마틱을 연동하여 PR 생성 시 자동으로 스냅샷 테스트 실행 -테스트 실행 및 검토:
  변경사항이 감지되면 크로마틱 대시보드에서 결과 확인
  예상치 못한 변경이라면 수정 후 재검증
  배포:
  테스트 완료 후 스토리북을 배포하여 팀원들이 최신 UI를 확인

스토리북의 한계:

- 단위 테스트나 통합 테스트처럼 코드를 통해 명시적으로 검증하기 어려움
  실제 사용 시 고려사항:
  컴포넌트의 UI만 검증하는 용도로 활용하는 것이 좋으며, 복잡한 비즈니스 로직 검증은 별도의 테스트 코드로 보완 필요

장점:
스토리북과 크로마틱을 통해 개발자는 UI 변경사항을 빠르게 감지하고 리뷰할 수 있어 생산성이 향상
스토리북은 컴포넌트 단위로 격리된 환경에서 다양한 상황을 테스트할 수 있어 개발 및 디자인 검증에 유용함

보완점:
복잡한 로직이 포함된 컴포넌트의 경우, 스토리 작성 시 모킹이 필요하거나 하위 컴포넌트로 분리하는 등 구조적 개선이 필요
비용과 피드백 속도 등 시각적 회귀 테스트의 한계를 이해하고, 프로젝트 특성에 맞게 보완할 필요 있음

## CSF(component stroy format)

- Storybook에서 스토리를 작성하는 표준 포맷(권장하는 규칙)
- 구조 및 구성요소

### 1. Default Export:

컴포넌트의 메타데이터(제목, 컴포넌트 등)를 포함하는 객체를 default export로 처리

```jsx
export default {
  title: "Components/Button",
  component: Button,
};
```

### 2. Named Export로 스토리 정의:

각각의 스토리는 named export로 정의하고, 보통 템플릿 함수를 사용해 컴포넌트를 렌더링하는 방식으로 처리(보통 props받아 처리)

```jsx
const Template = (args) => <Button {...args} />;

// 기본 스토리
export const Primary = Template.bind({});
Primary.args = {
  label: "Primary Button",
  variant: "primary",
};
```

### bind의 역할:

```js
Template.bind({});
```

- Template 함수의 복사본을 생성(부수효과:새로운 함수의 this 값이 빈 객체({})로 설정 / 함수 내부에서 this 참조하는 거 아니면 큰 의미 XX)
  -> 여러 스토리가 같은 함수 인스턴스를 공유하지 않게 됨
  -> 각 스토리마다 별개의 함수 인스턴스를 가지므로 테스트가 독립적
