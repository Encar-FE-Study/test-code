## 1. Retry-ability란?
Cypress의 기본 설계
Cypress의 모든 쿼리 명령(get, findBy* 등)은 내부적으로 `자동 재시도(폴링)`를 수행
Cypress는 DOM을 `지속적으로 감시(polling)`하여 요소가 렌더링될 때까지 기다림

```jsx
cy.get('.btn')은 .btn이 나타날 때까지 (기본 4초 동안) 계속 시도
이 덕분에 비동기 UI 렌더링에도 불구하고 테스트가 안정적
```

```jsx
cy.get('.loading-finished').should('exist');
.loading-finished 클래스가 비동기 작업 이후 DOM에 나타날 경우

Cypress는 해당 요소가 존재할 때까지 계속 .get() → .should()를 재시도
```

## 2. Cypress는 "Promise-like" 체이닝 기반
Cypress의 모든 커맨드는 내부적으로 `명령 큐(command queue)`에 등록되고 자동으로 체이닝됨.
`기존 Promise와는 다르지만, 사용자가 보기엔 비슷한 문법`

```jsx
cy.get('input')   // 요소 찾기
  .type('Hello')  // 텍스트 입력
  .should('have.value', 'Hello'); // 값 확인
차이점:
Cypress의 명령들은 즉시 실행되지 않고, 내부 큐에 쌓여 실행처리됨

그래서 await이나 .then()을 잘못 섞으면 타이밍 이슈가 생길 수 있다.
```

## 3. then()의 역할과 원리
### .then() 사용 시기
- `이전 커맨드의 실행 결과(subject)`를 사용하고 싶을 때
- Cypress 명령어와 JavaScript 로직을 연결할 때
- 비동기 체이닝 안에서 Cypress 커맨드를 섞고 싶을 때

```jsx
ex)
cy.get('input').then($input => {
  const val = $input.val();
  expect(val).to.equal('');
});
.get()이 반환하는 subject(jQuery 객체)가 .then()의 콜백 인자로 넘어옴

여기서 Cypress 외부 로직(JS 조건문, 로깅 등)을 수행 가능
```

## 4.then() 예시
```jsx
1.DOM 속성 사용
cy.get('input').then($el => {
  const placeholder = $el.attr('placeholder');
  expect(placeholder).to.include('이메일');
});

2.여러 커맨드 조합
cy.get('.product')
  .then($els => {
    return Cypress._.map($els, el => el.innerText);
  })
  .then(productNames => {
    expect(productNames).to.include('MacBook Pro');
  });
위 예제에서 .then() 체이닝 시, 리턴값이 다음 커맨드의 subject가 됨

3. 조건 분기 로직
cy.get('.user-badge').then($el => {
  if ($el.length) {
    cy.wrap($el).should('contain.text', 'Admin');
  } else {
    cy.log('No user badge found');
  }
});
if 조건문을 써야 할 때 Cypress 명령어 사이에 끼워넣기 적절

4.API 호출 결과에 따라 분기
cy.request('/api/user').then(({ body }) => {
  if (body.name === 'maria') {
    cy.visit('/dashboard');
  } else {
    cy.visit('/login');
  }
});
```


## Cypress의 Command Queue란?

### Cypress의 핵심 구조
```jsx
cy.get('input').type('hello').should('have.value', 'hello');
➡ 위 코드는 즉시 실행되지 않음
➡ Cypress는 내부적으로 **명령 큐(Command Queue)**에 차례대로 등록만 처리
➡ 그 후 Cypress 엔진이 이 명령들을 순차적으로 실행

cy.get('input');
cy.type('hello');
cy.should('have.value', 'hello');

Command Queue 상태:
[1] get('input')
[2] type('hello')
[3] should('have.value', 'hello')
Cypress는 이 큐를 읽고 하나씩 실행하면서, subject를 다음 명령으로 넘기고
이런 구조 덕분에 Cypress는 테스트 실행 흐름을 제어 가능
→ Cypress 명령은 동기처럼 보이지만 비동기적으로 동작
```


## 명령 큐는 뭐고 이벤트 큐와는 무슨 상관이 있나?
### Cypress는 JS의 이벤트 루프와 완전히 별개로 동작
- `비슷해 보이지만 완전히 다른 개념`

