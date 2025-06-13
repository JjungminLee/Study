https://ko.legacy.reactjs.org/docs/context.html

## 1. 리액트가 props를 움직이는 방법

### State 끌어올리기

- 두 컴포넌트를 조정하고 싶을 때 state를 그들의 공통 부모로 이동
- 그리고 공통 부모로부터 props를 통해 정보를 전달합니다.
- 마지막으로 이벤트 핸들러를 전달해 자식에서 부모의 state를 변경할 수 있도록 함.
- 컴포넌트를 (props로부터) “제어”할지 (state로부터) “비제어” 할지 고려하면 유용

## 2. State를 보존하고 초기화하기

- 각 컴포넌트는 독립된 state를 가짐
- 리액트는 UI 트리에서의 위치 통해 state가 어떤 컴포넌트에 속하는지 추적
  - UI 트리 : 컴포넌트 계층 구조
- 리렌더링마다 언제 state보존하고 또 state 초기화할지 컨트롤 가능.
-

## 3. State는 렌더트리의 위치에 연결

- React는 UI안에 있는 컴포넌트 구조로 렌더 트리를 만든다.
- 컴포넌트에 state를 줄 때 state가 컴포넌트 안에 살고 있다 생각할 수 있지만 사실 state는 React안에 있다.
- React가 컴포넌트를 제거할 때 그 state도 같이 제거한다.
  - 반대로 컴포넌트가 UI트리에서 그 자리에 렌더링하는 한 state는 유지
- React는 트리만 본다는 것을 기억!
- 리렌더링 할때 state를 유지하고 싶으면 트리구조가 같아야한다
- [예시1]

```
function App() {
  const [show, setShow] = React.useState(true);

  return (
    <div>
      <button onClick={() => setShow(!show)}>Toggle</button>
      {show ? <ComponentA /> : <ComponentB />}
    </div>
  );
}

function ComponentA() {
  return <h1>Component A</h1>;
}

function ComponentB() {
  return <h1>Component B</h1>;
}

```

- 트리구조를 간략하게 표현하자면

```
<div>
  <button />
  <ComponentA /> (or <ComponentB />)
</div>

```

- [예시2]

```
function App() {
  const [show, setShow] = React.useState(true);

  return (
    <div>
      <button onClick={() => setShow(!show)}>Toggle</button>
      {show ? <ComponentAWrapper /> : <ComponentBWrapper />}
    </div>
  );
}

function ComponentAWrapper() {
  return <ComponentA />;
}

function ComponentBWrapper() {
  return <ComponentB />;
}

function ComponentA() {
  return <h1>Component A</h1>;
}

function ComponentB() {
  return <h1>Component B</h1>;
}


```

- 트리구조로 간략하게 표현하자면

```
<div>
  <button />
  <ComponentAWrapper>
    <ComponentA />
  </ComponentAWrapper>
</div>

```

- ComponentAWrapper 가 오냐 ComponentBWrapper가 오냐에 따라 감싸지는 새로운 레벨이 달라지기 때문에 트리구조가 이전과 달라졌다 판단하여 초기화

## 4. Key를 통해 상태관리하자.

- 리스트 렌더링이나 동적으로 생성되는 컴포넌트의 상태 유지관리시 유리

```
function App() {
  const [components, setComponents] = React.useState([0]);

  const addComponent = () => {
    setComponents([...components, components.length]);
  };

  return (
    <div>
      <button onClick={addComponent}>Add Component</button>
      {components.map((id) => (
        <DynamicComponent key={id} id={id} />
      ))}
    </div>
  );
}

function DynamicComponent({ id }) {
  const [value, setValue] = React.useState("");

  return (
    <div>
      <input
        value={value}
        onChange={(e) => setValue(e.target.value)}
        placeholder={`Component ${id}`}
      />
    </div>
  );
}

```

- 대부분 map을 쓰거나 리스트 렌더링 할때 key를 쓰라고 많이 들었을텐데 그 말이다!

## 5. State 로직을 Reducer로 작성하기

- state 관리하는 이벤트 핸들러가 여러개이면 -> reducer라는 단일함수로 통합한다.
- dispatch
  - 액션을 상태관리 시스템에 전달하기 위한 함수
- 과정
  1.  state를 설정하는 것에서 action을 dispatch 함수로 전달하는 것으로 **바꾸기**.
  2.  reducer 함수 **작성하기**.
  3.  컴포넌트에서 reducer **사용하기**.

