## 개요

- 기존 코드에서 switch-case문으로 UI를 렌더링하는 코드를 봄
- 4개정도의 케이스가 있었는데 뭔가 JS스럽게 짜보고 싶다는 생각을 하게 됨
  - 리액트의 철학은 UI 렌더링을 선언적으로 하는 것이기 떄문

```jsx
const renderTableData = (
    key: (typeof HEADER_INFOS)[number]["key"],
    item: GetClientModel["client"][number],
  ): jsx.JSX.Element => {
    switch (key) {
      case "GOOGLE: {
        const clientType = CLIENT_TYPE_KEY_PAIR[item[key]];

        return (
          <span>
            {clientType
              ? (t(
                  findLookupTableLabel(
                    CLIENT_TYPE_RADIOS,
                    clientType,
                  ) as UserLanguages,
                ) ?? "-")
              : "-"}
          </span>
        );
      }

```

### Solution1 : 객체매핑을 사용해보자!

- 기존 코드에서 ReactNode때문에 Switch-case를 썼던것이였음

```jsx
type renderKeyType = (typeof VEHICLE_TABLE_HEADER_INFOS)[number]["key"];
  type renderItemType = GetVehiclesClientModel["vehicles"][number];

  const renderers: Partial<
    Record<renderKeyType, (item: renderItemType) => jsx.JSX.Element>
  > = {
    vehicleType: (item) => {
      const vehicleType = VEHICLE_TYPE_KEY_PAIR[item.vehicleType];
      return (
        <span>
          {vehicleType
            ? (t(
                findLookupTableLabel(
                  VEHICLE_TYPE_RADIOS,
                  vehicleType,
                ) as MembershipAdminLanguages,
              ) ?? "-")
            : "-"}
        </span>
      );
    },
    fuelType: (item) => {
      const fuelType = VEHICLE_FUEL_TYPE_KEY_PAIR[item.fuelType];

      return (
        <span>
          {fuelType
            ? (t(
                findLookupTableLabel(
                  VEHICLE_FUEL_TYPE_DROPDOWNS,
                  fuelType,
                ) as MembershipAdminLanguages,
              ) ?? "-")
            : "-"}
        </span>
      );
    },
    isPrivate: (item) => {
      const ownership = VEHICLE_OWNERSHIP_KEY_PAIR[item.isPrivate];

      return (
        <span>
          {ownership
            ? (t(
                findLookupTableLabel(
                  VEHICLE_OWNERSHIP_RADIOS,
                  ownership,
                ) as MembershipAdminLanguages,
              ) ?? "-")
            : "-"}
        </span>
      );
    },
    created: (item) => {
      return item.created ? (
        <time>{formatICTDateTime(item.created)}</time>
      ) : (
        <span>-</span>
      );
    },
  };
```

### Solution2 : Multi함수

https://www.zigae.com/avoid-switch/

- 패턴 매칭으로 생각해볼수 있음
- 보통 정규표현식 쓸 때 사용하는 방식
  - UI 렌더링하는 코드 맥락에 맞지 않음
  - 비효율적이다!

## 좀 더 가독성을 높이고 React스럽게 만들어보자!

### DRY원칙을 지키며 중복코드를 제거 하자

- `vehicleType`, `fuelType`, `isPrivate` 에서 똑같은 함수를 써서 렌더링 로직을 탐
- 좀 더 추상화해서 중복된 코드를 고쳐보자!
- 라벨 렌더링, date렌더링 따로!
- 어떻게 적용했을까?

```jsx
  const renderLabel = <T extends { key: string | number; label: Languages }>(
    list: ReadonlyArray<T>,
    key: T["key"],
  ) => {
    return (
      <span>
        {key
          ? (t(findLookupTableLabel(list, key) as MembershipAdminLanguages) ??
            "-")
          : "-"}
      </span>
    );
  };

  const renderDate = (date: string) => {
    return date ? <time>{formatICTDateTime(date)}</time> : <span>-</span>;
  };

```

### 고차함수를 사용해 함수형 프로그래밍의 이점을 살리자

