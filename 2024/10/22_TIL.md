# Radix-UI(1) - 주요 특징 과 createContext 구조 분석

## Radix UI 주요 특징

- **접근성**: Radix UI는 접근성을 우선시하여 모든 컴포넌트가 최고 실천 사례와 표준을 준수하도록 설계되었다.
- **맞춤화**: 컴포넌트는 매우 맞춤화 가능하여 특정 디자인 요구에 맞게 손쉽게 조정할 수 있다.
- **성능**: 성능을 위해 설계된 Radix UI 컴포넌트는 빠른 로드 시간과 효율적인 작동을 보장한다.
- **개발자 친화적**: 최소한의 설정과 직관적인 API로 Radix UI는 매우 개발자 친화적이다.
- **일관된 업데이트**: 정기적인 업데이트와 활발한 커뮤니티 기여로 최신 웹 표준에 맞춰 지속적으로 발전중.
- **포괄적인 문서화**: Radix UI는 상세한 가이드, 예제, 컴포넌트를 테스트할 수 있는 플레이그라운드를 제공하여 빠른 도입과 문제 해결 돕는다.

간단히 말해서 Radix를 사용하는 목적으로는 접근성과 성능 가져가면서 평소 개발자가 직접 구현해야하던 주요 기능들을 Headless UI 구조로 API를 제공해주는 디자인 시스템 라이브러리라고 볼 수 있다.

## createContext

Radix는 리액트를 기반으로 작성되었으며, Headless UI의 API를 제공하기위해 컴포넌트 별로 context를 감싸 기능을 사용하게 된다. 그중에서 core 컴포넌트 중 하나인 `createContext`를 살펴보자.

## 폴더 구조

기본적으로 라이브러리답게 대부분 barrel exports pattern으로 구성되어 있다. 하나의 기능을 하는 폴더 아래, export 되는 컴포넌트와 그것을 import하여 관리하는 index 파일 구조를 가진다.

```
ㄴ src
  ㄴ createContext.ts
  ㄴ index.ts
```

## index.ts

먼저 index.ts에선 createContext에서 context와 scope를 가져오고 해당 인터페이스를 동일 파일에서 가져온다.

```ts
export { createContext, createContextScope } from "./createContext";
export type { CreateScope, Scope } from "./createContext";
```

## createContext.ts

여기선 주요한 함수로 `createContext, createContextScope, composeContextScopes`가 선언되며 실제 사용되는 함수는 `createContext, createContextScope`이 된다.

### createContext

```ts
function createContext<ContextValueType extends object | null>(
  rootComponentName: string,
  defaultContext?: ContextValueType
) {
  const Context = React.createContext<ContextValueType | undefined>(
    defaultContext
  );

  const Provider: React.FC<ContextValueType & { children: React.ReactNode }> = (
    props
  ) => {
    const { children, ...context } = props;
    // Only re-memoize when prop values change
    // eslint-disable-next-line react-hooks/exhaustive-deps
    const value = React.useMemo(
      () => context,
      Object.values(context)
    ) as ContextValueType;
    return <Context.Provider value={value}>{children}</Context.Provider>;
  };

  Provider.displayName = rootComponentName + "Provider";

  function useContext(consumerName: string) {
    const context = React.useContext(Context);
    if (context) return context;
    if (defaultContext !== undefined) return defaultContext;
    // if a defaultContext wasn't specified, it's a required context.
    throw new Error(
      `\`${consumerName}\` must be used within \`${rootComponentName}\``
    );
  }

  return [Provider, useContext] as const;
}
```

구조는 react의 context api를 활용한 모습니다. 이때, `Provider`와 `useContext`를 직접 생성하여 return 하는 모습이 보이는데 하나의 커스텀 훅을 반환하는 모습을 볼 수 있다.

`Provider`에선 childe과 Context.Provider에 적용한 value를 props으로 주입하여 반환하는 모습을 보이고, `useContext`에선 단순히 context를 리턴하거나 defaultContext가 존재할시 defaultContext를 없다면 에러를 던지는 단순한 구조로 되어있다.

위 방식으로 보아 radix에서 전반적으로 createContext 훅을 커스텀 훅으로 생성하여 공통 인터페이스를 제공해주어, 새로 생성될 radix-ui에서도 동일하게 적용 가능할 수 있도록 제공하는 모습이 보인다.

### createContextScope

위 createContext와 달리 여기선 Scope에 관련된 타입과 함수가 정의되어있다. 대부분의 radix-ui에선 createContextScope를 사용하고 있으며, 어느 scope까지 context를 적용할지가 중요하기 때문에, 이와 같이 구성되어있다.

```ts
type Scope<C = any> = { [scopeName: string]: React.Context<C>[] } | undefined;
type ScopeHook = (scope: Scope) => { [__scopeProp: string]: Scope };
interface CreateScope {
  scopeName: string;
  (): ScopeHook;
}

