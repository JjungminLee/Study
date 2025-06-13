## 개요

![image](https://github.com/user-attachments/assets/c31154e6-1572-4a25-ac7f-a648523eeee6)

```tsx
<Input
  value={watch(`vehicles.${i}.regNo`)}
  maxLength={100}
  hasError={!!errors.vehicles?.[i]?.regNo}
  register={register(`vehicles.${i}.regNo`)}
/>
```

- register와 value를 함께 쓴 아주 끔찍한 코드다
- 발생하는 문제점
  - input필드에 무한렌더링 걸림
  - useFieldArray를 쓰고 있다면 필드 배열 A를 수정했는데 필드 배열B가 수정되는 기묘한 현상이 발생함
- 코드 원작자는 왜 이렇게 썼을까?
  - 사용자가 입력한 input value를 **추적**해서 에러메세지를 띄우기 위함!

### 중요한건 추적이다!

- 리액트가 상태를 관리하냐 마냐 이걸로 controlled input과 Uncontrolled input을 구분할 수 있다.

## react-hook-form

- 비제어 컴포넌트 방식

> https://react-hook-form.com/faqs#Whatifyoudonthaveaccesstoref
>
> ## **Performance of React Hook Form**
>
> Performance is one of the primary reasons why this library was created. React Hook Form relies on an uncontrolled form, which is the reason why the `register` function captures `ref` and the controlled component has its re-rendering scope with `Controller` or `useController`. This approach reduces the amount of re-rendering that occurs due to a user typing in an input or other form values changing at the root of your form or applications. Components mount to the page faster than controlled components because they have less overhead. As a reference, there is a quick comparison test that you can refer to at [this repo link](https://github.com/bluebill1049/react-hook-form-performance-compare).

- 간단히 말하면 RHF는 register함수가 ref를 캡처한다고 한다.
- 이러한 방식은 사용자가 입력창에 입력할때 발생하는 불필요한 리렌더링을 줄인다.
- 제어 컴포넌트보다 오버헤드가 적기 때문에 컴포넌트가 페이지에 더 빠르게 마운트된다.

## controlled component, uncontrolled component

- uncontrolled의 경우
  - ref를 사용해 DOM에서 처리한다 (직접 DOM에서 읽어오는 방식)
    - 변경된 필드만 다시 렌더링 하기 때문에 최적화된 성능을 제공한다
    - 필드가 변경되더라도 전체 폼이 리렌더링 되지 않는다
  - register()
  - 주로 defaultValue나 defaultChecked를 사용하여 초기값을 설정하고, ref를 통해 직접 값을 가져오거나 설정한다.
- controlled의 경우
  - onChange,value
  - useState를 이용한 방식이다.
  - 값이 변경될때마다 컴포넌트가 리렌더링 된다.
  - form 요소의 상태를 React의 state로 관리한다.
  - form 요소의 값이 항상 React의 state와 동기화되어 있다
- 리액트 공식문서의 글을 보길 추천한다!

[Sharing State Between Components – React](https://react.dev/learn/sharing-state-between-components#controlled-and-uncontrolled-components)

### Controller

- RHF의 기조는 비제어 컴포넌트 이다. 하지만 만약 제어 방식이 필요하다면, Controller를 써야한다.
- UI 라이브러리들은 대부분 제어 컴포넌트 방식으로 이루어져있다. 고로 MUI같은 React UI 라이브러리를 사용하려면 Controller를 사용해야한다
- Controller도 사용하는 방법이 2가지 있다.

  - Cotroller 컴포넌트 사용

    ```jsx
    import ReactDatePicker from "react-datepicker"
    import { TextField } from "@material-ui/core"
    import { useForm, Controller } from "react-hook-form"

    type FormValues = {
      ReactDatepicker: string
    }

    function App() {
      const { handleSubmit, control } = useForm<FormValues>()

      return (
        <form onSubmit={handleSubmit((data) => console.log(data))}>
          <Controller
            control={control}
            name="ReactDatepicker"
            render={({ field: { onChange, onBlur, value, ref } }) => (
              <ReactDatePicker
                onChange={onChange} // send value to hook form
                onBlur={onBlur} // notify when input is touched/blur
                selected={value}
              />
            )}
          />

          <input type="submit" />
        </form>
      )
    }
    ```

  - useController 훅 사용

    ```jsx
    import { TextField } from "@material-ui/core";
    import { useController, useForm } from "react-hook-form";

    function Input({ control, name }) {
      const {
        field,
        fieldState: { invalid, isTouched, isDirty },
        formState: { touchedFields, dirtyFields },
      } = useController({
        name,
        control,
        rules: { required: true },
      });

      return (
        <TextField
          onChange={field.onChange} // send value to hook form
          onBlur={field.onBlur} // notify when input is touched/blur
          value={field.value} // input value
          name={field.name} // send down the input name
          inputRef={field.ref} // send input ref, so we can focus on input when error appear
        />
      );
    }
    ```

## Register

[register](https://react-hook-form.com/docs/useform/register)

| `value`

| `unknown` | Set up `value` for the registered input. This prop should be utilised inside `useEffect` or invoke once, each re-run will update or overwrite the input value which you have supplied. |
| --------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |

- register에 value를 두고 싶으면 useEffect로 감싸거나 정적인 변수를 써야한다. (한번만 사용하게끔)
- 그렇지 않는다면 입력 값이 업데이트 되거나 덮어써질수 있다.

```tsx
<input {...register("checkbox")} type="checkbox" value="A" />
<input {...register("checkbox")} type="checkbox" value="B" />
<input {...register("checkbox")} type="checkbox" value="C" />

```

- 심하면 input에서 무한 랜더링 걸릴수도….

## value와 defaultValue

- 비제어 컴포넌트에서 defaultValue가 필요하다고 말했다
- value와 defaultValue의 차이를 알아보자

| 항목        | `value`                              | `defaultValue`                                      |
| ----------- | ------------------------------------ | --------------------------------------------------- |
| 의미        | 현재 입력값 (React가 제어)           | 초기 입력값 (DOM이 제어)                            |
| 제어 방식   | **Controlled (제어 컴포넌트)**       | **Uncontrolled (비제어 컴포넌트)**                  |
| 값 변경     | 반드시 `onChange`와 함께 상태로 변경 | 사용자가 자유롭게 입력 가능 (React가 관여하지 않음) |
| 값 업데이트 | 상태가 변하면 input도 리렌더됨       | 초기 렌더링 시 한 번만 적용되고 이후엔 무시됨       |

- controlled 방식 예시코드

```jsx
const [name, setName] = useState("홍길동");

<input value={name} onChange={(e) => setName(e.target.value)} />;
```

- 입력값은 항상 `name` 상태와 동기화됨.
  - 사용자가 입력할 때마다 `onChange` → `setName()` → 다시 렌더링됨.
  - **React가 완전히 제어**.
- uncontrolled방식 예시코드

```jsx
<input defaultValue="홍길동" />
```

- 초기값은 "홍길동"으로 보이지만, 그 뒤부터는 사용자가 자유롭게 수정.
  - **React가 이후 입력값을 관여하지 않음**.
  - 값을 가져오고 싶으면 `ref`로 읽어야 함.

---

## 💡 react-hook-form에서는?

- `defaultValue` 또는 `defaultValues`를 통해 초기값을 설정하고,
- 내부적으로는 비제어 방식이므로 **자동으로 상태를 추적**

```jsx
const formMethod =
  useForm <
  FormCreateVehicle >
  {
    defaultValues: VEHICLE_INIT_FORM,
    mode: "onTouched",
    resolver: yupResolver(schema),
  };
```

### Appendix

[[React] controlled vs. uncontrolled input](https://jihyundev.tistory.com/23)

[React Hook Form - performant, flexible and extensible form library](https://react-hook-form.com/)

[Controlled v/s Uncontrolled Component in React](https://medium.com/@agamkakkar/controlled-v-s-uncontrolled-component-in-react-2db23c6dc32e)

[[React] 제어 컴포넌트(Controlled Component)와 비제어 컴포넌트(Uncontrolled Component)](https://dori-coding.tistory.com/entry/React-%EC%A0%9C%EC%96%B4-%EC%BB%B4%ED%8F%AC%EB%84%8C%ED%8A%B8Controlled-Component%EC%99%80-%EB%B9%84%EC%A0%9C%EC%96%B4-%EC%BB%B4%ED%8F%AC%EB%84%8C%ED%8A%B8Uncontrolled-Component)

[Hook Form으로 상태 관리하기.](https://velog.io/@leitmotif/Hook-Form%EC%9C%BC%EB%A1%9C-%EC%83%81%ED%83%9C-%EA%B4%80%EB%A6%AC%ED%95%98%EA%B8%B0)

[[React Hook Form] mui와 같이 사용하기 | Controller, useController](https://velog.io/@boyeon_jeong/React-Hook-Form-Controller-useController-y6v2mfc9#1-%EC%BB%B4%ED%8F%AC%EB%84%8C%ED%8A%B8%EB%A5%BC-%EC%82%AC%EC%9A%A9--controller)
