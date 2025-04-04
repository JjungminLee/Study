## 왜 리액트를 쓰는거지?

- 업데이트 때문
- 즉각적인 업데이트를 대규모로 하긴 어려움
  - 성능
    - 업데이트 하면 브라우저가 페이지의 레이아웃을 다시 계산함(reflow)
      - 성능병목현상
  - 신뢰성
    - 상태를 여러 곳에서 추적해 모든 곳에서 일관되게 유지
  - 보안
    - XSS, CSRF악용 방지
- 디바운싱
  - 이벤트가 마지막으로 발생한 후 설정된 시간이 경과하기 전까지 함수의 실행을 지연시킴
- throttling
  - 함수가 설정된 시간 간격내에 한번만 실행되도록 제한해 빈번하게 실행되지 않게함

## 리액트의 장점

- 상호작용이 필요한 버튼을 아주 많이 반들고 매우 간결하고 효율적인 방식으로 이벤트에 반응.
- 이런작업을 테스트하고 재연하는 것이 가능.
- 선언적이고 성능이 뛰어나며 예측하고 신뢰할수 있음
- 사용자 인터페이스 상태를 완전히 제어하고 그 상태를 기반으로 렌더링

## Vanila JS로만 만든다면?

- form을 submit하는 로직이 있다했을때
  - onsubmit속성은 다른 클라이언트에 의해 쉽게 변형 가능
- 예측 불가
  - id가 동일한 앨리먼트가 여러개 있다면
- 비효율적
  - 화면에 렌더링을 너무 많이 한다면?(DOM렌더링 비용은 매우 비쌈)

### jQuery

- 전역적으로 수정가능
- DOM코드를 직접 조작함 -> 디버깅하기 어려워짐

### backbone

- 브라우저와 자바스크립트 간의 상태불일치, 코드 재사용, 테스트 가능성등을 처음으로 해결하고자 함
- MVC패턴을 적용
  - 복잡한 상호작용 및 상태관리
    - 사용자와 상호작용이 많으면 상태 변경과 UI의 다양한 부분에 미치는 영향 관리하기 어려움
  - 양방향 데이터 바인딩
    - 뷰가 모델에 동기화 되지 않거나, 모델이 뷰와 동기화되지 않을 수 있다
    - 리액트는 단방향 데이터 흐름을 가진다
  - 강한 결합

### Knockout

- 반응형 자바스크립트 라이브러리
- 관찰 가능한 방식으로 상태변화에따라 값을 업데이트
  - 이런 반응성을 현대적으로 구현한걸 시그널이라 부르기도 한다!
  - 뷰,스벨트,앵귤러에서 사용
- observable
  - 데이터의 출처
  - 모델
- binding
  - 해당 데이터를 소비하고 렌더링하는 사용자 인터페이스
  - 뷰
- MVVM사용
- MVC와 MVVM의 차이는 결합과 바인딩

#### React가 MVC와 MVVM보다 개선된 점

React는 MVC(Model-View-Controller)나 MVVM(Model-View-ViewModel)과 같은 기존 아키텍처보다 더 유연하고 효과적인 방법을 제공한다.

| 비교 항목       | MVC/MVVM                                                       | React                                                                      |
| --------------- | -------------------------------------------------------------- | -------------------------------------------------------------------------- |
| **구조**        | 명확한 역할 분리 (Model, View, Controller/ViewModel)           | 단방향 데이터 흐름과 컴포넌트 기반 UI                                      |
| **데이터 흐름** | 양방향 데이터 바인딩 (MVVM)으로 인해 예측하기 어려운 상태 변화 | 단방향 데이터 흐름으로 예측 가능하고 디버깅이 쉬움                         |
| **UI 업데이트** | DOM을 직접 조작하므로 성능 저하 발생                           | Virtual DOM을 사용하여 효율적인 업데이트                                   |
| **재사용성**    | 컨트롤러(ViewModel)는 View에 종속적이며, 코드 재사용이 어려움  | 컴포넌트 기반 구조로 재사용성이 뛰어남                                     |
| **확장성**      | 대규모 프로젝트에서는 복잡한 데이터 흐름 관리가 어려움         | React의 상태 관리 라이브러리(Redux, Zustand 등)와 조합하여 확장성이 뛰어남 |

