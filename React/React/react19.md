## useTransition이 비동기 함수를 지원할 수 있다

- 추가적인 상태관리 없이도 startTransition 함수 내에서 비동기 함수를 직접 실행가능.
- isPending의 상태가 자동으로 관리

```
import React, { useState, useTransition } from 'react';

function UpdateName() {
  const [name, setName] = useState('');
  const [error, setError] = useState(null);
  const [isPending, startTransition] = useTransition();

  const handleSubmit = () => {
    startTransition(async () => {
      try {
        await updateName(name); // 비동기 함수 호출
        // 성공 시 추가 작업 수행
      } catch (err) {
        setError(err);
      }
    });
  };

  return (
    <div>
      <input
        value={name}
        onChange={(event) => setName(event.target.value)}
      />
      <button onClick={handleSubmit} disabled={isPending}>
        Update
      </button>
      {error && <p>{error.message}</p>}
    </div>
  );
}

```

- 어디서 쓸수 있을까?
  - 사용자가 검색어 입력시, 입력 필드가 즉각 업데이트 되고, 필터링 작업은 비동기로 처리해야할때
  - 폼 제출, 데이터 저장 또는 서버 작업 중 상태 표시하고 UI 업데이트 해야할때
- 언제 쓰면 비효율적일까?
  - 상태 업데이트가 즉각적이고 부하가 크지 않을때
  - UI의 복잡성이 낮을때

## useOptimistic

- 낙관적 업데이트를 처리하기 위함
- 백엔드 요청 결과를 기다리지 않고, 사용자 경험을 원할하게 만들기 위해 즉시 UI업데이트 하는것을 말한다.
- 언제 사용될까?
  - 댓글 추가, 목록 필터링

```
import React, { useOptimistic } from "react";

function App() {
  const [comments, updateComments] = useOptimistic(
    [], // 초기 상태
    (currentComments, newComment) => {
      // 새로운 댓글을 추가하는 낙관적 업데이트
      return [...currentComments, newComment];
    }
  );

  const handleAddComment = async () => {
    const newComment = { id: Date.now(), text: "Optimistic Comment" };

    // UI 즉시 업데이트
    updateComments(newComment);

    // 실제 서버 요청
    await fetch("/api/comments", {
      method: "POST",
      body: JSON.stringify(newComment),
    });
  };

  return (
    <div>
      <button onClick={handleAddComment}>Add Comment</button>
      <ul>
        {comments.map((comment) => (
          <li key={comment.id}>{comment.text}</li>
        ))}
      </ul>
    </div>
  );
}

```

## useFormStatus

- 폼의 상태를 쉽게 관리하고 추적할 수 있게 해준다.
  - 폼이 제출중인지 추적하여 버튼 비활성화, 로딩상태 표시등을 해준다.
  - 유효성 검사는 하지 않는다.

```
import React from 'react';
import { useFormStatus } from 'react';

function SimpleForm() {
  const { pending } = useFormStatus(); // 제출 상태 추적

  return (
    <form action="/submit" method="POST">
      <input type="text" name="username" placeholder="Username" />
      <button type="submit" disabled={pending}>
        {pending ? 'Submitting...' : 'Submit'}
      </button>
    </form>
  );
}


```

## ref를 일반 prop으로 전달할 수 있다!

- forwardRef를 사용하지 않고도 ref를 쉽게 사용할 수 있다!
  - ref를 왜 쓸까?
    - 보통 상태와 props를 통해 UI를 제어하지만 실제 DOM요소에 접근해야하는 경우
    - 포커스 설정
    - HTML5요소 조작 (Video,Canvas)
    - 너비,높이 계산
- [기존방식]

```
import React, { forwardRef } from 'react';

const MyComponent = forwardRef((props, ref) => {
  return <div ref={ref}>{props.children}</div>;
});

export default function App() {
  const myRef = React.createRef();

  return <MyComponent ref={myRef}>Hello</MyComponent>;
}

```

- [React19]
  - ref를 일반 prop으로 전달

```
function MyComponent({ ref, children }) {
  return <div ref={ref}>{children}</div>;
}

export default function App() {
  const myRef = React.createRef();

  return <MyComponent ref={myRef}>Hello</MyComponent>;
}

```
