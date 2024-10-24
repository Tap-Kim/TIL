# Radix-UI(3) - Radix-UI의 조합 Popper 컴포넌트 분석

Radix에선 Dialog, Alert, Menu 등 공통적으로 사용되는 `Popper` 컴포넌트가 있다. 이 컴포넌트는 트리거 컴포넌트를 기준으로 컴포넌트 위에 띄울 모달과 같은 팝오버와 같은 형태를 제공하는데, `@floating-ui/react-dom`을 가져와 해당 영역을 다루고 있다. 지금 볼 컴포넌트는 앞서 공부했던, 커스텀 훅과 컴포넌트를 조합해서 어떻게 사용되는지 볼 수 있게된다.

## floating-ui

먼저 [floating-ui](https://floating-ui.com/docs/getting-started)는 절대 위치를 통해 주어진 참조 요소 옆에 고정할 수 있는 일종의 위치 지정 기능 툴킷을 제공한다. 팝오버의 기본 기능이고, 절대 위치 지정이기 때문에 위치 참조의 위치에 따라 원치 않은 오버플로우를 많아 뷰포트와의 충돌의 피하기 위한 기능도 제공한다.

또한 고도로 모듈화되어 설계되어 트리 셰이킹이 잘되고, 번들 크기가 조정된다. 게다가 유연하고 해킹이 가능하도록 설계되어 있어 어떤 프레임워크에 사용해도 문제가 없도록 설계되어 있어 타 라이브러리와 친화적인 라이브러기 때문에 radix에서 채택한 것이라고 생각한다.

```ts
// Popper에서 사용되는 "@floating-ui/react-dom 기능들
import {
  useFloating,
  autoUpdate,
  offset,
  shift,
  limitShift,
  hide,
  arrow as floatingUIarrow,
  flip,
  size,
} from "@floating-ui/react-dom";
import type { Placement, Middleware } from "@floating-ui/react-dom";
```

## Popper

이제 Popper에 관한 코드 구성을 살펴보겠다.

```tsx
const POPPER_NAME = "Popper";

type ScopedProps<P> = P & { __scopePopper?: Scope };
const [createPopperContext, createPopperScope] =
  createContextScope(POPPER_NAME);

type PopperContextValue = {
  anchor: Measurable | null;
  onAnchorChange(anchor: Measurable | null): void;
};
const [PopperProvider, usePopperContext] =
  createPopperContext<PopperContextValue>(POPPER_NAME);

interface PopperProps {
  children?: React.ReactNode;
}
const Popper: React.FC<PopperProps> = (props: ScopedProps<PopperProps>) => {
  const { __scopePopper, children } = props;
  const [anchor, setAnchor] = React.useState<Measurable | null>(null);
  return (
    <PopperProvider
      scope={__scopePopper}
      anchor={anchor}
      onAnchorChange={setAnchor}
    >
      {children}
    </PopperProvider>
  );
};

Popper.displayName = POPPER_NAME;
```

`Menu.tsx`에서도 보았듯이 앞서 파악했던 것들을 기반으로 컴포넌트가 제공된다. `createContextScope`를 통해 popper의 context와 scope를 반환받고, POPPER_NAME을 주어 고유한 컨텍스트를 반환받도록 한다. 다음으로 `createPopperContext`를 지엽적으로 선언하여 popper 전용 Provider와 Context를 이용해 Popper 컴포넌트를 구성한다. 이때 `anchor`가 `@floating-ui`의 타입으로 가져오는 것으로 보아, 위치 지정을 위한 용도의 상태값이라는 것을 파악 할 수 있다.

## PopperAnchor

`PopperAnchor`은 Popper를 호출하는 트리거 컴포넌트를 감싸는 컴포넌트이다. 즉, 트리거 컴포넌트위 위치를 잡기 위한 용도로 추측할 수 있다.

```tsx
const ANCHOR_NAME = "PopperAnchor";

type PopperAnchorElement = React.ElementRef<typeof Primitive.div>;
type PrimitiveDivProps = React.ComponentPropsWithoutRef<typeof Primitive.div>;
interface PopperAnchorProps extends PrimitiveDivProps {
  virtualRef?: React.RefObject<Measurable>;
}

const PopperAnchor = React.forwardRef<PopperAnchorElement, PopperAnchorProps>(
  (props: ScopedProps<PopperAnchorProps>, forwardedRef) => {
    const { __scopePopper, virtualRef, ...anchorProps } = props;
    const context = usePopperContext(ANCHOR_NAME, __scopePopper);
    const ref = React.useRef<PopperAnchorElement>(null);
    const composedRefs = useComposedRefs(forwardedRef, ref);

    React.useEffect(() => {
      // Consumer can anchor the popper to something that isn't
      // a DOM node e.g. pointer position, so we override the
      // `anchorRef` with their virtual ref in this case.
      context.onAnchorChange(virtualRef?.current || ref.current);
    });

    return virtualRef ? null : (
      <Primitive.div {...anchorProps} ref={composedRefs} />
    );
  }
);

PopperAnchor.displayName = ANCHOR_NAME;
```

주석에 따르면 DOM 노드(포인터의 위치)에 따라서 재정의하고, 이 경우 가상 참조로 `anchorRef`를 사용한다고 명시되어있다.

여기서 재밌는 구간이 `usePopperContext`를 할당한 context의 `onAnchorChange` 핸들러를 호출하고 있다. 인자 값으로 가상 참조(virtualRef) 값 또는 현재 ref를 넘겨주고 있는데, 앞서 공부 했듯이 `usePopperContext`는 위 `createPopperContext`에서 가져온 컨텍스트. 즉, createContext의 `composeContextScopes`이다.

이전에 살펴보았을 때 `composeContextScopes(createScope, ...createContextScopeDeps)` `composeContextScopes`는 생성된 scope와 생성된 scope의 컨텍스트 의존성들을 받는다. 즉, `virtualRef?.current || ref.current`를 넘긴다는 것은 해당 컨텍스트의 scope 영역을 변경하고, 의존성까지 변경한다는 의미이다. 이렇게되면, context의 scope 영역은 `PopperAnchor`를 기준으로 가져가게 될 것이고, context의 범위가 변경 될 것이다.

다음으로 살펴볼 것이 `useComposedRefs` 훅인데, 간단히 말하면, PopperAnchor와 상위에서 가져온 ref를 합성해서 `PopperAnchor`의 ref 값으로 참조하는 의미이다.

마지막 반환을 보면 가상 참조가 되어 있다면 null을 그렇지 않다면 PopperAnchor의 참조를 셋팅하고 반환한다.

## composeRefs

위에서 합성 참조를 뜻하는 `composeRefs`를 살펴보자.

```ts
import * as React from "react";

type PossibleRef<T> = React.Ref<T> | undefined;

/**
 * Set a given ref to a given value
 * This utility takes care of different types of refs: callback refs and RefObject(s)
 */
function setRef<T>(ref: PossibleRef<T>, value: T) {
  if (typeof ref === "function") {
    ref(value);
  } else if (ref !== null && ref !== undefined) {
    (ref as React.MutableRefObject<T>).current = value;
  }
}

/**
 * A utility to compose multiple refs together
 * Accepts callback refs and RefObject(s)
 */
function composeRefs<T>(...refs: PossibleRef<T>[]) {
  return (node: T) => refs.forEach((ref) => setRef(ref, node));
}

/**
 * A custom hook that composes multiple refs
 * Accepts callback refs and RefObject(s)
 */
function useComposedRefs<T>(...refs: PossibleRef<T>[]) {
  // eslint-disable-next-line react-hooks/exhaustive-deps
  return React.useCallback(composeRefs(...refs), refs);
}

export { composeRefs, useComposedRefs };
```

`useComposedRefs`를 살펴보면 여러 개의 참조를 구성하는 커스텀 훅이라는 것을 알수 있고, 콜백 참조와 RefObjects(s)를 허용한다. 이또한 useCallback으로 훅을 최적화하고 있다.

`composeRefs`에선 참조 정보들을 순회하여 `setRef`를 호출하는데, `function`이라면 함수형태로 ref에 노드를 인자로 넘기고, 변수라면 노드를 그대로 할당하는 단순한 형태를 가진다. 함수와 변수 할당 두가지 케이스에 대해 ref를 대응하는 모습을 볼 수 있다.

## PopperContent

Popper의 본체라 할 수 있는 `PopperContent`를 살펴보자. 코드가 너무 길어 분리해서 확인해보았다.

### 타입 선언

선언된 타입을 먼저 보면서 파악해보면, `PopperContentContextValue`를 보아 content의 위치 정보와 arrow 정보를 통해 어느 위치에 표현할 것이지 정하는 context가 있다.

`PopperContentElement`의 타입을 보면 참조형 Element이며 역시 Primitive.div를 기본 값으로 가지는 요소로 확인된다.

마지막으로 `PopperContentProps`를 보아, `PrimitiveDivProps`를 상속하고, `PopperContent`의 기본 값들을 정의해준다.

```tsx
const CONTENT_NAME = "PopperContent";

type PopperContentContextValue = {
  placedSide: Side;
  onArrowChange(arrow: HTMLSpanElement | null): void;
  arrowX?: number;
  arrowY?: number;
  shouldHideArrow: boolean;
};

const [PopperContentProvider, useContentContext] =
  createPopperContext<PopperContentContextValue>(CONTENT_NAME);

type Boundary = Element | null;

type PopperContentElement = React.ElementRef<typeof Primitive.div>;
interface PopperContentProps extends PrimitiveDivProps {
  side?: Side;
  sideOffset?: number;
  align?: Align;
  alignOffset?: number;
  arrowPadding?: number;
  avoidCollisions?: boolean;
  collisionBoundary?: Boundary | Boundary[];
  collisionPadding?: number | Partial<Record<Side, number>>;
  sticky?: "partial" | "always";
  hideWhenDetached?: boolean;
  updatePositionStrategy?: "optimized" | "always";
  onPlaced?: () => void;
}
```

### context, refs, arrow

`usePopperContext`, `useComposedRefs`를 선언하여 content용 context와 content들의 노드의 참조를 합성하여 선언했다.

popper의 anchor를 띄울 arrow의 크기와 상태를 선언 및 초기화한다.

```ts
const context = usePopperContext(CONTENT_NAME, __scopePopper);

const [content, setContent] = React.useState<HTMLDivElement | null>(null);
const composedRefs = useComposedRefs(forwardedRef, (node) => setContent(node));

const [arrow, setArrow] = React.useState<HTMLSpanElement | null>(null);
const arrowSize = useSize(arrow);
const arrowWidth = arrowSize?.width ?? 0;
const arrowHeight = arrowSize?.height ?? 0;
```

### placement, boundary

어느 위치지를 기준으로 할지 위치 정보를 규격화하여 변수 할당을 지정하고, padding을 초기화 및 커스텀할 수 있게 선언, collisionBoundary 정보를 할당받고, 명시적으로 boundary의 경계를 둘수 있는지 확인하고, `detectOverflowOptions`에 모두 할당하여, 체크할 수 있는 overflow 옵션정보를 선언한다.

```ts
const desiredPlacement = (side +
  (align !== "center" ? "-" + align : "")) as Placement;

const collisionPadding =
  typeof collisionPaddingProp === "number"
    ? collisionPaddingProp
    : { top: 0, right: 0, bottom: 0, left: 0, ...collisionPaddingProp };

const boundary = Array.isArray(collisionBoundary)
  ? collisionBoundary
  : [collisionBoundary];
const hasExplicitBoundaries = boundary.length > 0;

const detectOverflowOptions = {
  padding: collisionPadding,
  boundary: boundary.filter(isNotNull),
  // with `strategy: 'fixed'`, this is the only way to get it to respect boundaries
  altBoundary: hasExplicitBoundaries,
};
```

### useFloating

`floating-ui`의 useFloating을 사용하여, PopperContent를 띄울 초기값을 설정한다.

아래에 setProperty를 설정하는 부분이 있는데, 스타일을 직접 property로 커스텀하게 명시하여, floating 스타일을 셋팅하는 모습을 볼 수 있다.

```ts
const { refs, floatingStyles, placement, isPositioned, middlewareData } =
  useFloating({
    // default to `fixed` strategy so users don't have to pick and we also avoid focus scroll issues
    strategy: "fixed",
    placement: desiredPlacement,
    whileElementsMounted: (...args) => {
      const cleanup = autoUpdate(...args, {
        animationFrame: updatePositionStrategy === "always",
      });
      return cleanup;
    },
    elements: {
      reference: context.anchor,
    },
    middleware: [
      offset({
        mainAxis: sideOffset + arrowHeight,
        alignmentAxis: alignOffset,
      }),
      avoidCollisions &&
        shift({
          mainAxis: true,
          crossAxis: false,
          limiter: sticky === "partial" ? limitShift() : undefined,
          ...detectOverflowOptions,
        }),
      avoidCollisions && flip({ ...detectOverflowOptions }),
      size({
        ...detectOverflowOptions,
        apply: ({ elements, rects, availableWidth, availableHeight }) => {
          const { width: anchorWidth, height: anchorHeight } = rects.reference;
          const contentStyle = elements.floating.style;
          contentStyle.setProperty(
            "--radix-popper-available-width",
            `${availableWidth}px`
          );
          contentStyle.setProperty(
            "--radix-popper-available-height",
            `${availableHeight}px`
          );
          contentStyle.setProperty(
            "--radix-popper-anchor-width",
            `${anchorWidth}px`
          );
          contentStyle.setProperty(
            "--radix-popper-anchor-height",
            `${anchorHeight}px`
          );
        },
      }),
      arrow && floatingUIarrow({ element: arrow, padding: arrowPadding }),
      transformOrigin({ arrowWidth, arrowHeight }),
      hideWhenDetached &&
        hide({ strategy: "referenceHidden", ...detectOverflowOptions }),
    ],
  });

function getSideAndAlignFromPlacement(placement: Placement) {
  const [side, align = "center"] = placement.split("-");
  return [side as Side, align as Align] as const;
}

const [placedSide, placedAlign] = getSideAndAlignFromPlacement(placement);

const handlePlaced = useCallbackRef(onPlaced);
useLayoutEffect(() => {
  if (isPositioned) {
    handlePlaced?.();
  }
}, [isPositioned, handlePlaced]);
```

`useFloating`의 반환 정보중 `placement`를 가지고 side와 align 정보를 선언한다. 다음으로 `isPositioned`를 판단하여 `onPlaced`를 반는데 정확히 어떤 역활하는 하는 건지 모르겠다.

### arrow 좌표 설정

`useFloating`에서 셋팅한 middlewareData 정보(offset, size, collision의 정보를 갖고 있음)를 사용해 arrow의 위치 선언

```ts
const arrowX = middlewareData.arrow?.x;
const arrowY = middlewareData.arrow?.y;
const cannotCenterArrow = middlewareData.arrow?.centerOffset !== 0;
```

### content z-index 설정

content의 z-index를 설정하는데, 중요한 역할을 하는 로직이다. [getComputedStyle](https://developer.mozilla.org/ko/docs/Web/API/Window/getComputedStyle)를 적용하는 모습을 볼 수 있는데, `content`를 호출하게 되면, 동적으로 z-index 값을 할당하기 위해 `useLayoutEffect`에서 DOM이 페인팅 되기 전에 상태를 가져오는 모습을 볼 수 있다. `useEffect`로 가져온다면, 화면에 그려지고난뒤 z-index 값을 가져오기 때문에, `contentZIndex`를 사용하는 곳에서 값이 튀는 것을 방치하는 것을 막으려는 것으로 판단된다.

```tsx
const [contentZIndex, setContentZIndex] = React.useState<string>();
useLayoutEffect(() => {
  if (content) setContentZIndex(window.getComputedStyle(content).zIndex);
}, [content]);
```

### return 부

위에서 선언된 모든 정보를 div에 기본적으로 할당하고 있다.

먼저 미들웨어 로직을 보면, 숨기는 모습이 보이는데, 이는 콘텐츠를 숨겨야하는 상황에서 비활성화하는 모습이다.

또한 `dir`을 보면 floating-ui는 `dir` 속성을 기반으로 논리적 정렬 계산을 내부적으로한다. 따라서, reference/floating 노드를 보장하기 위해 반드시 필요한 속성이다. 이는 인라인 뿐만아니라 포팅(?)할때 계산된다.

마지막으로 `animation`은 `isPositioned`이 활성화되어 있다면, 다시말해 `PopperContent`가 배치되지 않을때, 애니메이션이 잘못 호출되는 것을 막도록 설정했다.

```tsx
<div
  ref={refs.setFloating}
  data-radix-popper-content-wrapper=""
  style={{
    ...floatingStyles,
    transform: isPositioned ? floatingStyles.transform : "translate(0, -200%)", // keep off the page when measuring
    minWidth: "max-content",
    zIndex: contentZIndex,
    ["--radix-popper-transform-origin" as any]: [
      middlewareData.transformOrigin?.x,
      middlewareData.transformOrigin?.y,
    ].join(" "),

    // hide the content if using the hide middleware and should be hidden
    // set visibility to hidden and disable pointer events so the UI behaves
    // as if the PopperContent isn't there at all
    ...(middlewareData.hide?.referenceHidden && {
      visibility: "hidden",
      pointerEvents: "none",
    }),
  }}
  // Floating UI interally calculates logical alignment based the `dir` attribute on
  // the reference/floating node, we must add this attribute here to ensure
  // this is calculated when portalled as well as inline.
  dir={props.dir}
>
  <PopperContentProvider
    scope={__scopePopper}
    placedSide={placedSide}
    onArrowChange={setArrow}
    arrowX={arrowX}
    arrowY={arrowY}
    shouldHideArrow={cannotCenterArrow}
  >
    <Primitive.div
      data-side={placedSide}
      data-align={placedAlign}
      {...contentProps}
      ref={composedRefs}
      style={{
        ...contentProps.style,
        // if the PopperContent hasn't been placed yet (not all measurements done)
        // we prevent animations so that users's animation don't kick in too early referring wrong sides
        animation: !isPositioned ? "none" : undefined,
      }}
    />
  </PopperContentProvider>
</div>
```

## 정리

나머지 컴포넌트는 크게 중요하지 않아서 정리하진 않았다. Popper는 Radix-UI에서 많이 사용되는 컴포넌트이다. 그만큼 구조가 복잡하고, 고려해야할 사항이 많다는 것을 확인할 수 있었다.

특히 floating-ui를 가져다가 사용해서, 위치에 대한 기능을 위임하여 적절히 사용한 모습이 인상 깊었고, 생각보다 Popper에서는 위치에 대한 정보를 위주로 구성한 것을 알 수 있었다. 이는 Popper 또한 다른 컴포넌트 처럼 context용도로 사용하는 목적이 크며, Popper를 가져다가 사용하는 곳에서는 비지니스 로직에 집중할 수 있도록 명확하게 포지션 작업을 하도록 구성되었다.

그중에서도 composeRef를 통해, 여러 context를 합성하여 ref를 관리하는 모습이 인상깊었다.

## 출처

- [Popper.tsx](https://github.com/radix-ui/primitives/blob/main/packages/react/popper/src/Popper.tsx#L49)
