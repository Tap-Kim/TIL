# Radix-UI(2) - React 내장 Hooks의 커스텀 코드 분석과 Primitive 구조

Radix에선 React의 내장 훅을 커스텀하여 사용하는 몇가지가 존재하는데, 이번에는 그중 `useLayoutEffect, useId, useCallbackRef`을 가지고 분석하고자한다.

이번 장에도 마찬가지로 `Menu` 컴포넌트에서 사용되는 기능을 위주로 조사했다.

## useId

[useId](https://react.dev/reference/react/useId)는 react에서 접근성 속성으로 전달하는 유니크한 ID를 생성을 위한 훅이다. 아무래도 18 이전 버전을 대응하기 위해 자체적으로 커스텀해서 작성한 것으로 보인다.

```ts
import * as React from "react";
import { useLayoutEffect } from "@radix-ui/react-use-layout-effect";

// We spaces with `.trim().toString()` to prevent bundlers from trying to `import { useId } from 'react';`
const useReactId =
  (React as any)[" useId ".trim().toString()] || (() => undefined);
let count = 0;

function useId(deterministicId?: string): string {
  const [id, setId] = React.useState<string | undefined>(useReactId());
  // React versions older than 18 will have client-side ids only.
  useLayoutEffect(() => {
    if (!deterministicId) setId((reactId) => reactId ?? String(count++));
  }, [deterministicId]);
  return deterministicId || (id ? `radix-${id}` : "");
}

export { useId };
```

주석에도 설명했듯 React에서 "useId" 항목을 찾아 그대로 사용할 것 인지, 없다면 noop을 반환하여 초기화하는 모습을 볼 수 있다.

다음으로 useId 훅을 직접 생성하여 id를 상태관리하는 모습을 볼 수 있는데, 특이한점이 `useLayoutEffect` 또한 커스텀한 훅으로 사용하게된다. 내부 로직에서 useReactId가 없다면 React에서 `useId`가 제공되지 않는 버전을 사용하고 있다는 뜻이고, 이때 전역에서 관리되는 count 변수를 카운팅하여 유니크한 id 값을 제어하는 모습이 보인다. 반환값으로 직접 지정한 id `deterministicId`를 사용하고 없다면 id 상태를 기준으로 radix용 id를 생성하고, id가 정의되지 않았다면 공백을 반환하여 버전 대응에 용이한 radix 용 `useId` 훅의 구성을 살펴볼 수 있다.

## useLayoutEffect

`useLayoutEffect` 훅 또한 커스텀하게 사용하는데 구조를 보자.

```ts
import * as React from "react";

/**
 * On the server, React emits a warning when calling `useLayoutEffect`.
 * This is because neither `useLayoutEffect` nor `useEffect` run on the server.
 * We use this safe version which suppresses the warning by replacing it with a noop on the server.
 *
 * See: https://reactjs.org/docs/hooks-reference.html#uselayouteffect
 */
const useLayoutEffect = Boolean(globalThis?.document)
  ? React.useLayoutEffect
  : () => {};

export { useLayoutEffect };
```

주석의 설명을 보면 서버 사이드에서 발생할 문제를 대응하기위해 noop 함수(`() => {}`)를 사용하는 것을 볼 수 있다. 먼저 node.js 환경에서의 글로벌 변수인 `globalThis`를 사용하고 내부에 document 여부를 확인을 하여 DOM이 확인되면 클라이언트 사이드 환경이므로 기존 `React.useLayoutEffect를` 반환 그렇지 않으면 noop을 반환하여 에러 상황을 억제하여 안전하게 사용하도록 구성한 모습이다.

## useCallbackRef

이 훅은 이름에서도 추론이 가능하듯이 callback을 Ref로 사용하여 렌더링 제어를 하기 위한 용도의 훅을 추론할 수 있다.

```ts
import * as React from "react";

/**
 * A custom hook that converts a callback to a ref to avoid triggering re-renders when passed as a
 * prop or avoid re-executing effects when passed as a dependency
 */
function useCallbackRef<T extends (...args: any[]) => any>(
  callback: T | undefined
): T {
  const callbackRef = React.useRef(callback);

  React.useEffect(() => {
    callbackRef.current = callback;
  });

  // https://github.com/facebook/react/issues/19240
  return React.useMemo(
    () => ((...args) => callbackRef.current?.(...args)) as T,
    []
  );
}

export { useCallbackRef };
```

주석의 설명을 보면 callback을 참조로 변환해서 전달될 때 재렌더링이 트리거 되지 않도록 하는 구조이다. 의존성으로 통과된다면 재실행의 상황을 실행하거나 피할수 있도록 설계되어있다.
callback 자체를 ref로 할당하고, effect에서 항상 새로운 callback이 내려올때마다 할당하는 모습을 볼 수 있다. 마지막 반환시 useMemo를 사용하여 callbackRef를 최적화하여 리렌더링을 피하도록하는 모습이다.

```tsx
// Menu.tsx
const Menu: React.FC<MenuProps> = (props: ScopedProps<MenuProps>) => {
  const { __scopeMenu, open = false, children, dir, onOpenChange, modal = true } = props;
  ...
  const handleOpenChange = useCallbackRef(onOpenChange);
```

사용되는 코드를 보면, props으로 넘어온 `onOpenChange`를 매번 재할당하지 않고 최적화한 함수를 할당하여, 렌더링 최적화를 수행한 콜백함수를 사용하는 모습을 볼 수 있다.

## Portal

여기서는 `ReactDOM.createPortal`을 사용한 `Portal` 컴포넌트를 볼 수 있다.

```ts
import * as React from "react";
import ReactDOM from "react-dom";
import { Primitive } from "@radix-ui/react-primitive";
import { useLayoutEffect } from "@radix-ui/react-use-layout-effect";

/* -------------------------------------------------------------------------------------------------
 * Portal
 * -----------------------------------------------------------------------------------------------*/

const PORTAL_NAME = "Portal";

type PortalElement = React.ElementRef<typeof Primitive.div>;
type PrimitiveDivProps = React.ComponentPropsWithoutRef<typeof Primitive.div>;
interface PortalProps extends PrimitiveDivProps {
  /**
   * An optional container where the portaled content should be appended.
   */
  container?: Element | DocumentFragment | null;
}

const Portal = React.forwardRef<PortalElement, PortalProps>(
  (props, forwardedRef) => {
    const { container: containerProp, ...portalProps } = props;
    const [mounted, setMounted] = React.useState(false);
    useLayoutEffect(() => setMounted(true), []);
    const container = containerProp || (mounted && globalThis?.document?.body);
    return container
      ? ReactDOM.createPortal(
          <Primitive.div {...portalProps} ref={forwardedRef} />,
          container
        )
      : null;
  }
);

Portal.displayName = PORTAL_NAME;

/* -----------------------------------------------------------------------------------------------*/

const Root = Portal;

export { Portal, Root };
export type { PortalProps };
```

여기서는 `react-primitive`에서 정의한, div 태그 정보를 가진 `Primitive`컴포넌트를 사용하여 Portal 컴포넌트를 구성하는 간단한 모습을 보인다. 또한 고차함수로 `React.forwardRef`으로 사용하여, Portal 컴포넌트의 ref를 받을 수 있도록 유연한 구성을 가지고 있따.

## primitive와 react-primitive

### primitive

radix에선 `primitive`와 `react-primitive`가 존재하는데, primitive는 특별한 코드는 없고, `composeEventHandlers` 함수를 반환하는데 오리지널 이벤트 핸들러와 radix용 이벤트 핸들러를 인자로 받아서, 두가지 모두를 실행할 수 있도록 합성 함수로 구성하여, 기존 html에서 제공하는 프로퍼티 메서드를 그대로 적용하면서, radix에서 정의한 이벤트 핸들러를 같이 사용하도록 반환해주는 형태이다.

```ts
function composeEventHandlers<E>(
  originalEventHandler?: (event: E) => void,
  ourEventHandler?: (event: E) => void,
  { checkForDefaultPrevented = true } = {}
) {
  return function handleEvent(event: E) {
    originalEventHandler?.(event);

    if (
      checkForDefaultPrevented === false ||
      !(event as unknown as Event).defaultPrevented
    ) {
      return ourEventHandler?.(event);
    }
  };
}

export { composeEventHandlers };
```

### react-primitive

react-primitive에선 독특한 구성으로 되어있는데, `NODES`라는 변수를 선언하여, radix-ui에서 사용되는 태그를 몇가지 추려내어, Primitive하게 사용한 노드들을 컴포넌트와해서 관리하도록 구성되어 있다.

```ts
import * as React from "react";
import * as ReactDOM from "react-dom";
import { Slot } from "@radix-ui/react-slot";

const NODES = [
  "a",
  "button",
  "div",
  "form",
  "h2",
  "h3",
  "img",
  "input",
  "label",
  "li",
  "nav",
  "ol",
  "p",
  "span",
  "svg",
  "ul",
] as const;

type Primitives = {
  [E in (typeof NODES)[number]]: PrimitiveForwardRefComponent<E>;
};
type PrimitivePropsWithRef<E extends React.ElementType> =
  React.ComponentPropsWithRef<E> & {
    asChild?: boolean;
  };

interface PrimitiveForwardRefComponent<E extends React.ElementType>
  extends React.ForwardRefExoticComponent<PrimitivePropsWithRef<E>> {}

/* -------------------------------------------------------------------------------------------------
 * Primitive
 * -----------------------------------------------------------------------------------------------*/

const Primitive = NODES.reduce((primitive, node) => {
  const Node = React.forwardRef(
    (props: PrimitivePropsWithRef<typeof node>, forwardedRef: any) => {
      const { asChild, ...primitiveProps } = props;
      const Comp: any = asChild ? Slot : node;

      if (typeof window !== "undefined") {
        (window as any)[Symbol.for("radix-ui")] = true;
      }

      return <Comp {...primitiveProps} ref={forwardedRef} />;
    }
  );

  Node.displayName = `Primitive.${node}`;

  return { ...primitive, [node]: Node };
}, {} as Primitives);

const Root = Primitive;
export type { PrimitivePropsWithRef };

/* -------------------------------------------------------------------------------------------------
 * Utils
 * -----------------------------------------------------------------------------------------------*/

/**
 * Flush custom event dispatch
 * https://github.com/radix-ui/primitives/pull/1378
 *
 * React batches *all* event handlers since version 18, this introduces certain considerations when using custom event types.
 * **/

function dispatchDiscreteCustomEvent<E extends CustomEvent>(
  target: E["target"],
  event: E
) {
  if (target) ReactDOM.flushSync(() => target.dispatchEvent(event));
}
export { Primitive, Root, dispatchDiscreteCustomEvent };
```

`PrimitiveForwardRefComponent` 인터페이스를 확인하면 그 이유를 알 수 있다. `Primitives` 타입에서 각 노드의 타입을 `PrimitiveForwardRefComponent`을 받도록 되어 있고, 이는 `React.ForwardRefExoticComponent`를 상속하는 모습을 볼 수 있다. 이는, `PrimitivePropsWithRef`타입에서 `asChild`라는 props를 받는 것을 알 수 있는데, NODES에서만 정의한 태그를 사용하게되면, 웹 접근성과 role의 역할, 기능 등이 이미 정해진 상태만 사용하게 된다. 그렇지않고 사용자가 원하는 태그를 내려받기 위해 `asChild`를 받아 태그를 얼마든지 확장시키는 기능을 제공하게되는데 `React.ForwardRefExoticComponent` 타입을 상속하여 컴포넌트의 다형성을 가져갈 수 있도록 했다.

asChild시 `Slot`을 받게 되는데 대충 Slot에서는 해당 ReactNode(children)를 받아 노드를 클론하여 반환하는 컴포넌트이고, 그렇지 않다면 NODES에 정의한 node를 반환하여 컴포넌트를 구성하는 것을 볼 수 있다.

```ts
if (typeof window !== "undefined") {
  (window as any)[Symbol.for("radix-ui")] = true;
}
```

중간에 위 조건문을 볼 수 있는데, 클라이언트 사이드일때, window 객체에 radix-ui에 대한 심볼을 낙인하여, 현재 window에선 radix-ui를 사용하고 있음을 심볼링한다.

마지막 `dispatchDiscreteCustomEvent`을 보게 되면 18버전 부터는 모든 배치는 이벤트 핸들러를 반응시키려면 사용자 지정 이벤트 유형을 사용시 고려해야할 사항들이 있어 자체적으로 커스텀한 함수이다.

리액트에선 내부적으로 다음과 같은 순서로 우선순위를 지정한다.[참고](https://github.com/facebook/react/blob/a8a4742f1c54493df00da648a3f9d26e3db9c8b5/packages/react-dom/src/events/ReactDOMEventListener.js#L294-L350)

- discrete
- continuous
- default

이러한 이벤트는 이벤트 내 업데이트가 즉시 적용되어 `discrete`는 중요한 차이를 가지게 된다. 하지만 내부적으로 감지되는 방식으로 인해 사용자 지정 이벤트 유형의 우선순위를 추론이 어렵다. 따라서 사용자 지정 이벤트의 업데이트가 예기치않게 배치처리 될 수 있다. 또 다른 `discrete` 이벤트에 의해 dispatch 된다.

때문에 사용자 지정 이벤트의 업데이트가 예측 가능하도록 적용하려면 배치를 `수동으로 flush`해야한다. 이 유틸 함수는 `discrete` 이벤트내에서 커스텀 이벤트를 발송시 사용해야한다. 이미 발송된 이벤트나 `discrete`가 아닌 이벤트 내에서 사용자 지정 유형을 발송시 필요하지 않다.

예를 들면

```ts
 // dispatching a known click 👎
 target.dispatchEvent(new Event(‘click’))

 // dispatching a custom type within a non-discrete event 👎
 onScroll={(event) => event.target.dispatchEvent(new CustomEvent(‘customType’))}

 // dispatching a custom type within a `discrete` event 👍
 onPointerDown={(event) => dispatchDiscreteCustomEvent(event.target, new CustomEvent(‘customType’))}
```

> 참고: 리액트는 `focus`, `focusin`, `focusout` 이벤트를 `discrete`으로 분류하지만 사용하는 것은 권장하지는 않는다. 이 유틸은 렌더링 중에 해당 핸들러를 암묵적으로 호출할 위험이 있따. 예를 들면, 포커스가 해제된 상태에서 구성 요소에 있을 때 또는 포커스를 관리시.

즉, `if (target) ReactDOM.flushSync(() => target.dispatchEvent(event));` 코드를 확인하면, `dispatchDiscreteCustomEvent` 호출시 target이 존재한다면(이벤트 핸들러가 있다면) `ReactDOM.flushSync`를 통해 현재 넘어온 타겟 이벤트를 동기적으로 배치를 수행하도록 구성한다.

## 정리

`useLayoutEffect, useId, useCallbackRef`와 같이 이미 존재하거나, 또는 그와 유사하게 radix 내부에서 문제가 발생하지 않도록 커스텀 훅을 구성해서 사용하는 모습이 꽤 흥미롭다. 또한 radix API의 기초가되는 Primitive와 react-primitive를 보면서 왜 radix를 사용할 때 해당 프로퍼티를 쓰거나, 지정하지 않은 API의 이벤트들이 유기적으로 연결되어 실행되는지 이해할 수 있었다. 특히 NODES를 미리 지정하여, ReactNode의 다형성을 가져갈 수 있도록 asChild를 사용하여 Slot을 반환하는 모습이 인상 깊었고, 권장하진 않지만, 리액트의 내부 동작 우선순위를 고려하여 동기적 배치를 구성할 수 있는 함수를 구성한 모습도 흥미롭다.

## 출처

- [createContext.ts](https://github.com/radix-ui/primitives/blob/main/packages/react/context/src/createContext.tsx)
- [Menu.tsx](https://github.com/radix-ui/primitives/blob/main/packages/react/menu/src/Menu.tsx)
