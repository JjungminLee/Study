## Partial

[Typescript Omit과 Partial 의 차이점](https://systorage.tistory.com/entry/Typescript-Omit%EA%B3%BC-Partial-%EC%9D%98-%EC%B0%A8%EC%9D%B4%EC%A0%90)

- 주어진 타입 T의 모든 프로퍼티를 선택적으로 만든다
- 모든 프로퍼티가 있어도 되고 없어도 된다.

```jsx
interface User {
    id: number;
    name: string;
    email: string;
}

type PartialUser = Partial<User>;

const user1: PartialUser = {};
const user2: PartialUser = { id: 1 };
const user3: PartialUser = { name: 'Alice', email: 'alice@example.com' };
출처: https://systorage.tistory.com/entry/Typescript-Omit과-Partial-의-차이점 [SY Storage:티스토리]
```

- 장점
  - 객체의 일부 프로퍼티만 업데이트할 때 유용
- 예시 상황
  - Partial<T>
    - API요청에서 선택적인 업데이트 데이터 보낼때

### Omit<T,K>

- 주어진 타입 T에서 특정 프로퍼티 K를 제거한 새로운 타입을 만든다.

```jsx
interface User {
    id: number;
    name: string;
    email: string;
}

type UserWithoutEmail = Omit<User, 'email'>;

const user: UserWithoutEmail = { id: 1, name: 'Alice' };

출처: https://systorage.tistory.com/entry/Typescript-Omit과-Partial-의-차이점 [SY Storage:티스토리]
```

- 장점
  - 특정 프로퍼티 제외한 나머지 프로퍼티만 필요할 때 유용
  - 불필요한 프로퍼티 제거하고 필요한 프로퍼티만 가지는 타입 만들때 사용
- 예시 상황
  - Omit<T,K>
    - 데이터베이스 엔터티에서 민감한 정보 제외한 데이터 반환시

### Union Type

- 여러개의 타입을 합쳐서 사용할 수 있게 한다

```jsx
let greeting: "hello" | "hi" = "hello";
greeting = "hi"; // ✅ 가능
greeting = "bye"; // ❌ 오류 발생
```

### Literal Type

- 특정한 값 자체를 타입으로 지정한다

```jsx
let greeting: "hello" = "hello"; // "hello"만 가능
greeting = "hi"; // ❌ 오류 발생
```

### 리터럴 타입과 유니온 타입을 조합한다면?

```jsx
type Status = "success" | "error" | "loading";

let currentStatus: Status = "success"; // ✅ 가능
currentStatus = "error"; // ✅ 가능
currentStatus = "loading"; // ✅ 가능
currentStatus = "failed"; // ❌ 오류 발생
```

### 실제 코드로 분석해보기

```jsx
export type variousLanguages =
  keyof (typeof languageResources)["en"]["translation"];

```

- languageResources)["en"]["translation"] 객체를 가져온다.
- 모든 키들을 Keyof를 사용해 타입으로 만든다.
- languageResources에 정의된 모든 키들의 유니온 타입을 생성한다.

```jsx
const languageResources = {
  en: {
    translation: {
      A: "A",
      B: "B",
      C: "C",
    },
  },
};
```

- 이 타입은 아래와 같이 유니온 타입이 된다.

```jsx
type languageResources = "A" | "B" | "C";
```

## Keyof

- 간단 정리 : 키를 유니온 타입으로 만든다
- 객체 타입의 모든 키를 유니온 타입으로 추출하는 역할

```jsx
const PAYMENT_STATUS = {
  PENDING: "pending",
  COMPLETED: "completed",
  FAILED: "failed",
};

type PaymentStatusKeys = keyof typeof PAYMENT_STATUS;
// "PENDING" | "COMPLETED" | "FAILED"

```

### typeof

- 객체 또는 변수를 타입으로 변환

## Valueof

- 간단 정리 : 값을 유니온 타입으로 만든다
- 객체 값들을 유니온 타입으로 가져온다

```jsx
type PaymentStatusValues = typeof PAYMENT_STATUS[keyof typeof PAYMENT_STATUS];
// "pending" | "completed" | "failed"

```
