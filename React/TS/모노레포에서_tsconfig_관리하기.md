```jsx
  "import/resolver": {
        typescript: {
          alwaysTryTypes: true,
          project: ["packages/typescript-config/base.json"],
        }
```

- import/resolver project의 역할
- project: [”..”] 는 Eslint 플러그인 중 eslint-import-resolver-typescript가 사용할 tsconfig.json을 지정함
- 이 설정이 없으면 기본적으로 최상위 tsconfig.json만 사용되며 모노레포나 서브디렉토리에 여러 Config가 있는경우, 제대로 해석하기 어려울 수 있다

[GitHub - import-js/eslint-import-resolver-typescript: This resolver adds `TypeScript` support to `eslint-plugin-import(-x)`](https://github.com/import-js/eslint-import-resolver-typescript#project)

## tsconfig.json을 커스텀화하기

[[Typescript] 상황별 tsconfig.json 설정하기](https://bo5mi.tistory.com/270)

- 모노 코드는 tsconfig.json이 base.json과 package.json으로 분리되어있었다.

```jsx
ㄴ apps/
	ㄴ 서비스1
	ㄴ 서비스2
ㄴ packages / 공용 코드들
```

- 그렇기에 서비스가 많아지거나/커질수록 모노레포 별로 tsconfig를 따로 커스텀해주는게 유용한 상황이였다.
- 그렇기 때문에 packages내에 공용 tsconfig 설정을 두고, 모노레포에서 extends해서 쓰면 관리하기가 유용해진다!
- 현재 모노레포 내에서는 base.json, package.json으로 나눠두었는데, base.json은 공용 tsconfig 역할을 한다.
- package.json이 굳이 필요한 이유는 base.json만 있을경우 Node가 모듈로 인식하지 못하기 때문이다!