function createContextScope(
  scopeName: string,
  createContextScopeDeps: CreateScope[] = []
) {
  let defaultContexts: any[] = [];

  /* -----------------------------------------------------------------------------------------------
   * createContext
   * ---------------------------------------------------------------------------------------------*/

  function createContext<ContextValueType extends object | null>(
    rootComponentName: string,
    defaultContext?: ContextValueType
  ) {
    const BaseContext = React.createContext<ContextValueType | undefined>(
      defaultContext
    );
    const index = defaultContexts.length;
    defaultContexts = [...defaultContexts, defaultContext];

    const Provider: React.FC<
      ContextValueType & {
        scope: Scope<ContextValueType>;
        children: React.ReactNode;
      }
    > = (props) => {
      const { scope, children, ...context } = props;
      const Context = scope?.[scopeName]?.[index] || BaseContext;
      // Only re-memoize when prop values change
      // eslint-disable-next-line react-hooks/exhaustive-deps
      const value = React.useMemo(
        () => context,
        Object.values(context)
      ) as ContextValueType;
      return <Context.Provider value={value}>{children}</Context.Provider>;
    };

    Provider.displayName = rootComponentName + "Provider";

    function useContext(
      consumerName: string,
      scope: Scope<ContextValueType | undefined>
    ) {
      const Context = scope?.[scopeName]?.[index] || BaseContext;
      const context = React.useContext(Context);
      if (context) return context;
      if (defaultContext !== undefined) return defaultContext;
      // if a defaultContext wasn't specified, it's a required context.
      throw new Error(
        `\`${consumerName}\` must be used within \`${rootComponentName}\``
      );
    }

    return [Provider, useContext] as const;
  }

  /* -----------------------------------------------------------------------------------------------
   * createScope
   * ---------------------------------------------------------------------------------------------*/

  const createScope: CreateScope = () => {
    const scopeContexts = defaultContexts.map((defaultContext) => {
      return React.createContext(defaultContext);
    });
    return function useScope(scope: Scope) {
      const contexts = scope?.[scopeName] || scopeContexts;
      return React.useMemo(
        () => ({
          [`__scope${scopeName}`]: { ...scope, [scopeName]: contexts },
        }),
        [scope, contexts]
      );
    };
  };

  createScope.scopeName = scopeName;
  return [
    createContext,
    composeContextScopes(createScope, ...createContextScopeDeps),
  ] as const;
}
```

일반 createContext와 달리 `Provider`와 `useContext`에서 scope를 받고 있다

```ts
type Scope<C = any> = { [scopeName: string]: React.Context<C>[] } | undefined;
```

scope 타입을 보아 어떤 scope의 context인지 구분하기위해 scopeName을 key로 가지고 각각의 context를 하나의 context로 관리하는 모습이다.

따라서 Provider에선 `const Context = scope?.[scopeName]?.[index] || BaseContext;`와 같이 해당 Provider에 정의된 scope의 정보를 가져오기위해 scopeName에서 `defaultContexts.length`를 받는 index 값. 즉, 현재 defaultContexts의 총 갯수 만큼의 context를 모두 불러오거나 없다면 초기 상태의 BaseContext를 받아 Context.Provider를 반환하고 useContext에서는 위에서 정의한 createContext와 동일하게 `React.useContext(Context)`를 주입받아 Provider와 useContext를 반환하는 커스텀 훅을 생성한다.

createScope에선 아래 composeContextScopes에서 넘길 scope를 생선하는 함수이다. 여기선 BaseContext가 아니라 scopeContext가 선언된만큼 createContext하여 반환 받고 scope와 context를 의존하여 최적화 를 진행한 useScope 훅을 반환한다. 이렇게 작업된 createScope는 `composeContextScopes`의 첫번째 매개변수로 넘어간다.

### composeContextScopes

```ts
function composeContextScopes(...scopes: CreateScope[]) {
  const baseScope = scopes[0];
  if (scopes.length === 1) return baseScope;

  const createScope: CreateScope = () => {
    const scopeHooks = scopes.map((createScope) => ({
      useScope: createScope(),
      scopeName: createScope.scopeName,
    }));

    return function useComposedScopes(overrideScopes) {
      const nextScopes = scopeHooks.reduce(
        (nextScopes, { useScope, scopeName }) => {
          // We are calling a hook inside a callback which React warns against to avoid inconsistent
          // renders, however, scoping doesn't have render side effects so we ignore the rule.
          // eslint-disable-next-line react-hooks/rules-of-hooks
          const scopeProps = useScope(overrideScopes);
          const currentScope = scopeProps[`__scope${scopeName}`];
          return { ...nextScopes, ...currentScope };
        },
        {}
      );

      return React.useMemo(
        () => ({ [`__scope${baseScope.scopeName}`]: nextScopes }),
        [nextScopes]
      );
    };
  };

  createScope.scopeName = baseScope.scopeName;
  return createScope;
}
```

여기선 위에 scope별로 createContext된 scopes를 받아 순회하여 `createScope()`를 호출해서 createContext를 진행하고, `useComposedScopes`를 통해서 위에서 생성된 scopeHooks를 순회하여 오버라이드할 스코프를 다시 선언해 다시한번 최적화하여 `__scopeProp`를 업데이트한다. 이때 변경할 스코프만 적용하기위해 `__scope`의 이름별로 key를 정의해서 반환하는 구조로 되어있다.

## 정리

정리해보면 대부분의 radix-ui에선 `createContextScope`훅을 이용하여 context를 관리하고 있고, 무수히 많은 context는 `createContextScope`를 통해 scope를 지정한 context들만 지엽적으로 관리가 된다. 예를 들면 아래 Menu.tsx를 보자.

```ts
// Menu.tsx
const MENU_NAME = "Menu";
const [createMenuContext, createMenuScope] = createContextScope(MENU_NAME, [
  createCollectionScope,
  createPopperScope,
  createRovingFocusGroupScope,
]);
const usePopperScope = createPopperScope();
const useRovingFocusGroupScope = createRovingFocusGroupScope();

