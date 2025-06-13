## 개요

- 신규 프로젝트 개발중에 백엔드에서 가끔 들어올수도 있는 undefined 데이터에 대한 예외처리로 스트레스 받아하고 있었다.
- 그런데 어느날! 실운영중인 서비스에서 Undefined 때문에 엑박떠버렸다. ㄸㄹㄹ
- 백엔드로부터 받아오는 데이터의 타입이 Undefined인지 판단하는게 필요한 시점이라는 생각이 들었다.
- 사족을 붙이자면 yup이나 zod를 가지고 유효성 검사를 판단해줘도 되지만 이러면 코드가 겁나길어진다. 코드가 보기 싫어질 정도로..
- 그래서 찾아낸 런타임 타입검사기 Typia
- 클라이언트가 백엔드로부터 받아야하는 타입이 일치하는지 일치하지 않는지 검사하는 컴파일러 기반 도구라고 생각하면 된다!
- 가이드 문서인데, 매우 상세하게 잘 나와 있다!

[Typia Guide Documents](https://typia.io/docs/)

https://okky.kr/articles/1428465

## typia의 런타임 함수들

```jsx
// RUNTIME VALIDATORS
export function is<T>(input: unknown): input is T; // returns boolean
export function assert<T>(input: unknown): T; // throws TypeGuardError
export function assertGuard<T>(input: unknown): asserts input is T;
export function validate<T>(input: unknown): IValidation<T>; // detailed
```

- 4개 가지고 만능으로 사용할 수 있다

## assert()

```jsx
export function assert<T>(input: T): T;
export function assert<T>(input: unknown): T;
```

- 잘못된 타입이 들어올 경우 TypeGuadError를 던져준다

```jsx
import typia, { tags } from "typia";
import { v4 } from "uuid";

typia.assert <
  IMember >
  {
    id: v4(),
    email: "kathy@gmail.com",
    age: 18, // wrong, must be greater than 19
  };

interface IMember {
  id: string & tags.Format<"uuid">;
  email: string & tags.Format<"email">;
  age: number &
    tags.Type<"uint32"> &
    tags.ExclusiveMinimum<19> &
    tags.Maximum<100>;
}
```

- interface로 정의한 IMember에 맞게 데이터가 정확하지 않으면 TypeGuadError가 나온다.
- 후술할테지만 tags가 기가막힌다. 무슨 디비 타입 보는줄

## **`assertEquals()`**

```jsx
export function assertEquals<T>(input: T): T;
export function assertEquals<T>(input: unknown): T;
```

- 이 함수의 경우, 불필요한 속성을 금지하여 더 강력한 검사를 한다

```jsx
import typia, { tags } from "typia";
import { v4 } from "uuid";

typia.assert <
  IMember >
  {
    id: v4(),
    email: "kathy@gmail.com",
    age: 24,
    school: "Soongsil university", // IMember에 정의되지 않은 속성!
  };

interface IMember {
  id: string & tags.Format<"uuid">;
  email: string & tags.Format<"email">;
  age: number &
    tags.Type<"uint32"> &
    tags.ExclusiveMinimum<19> &
    tags.Maximum<100>;
}
```

## is

```jsx
export function is<T>(input: T): input is T;
export function is<T>(input: unknown): input is T;
```

- 값의 타입을 검사하고 싶을때 is를 사용한다. 해당 입력값이 T를 만족하면 true, 만족하지 않으면 false를 리턴한다.

```jsx
import typia, { tags } from "typia";
import { v4 } from "uuid";

const matched: boolean =
  typia.is <
  IMember >
  {
    id: v4(),
    email: "samchon.github@gmai19l.com",
    age: 30,
  };

console.log(matched); // true

interface IMember {
  id: string & tags.Format<"uuid">;
  email: string & tags.Format<"email">;
  age: number &
    tags.Type<"uint32"> &
    tags.ExclusiveMinimum<19> &
    tags.Maximum<100>;
}
```

### AOT?

- 선행 컴파일을 의미하는데, class-validator, ajv같은 검증 라이브러리들이 AOT이다.
- 결국 typia의 장점은 typescript로 정의한 타입,인터페이스만으로 빠르게 검증할수 있다는 것이다.
- 사실상 typia를 도입한게 이 점이기도 하다. 스키마를 매번 쓰는게 짜치기 때문..ㅠ
- typia

```jsx
interface User {
  id: number;
  name: string;
}

typia.assert < User > input;
```

- class-validator
  - 매번 데코레이터를 붙여줘야한다

```jsx
import { IsInt, IsString } from "class-validator";
import { plainToInstance } from "class-transformer";
import { validateOrReject } from "class-validator";

class User {
  @IsInt()
  id: number;

  @IsString()
  name: string;
}

const user = plainToInstance(User, input);
await validateOrReject(user);
```

## equals

```jsx
export function equals<T>(input: T): input is T;
export function equals<T>(input: unknown): input is T;
```

- is는 불필요한 속성까지 잡아내지 못하는반면, equals는 불요한 속성까지 잡아낸다

```jsx
import typia, { tags } from "typia";
import { v4 } from "uuid";

const input: unknown = {
  id: v4(),
  email: "samchon.github@gmail.com",
  age: 30,
  extra: "superfluous property", // extra
};
const is: boolean = typia.is < IMember > input;
const equals: boolean = typia.equals < IMember > input;

console.log(is, equals); // true, false

interface IMember {
  id: string & tags.Format<"uuid">;
  email: string & tags.Format<"email">;
  age: number &
    tags.Type<"uint32"> &
    tags.ExclusiveMinimum<19> &
    tags.Maximum<100>;
}
```

## validate()

```jsx
export function validate<T>(input: T): IValidation<T>;
export function validate<T>(input: unknown): IValidation<T>;
```

- 입력값의 타입을 검증하고 입력값이 타입 T를 따르지 않을 경우, 모든 타입 오류를 상세하게 IValidation.IFailure.errors 배열에 저장
- 입력값이 타입 T를 정상적으로 만족하면 IValidation.ISuccess 객체 반환

```jsx
import typia, { tags } from "typia";

const res: typia.IValidation<IMember> =
  typia.validate <
  IMember >
  {
    id: 5, // wrong, must be string (uuid)
    age: 20.75, // wrong, not integer
    email: "kathy@gmail.com",
  };

if (!res.success) console.log(res.errors);

interface IMember {
  id: string & tags.Format<"uuid">;
  email: string & tags.Format<"email">;
  age: number &
    tags.Type<"uint32"> &
    tags.ExclusiveMinimum<19> &
    tags.Maximum<100>;
}
```

## tag

- Typescript만으로 세세한 검증이 어려울때 Type Tag와 Comment Tag를 통해 검증할수 있다

```jsx
interface IMember {
  // 방법 1: Type Tag
  age: number & typia.tags.Type<"uint32">;

  // 방법 2: Comment Tag
  /** @type int32 */
  score: number;
}
```

- typescript는 number로 퉁쳐지는데, tag를 사용하면 int, uint 까지 검증할수 있다.
- ipv4까지 있는거 보고 뒤집어질뻔 했다

## 실제 사용은 어떻게?

- 니즈는 백엔드에서 넘어오는 데이터들을 런타임에서 타입 검사하는 것이였다! 그래서 api.ts에서 typia 함수들을 적용해주었다.

```jsx
export const getFoodAPI = async (menuId: string) => {
  const path = `/food/${menuId}`;
  const { data } = await ax.get(path);

  const isMatched = is<GetFoodModel>(data);

  if (isMatched) return data;

  const response = validate<GetOdooRequestMenuServerModel>(data);

  if (!response.success) {
    const sanitizedData = sanitizeTypiaErrors<GetOdooRequestMenuServerModel>({
      method: "GET",
      path,
      data,
      errors: response.errors,
    });

    return sanitizedData;
  }
```

## 아쉬운점

- 런타임 검사기이기 때문에 제네릭을 사용할수 없다는점! 반드시 정적 타입을 명시해줘야한다.

```jsx
import { is, validate } from "typia";
const response = validate < FoodServerModel > data;
if (!res.success) {
  // 정의한 타입과 일치하지 않을때 에러 보여주고
  // 데이터 정제하는 로직
}
```

- FoodServerModel를 쓸거라고 명시해줘야한다.!!
- 런타임 검사기이기 떄문에 데이터 타입을 제네릭으로 받을 수 없어 if(!res.success) 이후의 로직들을 공용함수로 만들 수 없다 ㅠㅠ