```
function handleAddTask(text) {

dispatch({

type: 'added',

id: nextId++,

text: text,

});

}



function handleChangeTask(task) {

dispatch({

type: 'changed',

task: task

});

}



function handleDeleteTask(taskId) {

dispatch({

type: 'deleted',

id: taskId

});

}
```

- reducer함수를 어떻게 작성할까?

```
function tasksReducer(tasks, action) {

switch (action.type) {

case 'added': {

return [...tasks, {

id: action.id,

text: action.text,

done: false

}];

}

case 'changed': {

return tasks.map(t => {

if (t.id === action.task.id) {

return action.task;

} else {

return t;

}

});

}

case 'deleted': {

return tasks.filter(t => t.id !== action.id);

}

default: {

throw Error('Unknown action: ' + action.type);

}

}

}
```

- 들어온 액션의 타입에 따라 다른 행동을 처리해줄 수 있다!
  - 핸들러를 여러개 쓰는것보다 가독성이 좋아진다.
  - reducer는 switch-case를 사용하는것이 원칙

## 6. 컴포넌트에서 reducer 사용하는 방법

```
import { useReducer } from 'react';
import AddTask from './AddTask.js';
import TaskList from './TaskList.js';

export default function TaskApp() {
  const [tasks, dispatch] = useReducer(
    tasksReducer,
    initialTasks
  );

  function handleAddTask(text) {
    dispatch({
      type: 'added',
      id: nextId++,
      text: text,
    });
  }

  function handleChangeTask(task) {
    dispatch({
      type: 'changed',
      task: task
    });
  }

  function handleDeleteTask(taskId) {
    dispatch({
      type: 'deleted',
      id: taskId
    });
  }

  return (
    <>
      <h1>Prague itinerary</h1>
      <AddTask
        onAddTask={handleAddTask}
      />
      <TaskList
        tasks={tasks}
        onChangeTask={handleChangeTask}
        onDeleteTask={handleDeleteTask}
      />
    </>
  );
}

function tasksReducer(tasks, action) {
  switch (action.type) {
    case 'added': {
      return [...tasks, {
        id: action.id,
        text: action.text,
        done: false
      }];
    }
    case 'changed': {
      return tasks.map(t => {
        if (t.id === action.task.id) {
          return action.task;
        } else {
          return t;
        }
      });
    }
    case 'deleted': {
      return tasks.filter(t => t.id !== action.id);
    }
    default: {
      throw Error('Unknown action: ' + action.type);
    }
  }
}

let nextId = 3;
const initialTasks = [
  { id: 0, text: 'Visit Kafka Museum', done: true },
  { id: 1, text: 'Watch a puppet show', done: false },
  { id: 2, text: 'Lennon Wall pic', done: false }
];

```

## 7. Props전달하기의 문제점

- prop을 트리를 통해 깊이 전달하거나 많은 컴포넌트에서 같은 prop이 필요한 경우 장황하고 불편할 수 있음
- 높게 끌어올리면 Prop Drilling이 초래함
- Props를 전달,전달,전달하는 대신 순간이동하려면 -> React의 Context를 사용하자!

## 8. Context

- 3가지의 단계로 구성
  1.  context 생성
  2.  데이터가 필요한 컴포넌트에서 context사용
  3.  데이터를 지정하는 컴포넌트에서 context 제공
- 1단계 : context를 생성해보자!

```
// LevelContext.js
import { createContext } from 'react';

export const LevelContext = createContext(1);
```

- 2단계 : context를 사용하자

```
import { useContext } from 'react';

import { LevelContext } from './LevelContext.js';

export default function Heading({ children }) {

const level = useContext(LevelContext);

// ...

}

```

- 3단계 : Context 제공하기

```
import { LevelContext } from './LevelContext.js';



export default function Section({ level, children }) {

return (

<section className="section">

<LevelContext.Provider value={level}>

{children}

</LevelContext.Provider>

</section>

);

}
```

- Level Context를 자식에게 제공하기 위해 Context Provider로 감싸준다!
- React에게 Section 내의 어떤 컴포넌트가 LevelContext를 요구하면 level을 주라고 알려줌.
  - 컴포넌트는 그 위에 있는 UI트리에서 가장 가까운 <LevelContext.Provider>를 사용