type MenuContextValue = {
  open: boolean;
  onOpenChange(open: boolean): void;
  content: MenuContentElement | null;
  onContentChange(content: MenuContentElement | null): void;
};

const [MenuProvider, useMenuContext] =
  createMenuContext<MenuContextValue>(MENU_NAME);

type MenuRootContextValue = {
  onClose(): void;
  isUsingKeyboardRef: React.RefObject<boolean>;
  dir: Direction;
  modal: boolean;
};

const [MenuRootProvider, useMenuRootContext] =
  createMenuContext<MenuRootContextValue>(MENU_NAME);
```

여기서 `createContextScope`를 호출하여 첫번째 인자로 MENU_NAME를 주입하여 현재 스코프의 context를 정의했다. 그 다음 두번째 인자로 하위 스코프의 컨텍스를 주입하기위해 MENU_NAME 스코프에 필요한 스코프의 컨텍스트를 사용하도록 한다.

```ts
const [MenuProvider, useMenuContext] =
  createMenuContext<MenuContextValue>(MENU_NAME);
const [MenuRootProvider, useMenuRootContext] =
  createMenuContext<MenuRootContextValue>(MENU_NAME);
```

따라서 위와 같은 구조를 이용하여 context를 지엽적으로 사용하여 각각의 context들을 규격화하여 관리할수 있도록 작업되어있다.

Headless UI 특성상 context와 같이 각 컴포넌트에 필요한 API를 어떻게 주입하고 관리할 수 있는지 이해할 수 있게 되었다.

## 출처

- [createContext.ts](https://github.com/radix-ui/primitives/blob/main/packages/react/context/src/createContext.tsx)
- [Menu.tsx](https://github.com/radix-ui/primitives/blob/main/packages/react/menu/src/Menu.tsx)