- `Cypress와 Event Loop는 어떻게 상호작용할까?`
    -  Cypress는 내부적으로 JS의 이벤트 루프를 감지하고 기다리는 로직을 갖고 있다.
    -  예를 들어 React에서 상태가 바뀌며 DOM이 재렌더링되는 순간을 Cypress는 알아차리고 기다릴 수 있다.

```jsx
cy.get('.loading-finished').should('exist');
.loading-finished 요소가 DOM에 나타날 때까지 자동으로 기다리는 건, 
Cypress가 내부적으로 비동기 처리 완료를 감지하며 명령 큐를 실행하기 때문
```


```
Command Queue: Cypress 내부 테스트 실행을 위한 큐
→ Cypress가 직접 실행 순서를 제어

Event Queue: JS의 비동기 콜백들을 실행하는 시스템
→ 브라우저/Node가 제어 (콜스택 비면 실행)
```
- Cypress는 자체 명령 큐를 사용하면서도 JS의 Event Queue를 감지하여
DOM 변경, 렌더링 등 비동기 타이밍을 자동으로 기다림
→ Cypress의 Retry-ability와 안정성의 원리
- Cypress는 JS의 await나 setTimeout, wait() 없이도 자동으로 UI의 비동기 변화를 감지해서 기다려줌
- 브라우저 Event Loop과는 독립적이면서도 "협력하는" 구조이기 때문


# 궁금증
## 1.Cypress의 Command Queue는 어떻게 브라우저 DOM에 접근할 수 있나?

Cypress는 테스트 실행 시 브라우저에 자바스크립트 코드(클라이언트 사이드 코드)를 주입
따라서 Cypress의 Command Queue는 브라우저 내부 JS 컨텍스트(V8 엔진) 안에서 실행되는 코드에 접근 가능
즉, Cypress는 테스트 명령을 브라우저 안에서 실제 DOM을 제어하거나 관찰하는 코드로 변환해서 실행처리

```jsx
Node.js (Test Runner)
   │   테스트 명령 생성 (Command Queue)
   ▼
Cypress 브라우저 주입 스크립트 (Client)
   │   명령 전송 받아 실행
   ▼
브라우저 DOM 조작 (실제 사용자와 동일한 레벨)
```
그래서 Cypress는 iframe 기반으로 실제 앱을 브라우저 안에 렌더링 후,
Cypress 테스트 코드가 같은 DOM에 직접 접근해서 조작/검사 가능
이게 Selenium과 Cypress의 가장 큰 차이점 중 하나

## 2.Cypress가 실행되면 브라우저의 Event Loop도 같이 실행될까?
Cypress가 명령을 실행하는 중에도 브라우저의 Event Loop는 그대로 작동함
Cypress는 DOM에 직접 접근해서 이벤트를 발생시키거나 관찰할 뿐이고,
React/Vue 같은 UI 프레임워크는 브라우저의 이벤트 루프 기반으로 상태를 바꾸고 DOM을 갱신 처리

```
ex)
Cypress가 .click() 실행 → 브라우저 DOM에 실제 click 이벤트 발생

React는 이 이벤트를 받고 setState 처리 → 내부적으로 비동기적으로 상태 업데이트 예약

브라우저 Event Loop가 돌면서 DOM 업데이트 수행

Cypress는 DOM이 바뀌는 걸 감지하고 .should() 실행

➡ Cypress와 Event Loop는 동시에 작동하면서 상호작용함
```

## 3. "비동기 UI 상태 변화 감지"란?
정의:
버튼 클릭, API 요청, 상태 변경(setState 등) 같은 비동기 이벤트 이후,
**화면에 변경된 결과(DOM 요소나 텍스트 등)**가 반영될 때까지 Cypress가 감지하며 자동으로 기다림
```jsx
cy.get('button').click();
cy.contains('저장 완료').should('exist');

1.Cypress가 .click() 실행
2.클릭 → onClick → fetch() → setState() → 렌더링 예약
3.브라우저의 Event Loop가 상태 변경 → DOM 반영
4,Cypress는 DOM에 저장 완료 텍스트가 나올 때까지 재시도
5.나타나면 .should('exist') 통과

->Cypress의 Retry-ability + DOM 감지 시스템이 작동하는 방식


┌──────────────┐          ┌────────────────────────────┐
│ Cypress Queue│  ─────▶  │  Cypress Client (Browser)  │
└──────────────┘          └────────────┬───────────────┘
                                       │
        (명령 실행)                    ▼
                                ┌────────────┐
                                │  DOM 조작  │
                                └────┬───────┘
                                     ▼
                        ┌────────────────────────┐
                        │ 브라우저 Event Loop    │ ← 상태 업데이트, 렌더링 처리
                        └────────────────────────┘
                                     ▲
                        (Cypress가 변화 감지하며 기다림)
```