React는 특히 단방향 데이터 흐름과 컴포넌트 기반 UI를 통해 복잡한 상태 관리를 쉽게 하고, 성능을 향상시킨다.

### Angular

- 양방향 데이터 바인딩
  - 모델이 변경되면 뷰도 변경되게
- 트레이드 오푸
  - 성능
    - 변경감지를 위한 핵심기능인 다이제스트 주기는 대규모 애플리케이션에서 업데이트 지연과 사용자 인터페이스 응답성 저하를 발생
  - 복잡성
    - 지시자, 컨트롤러,서비스, 의존성 주입 등 새로운 개념을 도입했으나 복잡함
  - 복잡한 탬플릿 문법
    - html안에 비즈니스 로직이 뒤섞임
  - 타입 안정성 부재
    - 타입스크립트가 작동 안함!

## 리액트의 등장

- 컴포넌트 기반
- 단방향 데이터 흐름
  - 개발자가 애플리케이션을 제어하기에 용이
  - 시간에 따라 데이터가 어떻게 변화하는지 이해하기 쉬움
  - 가상 DOM을 통해 DOM조작 최소화
- 핵심가치
  - 선언적코드
    - 우리가 보고자 하는 것을 코드로 표현
    - 어떻게 할지는 리액트가 알아서
  - 가상 DOM
    - 개발자가 실제 DOM을 조작하지 않고도 가상 DOM을 통해 ui업데이트 가능
    - 실제 DOM트리와 일치하도록 가상 DOM을 생성하고 업데이트한다.
    - 가상 DOM에서 발생한 모든 변경사항은 재조정이라는 과정을 통해 실제 DOM에 반영
      - 재조정 : 새로운 가상 DOM과 이전 가상 DOM을 비교하는 과정
  - 불변 상태
    - 애플리케이션 상태를 불변하는 값의 집합으로 기술
    - 각각의 상태 업데이트는 새로운 독립된 스냅샷과 메모리 참조로 취급
    - 불변성을 강제하기 때문에 UI컴포넌트가 특정시점의 특정 상태를 반영
    - 상태가 변경되면 원래 있던 상태를 직접 변경하는게 아니라 새로운 상태를 표현하는 새로운 객체를 반환

## 플럭스 아키텍쳐

![[Pasted image 20250221060301.png]]

- 클라이언트 측 웹 애플리케이션 구축을 위한 아키텍쳐 디자인 패턴
- 단방향 데이터 흐름을 강조해 데이터의 흐름을 더욱 예측가능하게 만든다

### 액션

- 새로운 데이터와 액션의 종류를 식별하는 속성을 포함하는 단순한 객체

### 디스패처

- 액션을 받아서 애플리케이션에 등록된 스토어에 보낸다
- 모든 스토어는 디스패처에 스토어 자신과 콜백을 등록해두는데 콜백 목록을 관리하는 것도 디스패처
- 액션을 디스패칭하면 등록된 모든 콜백으로 해당 액션 전송

### 스토어

- 애플리케이션 상태와 로직 포함
- 다수 객체의 상태를 관리
- 스토어 자신을 디스패처에 등록
- 스토어의 상태가 업데이트 되면 변경 이벤트를 발생시켜 뷰에 변경 사항을 알림

### 뷰

- 리액트의 컴포넌트
- 스토어에서 변경 이벤트를 받으며 의존하는 데이터가 변경되면 스스로 업데이트

### 플럭스 아키텍쳐의 장점

- 단일정보 출처
  - 단일정보 출처가 스토어에 저장됨
  - 예측하기 쉬움
- 테스트 가능성
  - 액션,디스패처,스토어,뷰로 관심사를 분리하면 각 부분을 독립적으로 단위테스트 가능
- 관심사 분리
  - 더 모듈화되고 유지보수가 쉬우며 파악하기 쉬워짐
