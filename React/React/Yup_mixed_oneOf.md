## 문제 상황

- 사용자가 선택한 라디오 버튼 값이 라디오 객체 배열에 속하는 값인지 유효성 검사를 해줘야했다
- oneOf, mixed중 어떤걸 써야할지 고민이 되었는데 팀 컨벤션은 mixed를 쓰는것으로 결정이 되었다
- 근데 mixed를 쓰는게 yup공식문서에 맞게 쓰는것인가 궁금해졌다

## mixed

Creates a schema that matches all types, or just the ones you configure. Inherits from [`Schema`](https://github.com/jquense/yup?tab=readme-ov-file#Schema). Mixed types extends `{}` by default instead of `any` or `unknown`. This is because in TypeScript `{}` means anything that isn't `null` or `undefined` which yup treats distinctly.

```
import { mixed, InferType } from 'yup';

let schema = mixed().nullable();

schema.validateSync('string'); // 'string';

schema.validateSync(1); // 1;

schema.validateSync(new Date()); // Date;

InferType<typeof schema>; // {} | undefined

InferType<typeof schema.nullable().defined()>; // {} | null
```

Custom types can be implemented by passing a type `check` function. This will also narrow the TypeScript type for the schema.

```jsx
import { mixed, InferType } from 'yup';

let objectIdSchema = yup
  .mixed((input): input is ObjectId => input instanceof ObjectId)
  .transform((value: any, input, ctx) => {
    if (ctx.isType(value)) return value;
    return new ObjectId(value);
  });

await objectIdSchema.validate(ObjectId('507f1f77bcf86cd799439011')); // ObjectId("507f1f77bcf86cd799439011")

await objectIdSchema.validate('507f1f77bcf86cd799439011'); // ObjectId("507f1f77bcf86cd799439011")

InferType<typeof objectIdSchema>; // ObjectId
```

## oneOf

**`Schema.oneOf(arrayOfValues: Array<any>, message?: string | function): Schema` Alias: `equals`**

Only allow values from set of values. Values added are removed from any `notOneOf` values if present. The `${values}` interpolation can be used in the `message` argument. If a ref or refs are provided, the `${resolved}` interpolation can be used in the message argument to get the resolved values that were checked at validation time.

Note that `undefined` does not fail this validator, even when `undefined` is not included in `arrayOfValues`. If you don't want `undefined` to be a valid value, you can use `Schema.required`.

```jsx
let schema = yup.mixed().oneOf(["jimmy", 42]);

await schema.isValid(42); // => true
await schema.isValid("jimmy"); // => true
await schema.isValid(new Date()); // => false
```

## 한국어로 정리

- Mixed
  - 타입을 좁히는데 집중
  - 특정 값 검증 보다는 타입만 정의하는데 집중
  - 이 데이터가 어떤 `타입`이여야하나를 설정
- oneOf
  - 주어진 배열 안에 있는 값들만 허용하는 유효성 검사기
  - yup 스키마인 mixed,string,number등에 추가적으로 적용하는 메서드
  - 이 데이터가 어떤 `값`이여야하나를 설정

## 나의 상황에 대입하기

```jsx
const schema = object({
  status: mixed<(typeof DRIVER_STATUS_RADIOS)[number]["key"]>()
    .required(COMMON_ERROR_MESSAGE.FIELD)
    .test({
      name: "required",
      test: (value) => !!value,
      message: ADMIN_ERROR_MESSAGE.DRIVER_TYPE_REQUIRED,
    }),
});

export const DRIVER_STATUS_RADIOS = [
  { key: "activated", label: LANGUAGE_LABEL.ACTIVATED },
  { key: "deactivated", label: LANGUAGE_LABEL.DEACTIVATED },
  { key: "pending", label: LANGUAGE_LABEL.PENDING },
] as const;
```

- DRIVER_STATUS_RADIOS는 운전자의 상태를 저장해놓은 객체 배열이다 → 이게 라디오버튼 리스트 값이 된다! 사용자가 선택한 라디오버튼 값은 해당 객체 배열에 있는 값중 하나여야한다.
- status에 hover해서 타입 추론을 할때 “activated” | “deactivated” | “pending” 이런식으로 문자열 리터럴로 추론이 되어야한다
- oneOf를 써도 DRIVER_STATUS_RADIOS 배열내 속하는 값인지 검증이 될것이고
- mixed를 써도 문자열 리터럴 타입으로 좁힐수 있기에 어떤걸 써도 무방하다는 결론을 내렸다!
