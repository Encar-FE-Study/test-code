# 단위 테스트
- 독립성을 가짐 : 다른 모듈과 별개로 테스트 가능한 대상
- 다른 외부 모듈을 사용한다면 이에 영향을 받지 않기 위해 mocking으로 내부 로직에 집중(외부영향 최소화)
- 장점: 범위가 작으므로 변경 사항에 대한 테스트 코드 수정과 에러 파악이 쉽다.
- 단점: 여러 모듈이 같이 조합되서 사용되는 경우에 대해서는 의미가 없다.

- ex) Input / useDebounce 등


# React-Testing-Library의 역할과 특징
- 역할: JSDOM에 리액트를 통해 생성된 노드를 그려주고, 브라우저 없이 테스트 코드 실행을 가능케함
- 테스트 러너는 아니므로 .test / .spec 형식의 파일을 찾으려면 jest나 vitest가 필요
- 사용 권장방식: Testing Library recommends finding elements by accessibillity handling
- 테스트 코드 작성시 Role에 의한 쿼리를 제일 추천한다(getByRole, queryByRole 등) -> 사용자가 실제로 사용하는 방식과 제일 유사하므로


# jsdom
```
역할: 노드에서 돔에 접근가능하게 해줌(이게 없다면 노드에서 DOM에 접근할 방법이 없다.)
특징: DOM API 제공
즉, 노드를 브라우저처럼 동작하는 가짜 환경을 만들어줌
```

# 테스트 패턴 : AAA패턴(Given-when-then과 비슷하므로 둘 중 무엇을 쓰더라도 무방)
- AAA : 준비 - 실행 - 검증 (이때 실행과 검증은 유저의 실제 사용방식에 대해 검증하는 것을 추천)
```


function UserProfile() {
  return (
    <>
      {/* 1. label 사용 */}
      <label htmlFor="name">이름</label>
      <input id="name" />
      
      {/*2. 버튼 */}
      <button>수정</button>
    </>
  );
}



import { render, screen } from '@testing-library/react';
import userEvent from '@testing-library/user-event';
import '@testing-library/jest-dom';

test('사용자 프로필 업데이트', async () => {
    // Arrange (준비)
    render(<UserProfile user={{ name: '승찬' }} />);
    const user = userEvent.setup();
  
    const nameInput = screen.getByRole('textbox', { name: '이름' });
    const editButton = screen.getByRole('button', { name: '수정' });

    // name의 찾는 순서
    <label> 요소의 텍스트 ->   <label htmlFor="name">이름</label><input id="name" />
    <button>의 경우 내부 텍스트 -> <button>수정</button>

  
    // Act (실행)
    await user.click(editButton);
    await user.type(nameInput, '엔카닷컴');
    
    // Assert (검증)
    expect(nameInput).toHaveValue('엔카닷컴');
});

```