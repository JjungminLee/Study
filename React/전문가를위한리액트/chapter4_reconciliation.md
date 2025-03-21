## 재조정

- DOM에 대한 가장 효율적인 일괄 업데이트를 계산하기 위해
  - 리액트는 현재 가상 DOM트리를 복제해 새로운 가상 DOM트리 생성하고 업데이트된 값 적용

```
const handleClick=()=>{

	setCount((prevCount)=>prevCount+1);
	setCount((prevCount)=>prevCount+1);
	setCount((prevCount)=>prevCount+1);
}
```

- 실제로는 업데이트가 한번만 되고 0->3으로 업데이트 됨

## 기존 기술 - stack 재조정자

- 이전 리액트는 랜더링에 스택 데이터 구조 사용
- 작업을 일시중지하거나 연기하지 않고 순차적으로 변경사하을 렌더링
- 계산비용이 비싼 컴포넌트가 렌더링을 막으면 사용자 입력이 눈에띄게 버벅거림
  - 사용자 입력처럼 우선 순위가 높은 렌더링 작업이 끼어들때는 현재 진행중인 렌더링 작업을 막을 수 있어야한다
  - 그러려면 리액트가 특정 종류의 렌더링 연산을 다른 종류의 렌더링 보다 우선시해야한다!
- 스택 재조정자는 업데이트의 우선순위를 설정하지 않는다!
- 리액트 애플리케이션에서는 가상트리에 대한 업데이트는 중요도가 각기 다를 수 있다!
  - 스택 재조정자는 덜 중요한 업데이트가 더 중요한 업데이트를 차단할 수 있다

## 현재 기술 - fiber 재조정자

- fiber는 리액트 엘리먼트에서 생성된다
- 파이버는 상태를 저장하고 수명이 긴 반면 리액트 엘리먼트는 임시적이고 상태가 없다

### 파이버 데이터 구조

- 파이버 재조정자는 **업데이트의 우선순위를 정하고** 이에따라 동시 실행을 가능하게 해서 리액트 애플리케이션의 성능과 응답성을 향상시킴
- 파이버 데이터 구조는 리액트 애플리케이션에서 **컴포넌트 인스턴스와 그 상태를 표현한다.**
  - 변경 가능한 인스턴스로 설계되었으며 조정 과정에서 필요에따라 업데이트되고 재배치됨

### 파이버 재조정 흐름

- 파이버 재조정에는 현재 파이버 트리와 다음 파이버 트리를 비교해 어느 노드를 업데이트, 추가, 제거할지 파악하는 로직이 포함됨
- 조정 과정 중에 파이버 재조정자는 가상 DOM의 각 리액트 엘리먼트에 대해 파이버 노드를 생성
  - 이 작업은 createFiberFromTypeAndProps함수가 수행
    - 타입과 프롭은 리액트 엘리먼트로도 부를 수 있다!
  - 앨리먼트에서 파생된 파이버를 반환
  - 파이버 노드가 생성되면 파이버 재조정자는 작업 루프를 사용해 사용자 인터페이스 업데이트
    - 업데이트가 필요한 경우 각 파이버 노드를 '더티'라고 표현
    - 끝에 도달하면 다시 반대로 순회하면서 브라우저의 DOM트리와는 분리된 새 DOM트리를 메모리에 생성
    - 새 DOM트리는 이후 화면에 반영
      - 이걸 flushed

### 더블 버퍼링

- 파이버 아키텍쳐가 착안한 개념
  - 첫번째 버퍼가 초기 이미지나 프레임으로 채워짐
  - 첫번째 버퍼가 표시되는 동안 두번째 버퍼가 새 데이터나 이미지로 업데이트
  - 두번째 버퍼가 준비되면 첫번째 버퍼로 전환되어 화면에 표시
  - 첫번째와 두번째 버퍼가 일정한 간격으로 전환되어 최종 이미지나 동영상을 표시하는 프로세스가 계속됨