## 4.Cypress와 브라우저의 Event Loop가 동시에 작동하면 흐름이 꼬이는 건 아닌가?

- 결론부터 말하면 Cypress는 브라우저의 Event Loop를 방해하지 않지만, 동기화(=interaction sync)를 "보장"하기 위해 특별한 설계를 가지고 있으므로 괜찮다.

즉 흐름이 깨지지 않게 자동 제어함,하지만 Event Loop의 완료를 Cypress가 직접 ‘보장’하지는 않음

대신 Cypress는 `DOM 변화 결과를 감지하며 "적절한 타이밍까지 기다리는 방식"`으로 안전성을 확보

- 왜 문제가 생길 수 있냐?
브라우저의 Event Loop는 아래와 같은 상황에서 DOM 상태를 변경처리

```

작업	                |       설명
setTimeout()	        |   macro task, 렌더링 이후 실행
Promise.then()	        |   micro task, 다음 tick에 실행
requestAnimationFrame()	|   렌더링 프레임 직전에 실행
React의 setState()	|  비동기 처리, Event Loop 내부에서 실제 DOM 변경
➡ 이 과정들은 Cypress 입장에서 "언제 변화가 끝날지 명확하지 않음"
```

- Cypress는 이걸 어떻게 처리할까?
    -   Cypress의 자동 대기 전략(DOM Query의 Retry-ability)
    -  ex) .get(), .contains(), .should() 등은 조건이 만족할 때까지 반복 실행 (기본 4초)

- Promise나 setTimeout을 기다리지 않음
→ Cypress는 JS 이벤트 큐의 상태 자체를 감지하지 않음
→ 대신 UI(DOM)가 변화했는지만 감지함

- 그래서 Cypress `DOM이 내가 원하는 상태로 바뀌는 것만 기다린다`는 전략을 씀
→ Event Loop가 완료됐는지는 관심 없음, 결과만 확인

흐름이 꼬일 수 있는 예시
위험한 예시 (Event Loop 완료 보장 못함)
```jsx
cy.get('button').click();

// 여기서 UI가 500ms 후에 바뀌는데
setTimeout(() => {
  document.body.append('완료');
}, 500);

// Cypress가 너무 빨리 아래를 실행할 수도 있음
cy.contains('완료').should('exist');
문제:
cy.contains('완료')는 setTimeout()으로 인해 아직 DOM에 append되지 않았을 수도 있음
→ 이 경우 Cypress는 4초 동안 반복 시도하다가 성공하거나 실패함
→ 우리가 Event Loop의 완료를 직접 기다릴 수 있는 수단은 Cypress에는 없음
```
해결 전략
```jsx
방법 1: should()나 contains()로 결과를 기다리게 하기
cy.get('button').click();

// 결과 DOM이 실제로 존재할 때까지 Cypress가 자동으로 기다림
cy.contains('완료').should('exist');
➡ 이렇게 하면 내부적으로 Cypress는 DOM 상태가 바뀔 때까지
cy.contains()를 계속 재시도하며,
실제로 변경이 일어난 뒤에만 should()가 통과됨

방법 2: 비동기 로직을 Cypress 체인 안으로 감싸기 (cy.wrap())

cy.wrap(null).then(() => {
  return new Promise(res => setTimeout(res, 500));
}).then(() => {
  document.body.append('완료');
});

cy.contains('완료').should('exist');
➡ Cypress는 .then() 안의 Promise를 인식하고, 비동기 처리가 완료될 때까지 기다림
→ 이후 체이닝이 보장된 순서로 실행됨
```