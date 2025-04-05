## 문제사항

### 서버에서 넘겨주는 데이터

```jsx
{
	"FruitTypes": [
		{ "code": 0, "name": "Apple" },
		{ "code": 1, "name": "Banana" },
		{ "code": 2, "name": "Strawberry" },
		{ "code": 3, "name": "Cherry" },
		{ "code": 4, "name": "Tomato" }
	]
}
```

- 현재 서버(Nest.js)에서 넘겨주는 데이터는 위와 같은 형식이다.
  - 드롭다운에 보여주는 데이터기에 Enum으로 관리하고, code-name의 key-value쌍으로 데이터를 넘겨준다.
  - 하지만 유저마다 드롭다운으로 받아오는 데이터가 다르고, 현재는 이런 구조를 유지하지만 기획이 변경된다면 이런 key-value쌍에 변경이 생길수도 있다.

### 클라이언트에서 드롭다운 객체 관리

```jsx
export const FRUIT_TYPE_DROPDOWNS =
[
		{ "code": 0, "name": "Apple" },
		{ "code": 1, "name": "Banana" },
		{ "code": 2, "name": "Strawberry" },
		{ "code": 3, "name": "Cherry" },
		{ "code": 4, "name": "Tomato" }
] as const;

```

- 클라이언트 (React)는 서버에서 넘어오는 Enum데이터를 위와 같은 Key-pair로 관리한다.

### 클라이언트의 API데이터 모델

```jsx
// as-is
export interface GetFruitsClientModel {
  fruits: {
    fruitId: string;
    fruitype: keyof typeof FRUIT_TYPE_KEY_PAIR;
  }[];
  pageInfo: PageInfo;
}

// as -is
export interface GetFruitsServerModel {
  fruits: {
    fruitId: string;
    fruitype: keyof typeof FRUIT_TYPE_KEY_PAIR;
  }[];
  pageInfo: PageInfo;
}
```

- 서버 타입에 종속적으로 개발하게 된다면, 클라이언트 코드를 다 뜯어고쳐야 하는 문제가 발생한다. (as-is)
- 그래서 데이터 모델을 두가지로 나눌 것이다. 클라이언트 모델, 서버모델

## 해결책 : DDD

- DDD의 목적은 유지보수에 용이해야 한다.
- 고로, 클라이언트 모델은 유연하게, 서버 모델은 종속적으로 만들자

### 클라이언트모델은 유연해야한다

- 클라이언트 모델은 유저한테 보여주는 값에만 집중하다
- 고로 모든 필드를 string으로 처리한다.
- 클라이언트모델 설계시, vehicleType을 0으로 두는게 아닌 “SUV”로 지정한다.
- 오로지 유저가 보는 유효한 값에만 집중한다.

### 서버모델은 서버에 종속적이여야한다.

- 서버에게 보내주는 데이터에만 집중한다.
- 하기와 같은 전략을 생각해볼 수 있다
  - 서버모델에서 Enum으로 넘겨주는 값은 number로 통일한다

```jsx
// to-be
export interface GetFruitsClientModel {
  vehicles: {
    fruitId: string,
    fruitype: string, // 클라이언트에서는 어찌됐든 드롭다운이 렌더링해서 보여지기만 하면돼서 string으로 바뀐다.
  }[];
  pageInfo: PageInfo;
}

// to-be
export interface GetFruitsServerModel {
  vehicles: {
    fruitId: string,
    fruitype: number, // 서버에서는 드롭다운 값이 Enum 즉, Number로 들어오기 때문에 서버 종속적으로 맞춘다
  }[];
  pageInfo: PageInfo;
}
```

## 문의를 받았다

![image.png](attachment:a3387612-d932-432e-9958-751a3163b23b:image.png)

나름대로의 설득을 세가지로 해보았다.

1. 어찌됐든 UI단에서는 string으로 처리해야하기 때문에 클라이언트 모델인 client.ts에서는 fuelType의 실제 데이터 타입을 알필요가 없다고 생각하여 string으로 두었음
2. 향후 서버에서 fuelType을 문자열("0","1","disel")로 던져줄수도 있음 (희박한 확률이긴 함)
3. 서버에서 새로운 FuelType이 추가된 경우에도 ("hybrid" 같이 새로운 타입이 추가된 경우) 유연하게 대응하기 위해서는 클라이언트 타입을 string으로 두는것이 적합하다고 판단

## 결론

- 클라이언트 모델 / 클라이언트타입 또는 서버 모델 / 서버타입에 로직이 들어가서는 안된다
  - 최대한 원시타입으로 풀어주면서 유연하게 만들어야한다.
- 클라이언트 타입은 렌더링에 중점을 두고, 서버 타입은 서버에서 넘어오는 값 그대로 타입을 지정한다.
- Mapper함수를 두면 더 좋았을것 같다. 아마 사이드 플젝이였다면 매핑 시키는 로직을 더 추가했을것 같음!
