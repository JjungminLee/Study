## 개요

```jsx
decorators: [
  (Story) => {
    const originalSetTimeout = window.setTimeout;

    window.setTimeout = ((handler: () => void, timeout?: number) => {
      return originalSetTimeout(() => {}, timeout);
    }) as typeof window.setTimeout;

    return <Story />;
  }
]

```

- Toast Message를 띄우는 스토리북에서 이런 코드를 봤다.
- 토스트 메세지는 3초 이후에 꺼지기 때문에 timeout을 사용해서 토스트메세지가 계속 렌더링하게 할 용도 인것 같았다.

## 일반적인 setTimeout의 동작방식

[자바스크립트 setTimeout()로 타이머 설정하기](https://www.freecodecamp.org/korean/news/how-to-set-a-timer-in-javascript/)

### 함수의 기본 동작

```jsx
setTimeout(() => {
  console.log("hello");
}, 1000);
```

- 일정 시간이 지난 후 함수가 실행된다.
  - 비동기 함수이다.
- 이 코드에서는 1000Ms 이후에 콜백 함수를 실행한다.
- 내부적으로 브라우저의 타이머 API를 사용해 이벤트 루프 큐에 해당 콜백을 등록한다.
- 반환값은 timeID이고 이 ID는 clearTImeout(id)로 타이머 취소 시 사용된다.

## 현재 코드의 동작방식

- originSetTiemout에 기존의 진짜 window.setTimeout(토스트 메세지에서 지정해둔 timeout 시간)이 저장된다.
- 그 다음 window.setTimeout을 새로운 함수로 덮어 씌운다.

```jsx
(handler: () => void, timeout?: number) => {
  return originalSetTimeout(() => {}, timeout);
};
```

- 실제로 넘겨받은 handler는 실행되지 않고 빈 함수 ()⇒{} 가 타이머에 등록된다.
- 그래서 타임아웃하고 꺼질때 빈함수를 렌더링하는데
  - 사실상 사라져야하지만 그 동작 자체가 실행되지 않고 빈 함수만 등록이 되어 아무것도 안한다.

### 좀 더 쉽게 설명하자면

```jsx
setTimeout(() => {
  // toast UI 제거
  hideToast();
}, 3000); // 3초 뒤 사라짐

window.setTimeout = (handler, timeout) => {
  return originalSetTimeout(() => {}, timeout);
};
```

- 3초 후에 토스트가 제거되어야하지만
- hideToast()는 실행되지않는다.
- setTimeout은 호출되지만 내부적으로 등록된 함수가 빈함수이기 때문이다

```jsx
originalSetTimeout(() => {}, timeout);
setTimeout(()=>{},timeout); -> 이렇게 해도 무방하다!
```

### 스토리북이라는 특수성

[[React]리액트 컴포넌트에서 setTimeout 사용하기](https://velog.io/@jaewoneee/%EB%A6%AC%EC%95%A1%ED%8A%B8-%EC%BB%B4%ED%8F%AC%EB%84%8C%ED%8A%B8%EC%97%90%EC%84%9C-setTimeout-%EC%82%AC%EC%9A%A9%ED%95%98%EA%B8%B0#3a-id3a-usetimeout-%ED%9B%85%EC%9D%84-%EC%9D%B4%EC%9A%A9%ED%95%9C-%EC%84%A0%EC%96%B8%EC%A0%81-timeouts)

- 스토리북은 상태변화나 사용자 이벤트에 의한 리렌더링의 거의 없다
  - 그래서 한번 렌더링되고 끝이다
- 하지만 일반 리액트 코드에서는 setTimeout이 리렌더링 될때마다 다시 실행될수 있다.
- 안전하게 쓰려면 clearTImeout을 통해 지우고 사용해야한다.

```jsx
const timerRef = (useRef < NodeJS.Timeout) | (null > null);

useEffect(() => {
  if (timerRef.current) clearTimeout(timerRef.current);

  timerRef.current = setTimeout(() => {
    console.log("timeout 실행");
  }, 1000);
}, [count]);
```

## setInterval

- setTimeout이 지정된 시간이 경과한 후에 한번만 특정함수를 실행하는 반면 setInterval은 일정시간 간격으로 지정된 함수를 반복실행한다
- 보통 리액트에서는 useRef를 사용해 interval을 기억하도록 만든다.
  - 그냥 setInterval만 사용한다면?
    - 상태가 변화할때마다 리렌더링 되기 때문에 setInterval이 여러번 실행되어 에러가 난다.
- useInterval을 사용한다!
  - useRef를 사용해서 함수 핸드러를 기억해 리렌더링을 방지한다!
  - useRef사용해 최신 콜백을 저장해두고 interval을 초기 1번만 등록한 뒤 계속 Ref안의 최신 콜백을 실행한다

[React에서 setInterval 현명하게 사용하기(feat. useInterval)](https://mingule.tistory.com/65)

```jsx
import { useEffect, useRef } from "react";

const useInterval = (callback, delay) => {
  const savedCallback = useRef(null);

  useEffect(() => {
    savedCallback.current = callback;
  }, [callback]);

  useEffect(() => {
    const executeCallback = () => {
      savedCallback.current();
    };

    const timerId = setInterval(executeCallback, delay);

    return () => clearInterval(timerId);
  }, []);
};

export default useInterval;
```
