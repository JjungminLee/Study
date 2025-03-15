## 개요

```jsx
 [K in FilterConditionTypes]: (props: FilterConditionProps<K>, key: number) => jsx.JSX.Element;
```

- 회사 코드에서 이렇게 배열같이 생겼는데 뭔가 인덱스 시그니처도 같으며 같기도 아니같기도 한 코드를 봤다.
  ```tsx
  type A = { foo: string; bar: number };
  type FooType = A["foo"]; // string
  ```
  - 이거랑 비슷한 문법이다
  - 객체의 키에 대한 타입 접근!!

### 코드를 쉽게 풀어보자!

```jsx
{
  radio: (props: FilterConditionProps<"radio">, key: number) => JSX.Element;
  input: (props: FilterConditionProps<"input">, key: number) => JSX.Element;
  dropdown: (props: FilterConditionProps<"dropdown">, key: number) =>
    JSX.Element;
  calendar: (props: FilterConditionProps<"calendar">, key: number) =>
    JSX.Element;
}
```

- 실제로 타입스크립트는 이렇게 이해한다고 한다.

## Mapped Type

[맵드 타입 | 타입스크립트 핸드북](https://joshua1988.github.io/ts/usage/mapped-type.html#%EB%A7%B5%EB%93%9C-%ED%83%80%EC%9E%85-mapped-type-%EC%9D%B4%EB%9E%80)

### 정의

- 기존에 정의되어 있는 타입을 새로운 타입으로 변환해주는 문법이다.

```jsx
{ [ P in K ] : T }
{ [ P in K ]? : T }
{ readonly [ P in K ] : T }
{ readonly [ P in K ]? : T }
```

- 제네릭을 사용해 새로운 타입을 만든다!

### 어떨때 MappedType이 유용할까?

- as-is

```jsx
interface PersonPartial {
   name?: string;
   age?: number;
}

interface PersonReadonly {
   readonly name: string;
   readonly age: number;
}
```

- to-be

```jsx
interface Person {
   name: string;
   age: number;
}

type ReadOnly<T> = {
   readonly [P in keyof T]: T[P];
};

type ParTial<T> = {
   [P in keyof T]?: T[P];
};

type PersonPartial = Partial<Person>;

type ReadonlyPerson = Readonly<Person>;
```

- MappedType 문법 사용하여 마치 함수를 이용하는 것처럼 속성들을 순회해서 변경하고 그 결과물을 Type alias에게 변환해준다.
- 제네릭과 결합시 매우 강력해진다!

### 내가 PR에서 제안해본 코드

```tsx
type BaseConditionPropsMap = {
  radio: { count: number; width: number };
  input: { placeholder: Languages };
  dropdown: { placeholder: Languages };
  calendar: {};
};
type FilterConditionPropsMap = {
  [K in keyof BaseConditionPropsMap]: BaseConditionProps &
    BaseCondtionPropsMap[K];
};
```

- BaseCondtionPropsMap[K]가 배열처럼 쓴다고 생각했는데 MappedType의 기본 문법이라고 한다.
  - 키에 대한 타입을 가져오는
- K를 기준으로 해당 키에 대한 타입을 가져온다!
