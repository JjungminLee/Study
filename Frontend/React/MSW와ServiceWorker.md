## Service Worker

- MSW를 알기 전에 서비스워커의 개념부터 짚고 넘어가야한다
- 특정 사이트의 하나 혹은 그 이상의 페이지를 제어하는 스크립트 이며, 이벤트 워커로서 자바스크립트로 작성된 파일이다. 서비스워커는 자신이 제어하는 페이지에서 발생하는 이벤트를 수신할 수 있다.
- 여기서 중요한건 **웹에서의 네트워크 요청과 같은 이벤트를 가로채서 수정**할 수 있다는 점이다!

### Service Worker의 위치

- 어떻게 네트워크 요청을 가로채고, 리소스를 캐싱할 수 있을까?
  - 리소스를 캐싱할수 있는 이유는 **브라우저 또는 탭 외부에 위치**하기 때문이다.
  - 브라우저와 네트워크 사이의 새로운 계층에 존재한다. 브라우저와 네트워크 사이의 연결과 관계없이 서비스 워커는 브라우저와 독립적인 연결을 갖는다.
    - 오프라인 상태 또는 네트워크가 느린 상태일때 브라우저에 캐시된 리소를 전달할 수 있다.
    - 서비스 워커는 독립적이기 때문에 사용자가 브라우저에서 사용중인 모든 탭을 닫아도 서비스 워커는 여전히 네트워크와 통신할 수 있다!
      - 푸쉬알림 수신하면 브라우저에 전달할 수 있고, 백그라운드에서 앱의 데이터를 최신화 할 수 있다!
      - Service Worker는 브라우저와 네트워크 사이에서 프록시 서버 역할을 한다고 보면된다.
- 코드로 봤을때는

```jsx
navigator.serviceWorker;
```

- msw 코드를 까보면 이런식으로 사용한다.