- 업데이트 발생시 -> 현재 파이버 트리가 포크되어 주어진 사용자 인터페이스의 새로운 상태 반영하도록 업데이트 됨 -> 이를 렌더링이라고 함
- 현재 트리를 대체할 트리가 준비되고 사용자가 기대하는 상태를 정확하게 반영하면 더블 버퍼링에서 비디오 버퍼 교체하는 것처럼 현재 파이버 트리와 교체됨 -> 이를 재조정의 커밋

### 파이버 재조정자가 화면에 표시되지 않는 작업용 트리 사용시 장점

- 실제 DOM에 불필요한 업데이트 줄일 수 있어 성능 개선하고 깜빡임 줄임
- 화면 밖에서 UI의 새 상태 개선하고 우선순위가 더 높은 새로운 업데이트 필요시 이를 버림
- 재조정은 화면 밖에서 이루어짐
  - 사용자가 현재 보고 있는 내용을 망치지 않고 일시중지 했다 다시 시작가능

### 파이버 재조정

- 렌더링 단계와 커밋단계로 이루어짐
  - 두 단계 접근 덕분에 렌더링 작업 수행 후 -> DOM에 커밋해서 사용자에게 보여주기 전에 언제든 폐기 가능
    - 렌더링 중단이 가능하다!
    - 렌더링이 중단이 가능한듯 보이는 이유?
      - 리액트 스케쥴러가 실행을 5밀리초마다 메인 스레드로 돌려줌

### 렌더링 단계

- 현재 트리에서 상태 변경 이벤트가 발생하면 실시
- 각 파이버를 재귀적, 단계적으로 순회

#### beginWork(작업시작)

    - 작업용 트리에 있는 파이버 노드의 업데이트 필요여부를 나타내는 플래그 설정
    - 여러 플래그를 설정하고 계속해서 다음 파이버 노드로 이동하여 트리의 맨 아래에 도달할때까지 동일한 작업 수행
    - 작업 완료시 파이버노드에서 completeWork호출하여 다시 거슬러 올라가며 순회
    - beginWork의 시그니처
    	- current
    		- 업데이트 중인 작업용 노드
    	- workInProgress
    		- 작업용 트리에서 업데이트 중인 파이버 노드
    		- 더티로 표시된채 반환
    	- renderLanes
    		- 리액트가 업데이트 우선순위를 더 잘 정하고 업데이트 프로세스를 더 효율적으로 만들 수 있음
    		- 우선순위가 높은 레인에 할당된 업데이트가 먼저 처리됨
    		- 동시성 관리에도 용이
    			- 리액트는 타임 슬라이싱 기능을 이용해 실행 시간이 긴 업데이트를 더 작고 관리하기 쉬운 덩어리로 분할
    				- 이때  renderLanes는 리액트가 어떤 업데이트를 먼저 처리할지, 미룰지 결정

#### completeWork

    	- 작업용 파이버 노드에 업데이트 적용하고 애플리케이션의 업데이트된 상태를 나타내는 실제 DOM트리 새롭게 생성
    	- 호스트 환경에 커밋할 새 트리 구성하는 역할

### 커밋단계

- 가상 DOM트리에 적용된 변경사항을 실제 DOM트리에 반영
- 커밋 단계의 효과
  - 배치 효과
  - 업데이트 효과
  - 삭제효과
  - 레이아웃 효과
- 패시브 효과
  - 브라우저의 페인트 가능 시점 후에 실행되도록 예약된 효과
  - useEffect를 통해 관리
  - 데이터 요청처럼 페이지의 초기 렌더링에 중요하지 않은 작업 수행하는데 용이

#### 변형단계

- 커밋단계의 첫부분
- 가상 DOM에 적용된 변경 사항을 실제 DOM에 반영
- 적용할 업데이트를 식별 -> commitMutationEffects라는 특수함수 호출
- 렌더링 단계에서 작업용 트리의 파이버 노드에 적용된 업데이트를 실제 DOM에 반영

#### 레이아웃 단계

- DOM에서 업데이트 된 노드의 새 레이아웃을 계산한다.
- commitLayoutEffects라는 특수함수 호출

### fiberRootNode

- 리액트는 현재 트리나 작업용 트리 중 하나에 파이버 루트 노드를 둔다.
  - 재조정과정의 커밋단계 관리하는 핵심 데이터 구조
