[https://medium.com/@yujso66/번역-typescript-4-9-73f94ec1ce9c](https://medium.com/@yujso66/%EB%B2%88%EC%97%AD-typescript-4-9-73f94ec1ce9c)

https://ko-de-dev-green.tistory.com/110

### 버전

- 작성시기 ts버전 5.9.3

### 왜 as 대신 satisfies를 쓰자고 제안했을까?

- 코드를 보다보니 Key-val 쌍에 as로 타입단언을 했다.
- 무려 Key-Val쌍이 100개나 있었다.

```jsx
VALIDITY_PERIOD: "Validity preiod" as adminLanguages ,
  VEHICLE: "Vehicle" as adminLanguages,
  YOUR_EMAIL_DOT_COM: "your@email.com" as adminLanguages,
```

- as를 쓰면 타입 체크의 이점을 하나도 누릴수 없게 된다

### satisfies의 동작원리를 알아보자!

- TypeScript 4.9버전에서 새로 도입된 기능
- 변수나 객체가 특정 타입을 만족하는지 검사할때 사용
- 값을 특정 타입과 비교하지만 해당 타입으로 강제 변환하지 않고 원래의 타입 추론을 유지함

```jsx
satisfies Record<string, MembershipAdminLanguages>
```

- 객체 자체를 as const로 선언하긴 하지만 key-val를 as로 타입 선언하는 것이 문제라 생각
- 고로 satisfies를 쓰자고 건의

### 표면적으로 as대신 satisfies를 써야하는 이유

[타입스크립트 타입 호환성 문제 해결하기 "as const vs satisfies"](https://soobing.github.io/typescript/as-const-vs-satisfies/)

- as는 잘못된 타입을 적거나 타입이 빠져도 에러를 나타내지 않는다!
- satisfies를 쓰게 되면 필요한 값이 하나라도 빠지면 에러를 나타내게 된다

```jsx
type Colors = "red" | "green" | "blue";

type RGB = [red: number, green: number, blue: number, ...rest: string[]]; //맨아래 섹션 ## type element inside array. 에서 추가 설명할것을 미리 알려드림

const palette:Record<Colors, string | RGB> = {
  red: [255, 0, 0],
  green: "#00ff00",
   /**
 Object literal may only specify known properties, and 'bleu' does not exist in type 'Record<Colors, string | RGB>'
  */
  bleu: [0, 0, 255],

};

/**
Property 'sort' does not exist on type 'string | RGB'.
  Property 'sort' does not exist on type 'string'.ts(2339)
*/
palette.blue.sort();
```

- blue의 경우 단순 오타
- sort를 사용할 수 없다
  - 왜냐면 bleu의 타입이 String인지 RGB인지 모르기 때문

```jsx
const palette = {
  red: [255, 0, 0],
  green: "#00ff00",
  blue: [0, 0, 255],

} satisfies Record<Colors, string | RGB>

palette.blue.sort();
```

- 이렇게 쓰면 string | RGB 타입을 평가한 다음 palette 안에 할당된 값을 만족시키는게 있는지 확인
- bleu라고 오타내면 또 알려줌
- 정리
  - Key의 오타 체크, value의 타입체크도 가능하다

### as const와 satisfies의 차이는?

- as const 는 타입 단언이다
  - 장점
    - 불변성 보장 ( read-only)
    - 정확한 타입 추론
    - 타입을 좁힐 수 있으며
    - 런타임 안정성을 가질 수 있음
- satisfies
  - 명시적인 타입체크
  - 코드 읽기 쉬움
- 두 방법의 차이점
  - 타입의 명시성
    - as const는 객체 리터럴의 모든 속성을 최대한 정확하게 추론하게 함
    - satisfies는 개발자가 만족해야할 명시적인 인터페이스 제공
  - 용도와 적용 범위
    - as const는 주로 값의 변동성 제한, 리터럴 타입 유지하는 경우
    - satisfies는 객체가 특정 인터페이스나 타입 만족하는지 확인

### palette의 예시에서 as const를 적용한다면?

```jsx
type Colors = "red" | "green" | "blue";
type RGB = [red: number, green: number, blue: number, ...rest: string[]];

const palette: Record<Colors, string | RGB> = {
  red: [255, 0, 0],
  green: "#00ff00",
  blue: [0, 0, 255],
} as const;

```

- as const를 사용하면 객체의 모든 속성이 read-only가 되고 값이 리터럴 타입으로 추론된다
- red : [255,0,0]이 [number,number,number]가 아니라 [255,0,0] 이라는 튜플 리터럴 타입인 이유
  ```jsx
  const red = [255, 0, 0] as const;
  ```
  - 값이 정확히 [255,0,0]으로 고정된 튜플 리터럴 타입이 된다. 딱 [255,0,0]만 허용한다.
    - as const를 사용하면 값이 고정되기 때문에 수정이 불가능
    - 애초에 RGB를 나타내지도 않음
- palette.red의 타입이 [255,0,0] 리터럴 튜플로 고정된다
- RGB타입과 정확히 일치하지 않는다

### 근데 왜 as를 써도 타입검사에서 문제가 없을까?

- 이미 Import시 union literal type으로 타입지정을 해놨기 때문이다!

```jsx
export type AdminLanguages =
  keyof (typeof adminResources)["en"]["translation"];
```

- 비즈니스 로직 자체가 번역이 필요하다.
  - 초기에는 번역시트가 없어서 as로 타입단언을 하고
  - 번역시트가 올라오면 as를 빼주는 프로세스를 취하는게 더 효율적이다!