[Navigator: serviceWorker property - Web APIs | MDN](https://developer.mozilla.org/en-US/docs/Web/API/Navigator/serviceWorker)

```jsx

// getWorkerInstance.ts
const registrationResult = await until<Error, ServiceWorkerInstanceTuple>(
    async () => {
      const registration = await navigator.serviceWorker.register(url, options)
      return [
        // Compare existing worker registration by its worker URL,
        // to prevent irrelevant workers to resolve here (such as Codesandbox worker).
        getWorkerByRegistration(registration, absoluteWorkerUrl, findWorker),
        registration,
      ]
    },
```

- 이렇게 내장객체 navigator.serviceWorker.register를 통해 service worker를 등록한다.
  - url이 내 로컬 프로젝트 안에 ./mockServiceWorker.js라서 service worker등록하면 해당 스크립트를 실행시킨다.
  - vite,next.js사용시 반드시 Public 하위에 해당 Js파일을 만들어야 한다. 정적파일로 제공하여 브라우저가 찾을수 있게 하기 위함!

## MSW를 써보자!

- 해당 글을 보면 Mocking을 직접 구현한다. 하지만 MSW라는 라이브러리르 사용하면 더욱 편리하다!
  [서비스 워커에 대해 알아보고 Mock Response 만들기 | 카카오엔터테인먼트 FE 기술블로그](https://fe-developers.kakaoent.com/2022/221208-service-worker/)
- 서버를 향한 실제 네트워크 요청을 가로채서 (Intercept) 모의 응답 (Mocking Response)를 보내주는 API Mocking 라이브러리이다.
  - 굳이 MSW를 안써도 된다. Nock, axios-mock-adapter 등 다양하다
  - 하지만 MSW는 Axios나 Fetch같은 HTTP 라이브러리를 사용한 Mocking 작업을 간단하게 만들어준다.
- 앞서 말했던 Service Worker의 HTTP 요청 가로채기 기능을 사용해서 구현한다고 보면 된다.

![image.png](attachment:299376e0-165e-44de-920b-69eccc179ff5:image.png)

## MSW의 동작 순서를 알아보자

1. 브라우저가 Service Worker에 요청을 보낸다
2. Service Worker가 해당 요청을 가로채서 복사한다
3. 서버에 요청을 보내지 않고, MSW라이브러리의 핸들러와 매칭시킨다.
4. MSW가 등록된 핸들러에서 Mock Response를 Service Worker에 전달한다.
5. 마지막으로 Service Worker가 Mock Response를 브라우저에게 전달한다.

한줄 정리 : Axios, ky, fetch 같은 HTTP 클라이언트의 네트워크 요청을 가로채서 Mock data를 넘겨줌

- 고로 API 요청 하는 함수를 만들어 놓으면 해당 함수 호출시 MSW가 작동해서 Mock data를 던져준다!

## Intercepting

- 가로채기의 핵심은 Predicate과 Resolver 함수이다.

> MSW 문서 내
>
> - `predicate` (`string | RegExp`), a request path predicate;
> - `resolver`, ([**`Response resolver`**](https://mswjs.io/docs/concepts/response-resolver)), a function that decides how to handle an intercepted request.

```jsx
export const mockingVehicle = [
  http.get < never,
  never,
  GetVehiclesServerModel >
    ("/vehicles",
    async () => {
      return HttpResponse.json(vehicleDB, { status: 200 });
    }),
];
```

- 여기서 get이 Request Handler이다. get이 http 요청을 가로챈다.

```jsx
declare const http: {
    all: HttpRequestHandler;
    head: HttpRequestHandler;
    get: HttpRequestHandler;
    post: HttpRequestHandler;
    put: HttpRequestHandler;
    delete: HttpRequestHandler;
    patch: HttpRequestHandler;
    options: HttpRequestHandler;
};

export { type HttpRequestHandler, type HttpResponseResolver, http };

```

- 이렇게 http 메서드마다 핸들러가 붙어있다.

### Resolver

- 가로챈 HTTP 요청에 대해 어떻게 응답할지 결정하는 함수이다.

```jsx
http.get("/pets", ({ request, params, cookies }) => {
  return HttpResponse.json(["Tom", "Jerry"]);
});
```

- `({ request, params, cookies }) =>`
  - 이 친구들이 응답을 어떻게 줄지 결정하는 resolver이다.
- 지피티 왈, 가짜 백엔드라고 생각하면 된다고 한다.

### Predicate

- MSW에서 intercept할 요청을 식별할 기준을 의미한다.

```jsx
http.get("/pets", resolver); // 경로가 "/pets"인 GET 요청만 처리
http.post(/^\/auth\/.*/, resolver); // "/auth/..."로 시작하는 POST 요청 처리
```

- 여기서 `/pets`가 predicate가 된다.
- 또한 정규식도 Predicate가 된다

## Call Signature

- 가장 보편적인게 GET메서드라 이를 기준으로 설명하겠다.

```jsx
http.get<PathParams, RequestBodyType, ResponseBodyType>(
  predicate: string | RegExp,
  resolver: ResponseResolver<
    HttpRequestResolverExtras<Params>,
    RequestBodyType,
    ResponseBodyType
  >,
  options?: RequestHandlerOptions
```

### Never type

```jsx
  http.get<never, never, GetVehiclesServerModel>("/vehicles", async () => {
    return HttpResponse.json(vehicleDB, { status: 200 });
  }),
```

[타입스크립트의 Never 타입 완벽 가이드](https://ui.toast.com/posts/ko_20220323)

[[TypeScript] 함수의 기타 반환타입(unknown, void, never)](https://ramincoding.tistory.com/entry/TypeScript-%ED%95%A8%EC%88%98%EC%9D%98-%EA%B8%B0%ED%83%80-%EB%B0%98%ED%99%98%ED%83%80%EC%9E%85unknown-void-never)

- 코드에서 왜 never를 쓰는지 궁금했는데, never는 `존재하지 않음`을 의미하는 타입이라고 한다.
- get이다 보니 당연히 Request는 never고, Response(여기선 status나 Header)도 Never다. 딱 json 반환 바디만 정의해준 타입이 필요하다!

## Appendix

[Mocking HTTP Request in React using Mock Service Worker](https://medium.com/simform-engineering/mocking-http-request-in-react-using-mock-service-worker-0f9c5400b035)

[Describing REST API](https://mswjs.io/docs/network-behavior/rest)

[Next.js에서 MSW(Mock Service Worker)로 네트워크 Mocking하기 | 올리브영 테크블로그](https://oliveyoung.tech/2024-01-23/msw-frontend/)

[Service Worker API - Web API | MDN](https://developer.mozilla.org/ko/docs/Web/API/Service_Worker_API)

[서비스 워커에 대해 알아보고 Mock Response 만들기 | 카카오엔터테인먼트 FE 기술블로그](https://fe-developers.kakaoent.com/2022/221208-service-worker/)

[MSW(Mock Service Worker)로 더욱 생산적인 FE 개발하기](https://velog.io/@khy226/msw%EB%A1%9C-%EB%AA%A8%EC%9D%98-%EC%84%9C%EB%B2%84-%EB%A7%8C%EB%93%A4%EA%B8%B0)