[📚 JavaScript 배열 고차 함수 총정리](https://inpa.tistory.com/entry/JS-%F0%9F%93%9A-%EB%B0%B0%EC%97%B4-%EA%B3%A0%EC%B0%A8%ED%95%A8%EC%88%98-%EC%B4%9D%EC%A0%95%EB%A6%AC-%F0%9F%92%AF-mapfilterfindreducesortsomeevery)

- 궁금해지는 부분
  - 함수형 프로그래밍과 선언형 프로그래밍의 차이?
    - 한줄정리 : 선언형 프로그래밍은 “무엇”을 할지 설명하는 방식이며 함수형 프로그래밍은 이를 실현하는 방식
  - 명령형 프로그래밍 vs 선언형 프로그래밍
    - 명령형은 프로그램이 어떻게 진행되어야 하는지 순서를 명시
    - 선언형은 무엇을 하는지 설명, 제어 흐름을 명시적으로 지정하지 않음
- 개념정리가 끝났으니 고차함수를 알아보자!
  - 고차함수는 함수형 프로그래밍의 핵심
  - 함수를 파라미터로 전달받거나 연산의 결과로 반환해주는 메서드
    - 예시로는 .find(), .filter(), .reduce() 등이 있다
      - reduce()의 경우, foreach()와 map()의 부모격
- 어떻게 적용했을까?

```jsx
const renderers: Partial<Record<RenderKeyType, RenderFunction>> = {
  vehicleType: (item) =>
    renderLabel(VEHICLE_TYPE_RADIOS, VEHICLE_TYPE_KEY_PAIR[item.vehicleType]),
  fuelType: (item) =>
    renderLabel(
      VEHICLE_FUEL_TYPE_DROPDOWNS,
      VEHICLE_FUEL_TYPE_KEY_PAIR[item.fuelType]
    ),
  isPrivate: (item) =>
    renderLabel(
      VEHICLE_OWNERSHIP_RADIOS,
      VEHICLE_OWNERSHIP_KEY_PAIR[item.isPrivate]
    ),
  created: (item) => renderDate(item.created),
};
```

### 전략패턴

[💠 전략(Strategy) 패턴 - 완벽 마스터하기](https://inpa.tistory.com/entry/GOF-%F0%9F%92%A0-%EC%A0%84%EB%9E%B5Strategy-%ED%8C%A8%ED%84%B4-%EC%A0%9C%EB%8C%80%EB%A1%9C-%EB%B0%B0%EC%9B%8C%EB%B3%B4%EC%9E%90)

- 개요
  - 런타임에서 알고리즘을 전략을 선택하여 객체 동작을 실시간으로 바뀌도록 하는 행위 디자인 패턴
  - 알고리즘 변형이 빈번하게 필요한 경우에 적합한 패턴
  - 자바로 생각해보면 interface로 알고리즘을 추상화하고
    - 구현체에서 전략 알고리즘을 구현한다.
- 전략패턴의 구조
  - 전략 알고리즘 객체들
    - 알고리즘, 행위, 동작을 객체로 정의한 구현체
  - 전략 인터페이스
    - 모든 전략 구현체에 대한 공용 인터페이스
  - 컨텍스트
    - 알고리즘을 실행할때마다 해당 알고리즘과 연결된 전략 메서드 호출
  - 클라이언트
    - 특정 전략 객체를 컨텍스트에 전달함으로써 전략을 등록하거나 변경하여 전략 알고리즘 실행한 결과 누림
- 쉽게 말해 OOP의 집합체

### Strategy Pattern in JS

[Design Patterns - Strategy Pattern in JavaScript](https://dev.to/carlillo/design-patterns---strategy-pattern-in-javascript-2hg3)

- 조건문을 객체매핑을 바꾸어 유지보수성을 높인다
- 현재 코드에서는
  - 객체 매핑을 사용해서 동적으로 key를 찾게 적용

```jsx
const renderers: Partial<Record<RenderKeyType, RenderFunction>> = {
  vehicleType: (item) =>
    renderLabel(VEHICLE_TYPE_RADIOS, VEHICLE_TYPE_KEY_PAIR[item.vehicleType]),
  fuelType: (item) =>
    renderLabel(
      VEHICLE_FUEL_TYPE_DROPDOWNS,
      VEHICLE_FUEL_TYPE_KEY_PAIR[item.fuelType]
    ),
  isPrivate: (item) =>
    renderLabel(
      VEHICLE_OWNERSHIP_RADIOS,
      VEHICLE_OWNERSHIP_KEY_PAIR[item.isPrivate]
    ),
  created: (item) => renderDate(item.created),
};
```

## 최종코드

```jsx
  const renderLabel = <T extends { key: string | number; label: Languages }>(
    list: ReadonlyArray<T>,
    key: T["key"],
  ) => {
    return (
      <span>
        {key
          ? (t(findLookupTableLabel(list, key) as MembershipAdminLanguages) ??
            "-")
          : "-"}
      </span>
    );
  };

  const renderDate = (date: string) => {
    return date ? <time>{formatICTDateTime(date)}</time> : <span>-</span>;
  };

  const renderers: Partial<Record<RenderKeyType, RenderFunction>> = {
    vehicleType: (item) =>
      renderLabel(VEHICLE_TYPE_RADIOS, VEHICLE_TYPE_KEY_PAIR[item.vehicleType]),
    fuelType: (item) =>
      renderLabel(
        VEHICLE_FUEL_TYPE_DROPDOWNS,
        VEHICLE_FUEL_TYPE_KEY_PAIR[item.fuelType],
      ),
    isPrivate: (item) =>
      renderLabel(
        VEHICLE_OWNERSHIP_RADIOS,
        VEHICLE_OWNERSHIP_KEY_PAIR[item.isPrivate],
      ),
    created: (item) => renderDate(item.created),
  };

  const renderTableData = (
    key: RenderKeyType,
    item: RenderItemType,
  ): jsx.JSX.Element => {
    return renderers[key] ? (
      renderers[key](item)
    ) : (
      <span>{item[key] || "-"}</span>
    );
  };
```
