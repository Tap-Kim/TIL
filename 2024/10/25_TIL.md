# Radix-UI(4) - Headless UI API를 조합하여 Menu 컴포넌트 살펴보기

Menu를 보기위해 지금까지 많은 코드를 조사해보았다. 이번엔 지금까지 공부했던 Radix API들을 모두 활용하여 Menu라는 Radix-UI 컴포넌트를 살펴보자. 핵심이 되는 `Menu`, `MenuContent` 정도만 확인해보자.

## Menu

초기 코드는 contextScope를 생성하여 할당하는 영역까지는 여타 컴포넌트와 동일하다. useEffect를 살펴보면, 다른 동작에 의한 사이드 이펙트를 막기위해 `addEventListener`의 capture, once 옵션을 주어, 키보프나, 포인터 이벤트시 옵션을 달아주어 Menu 컴포넌트의 이벤트를 우선 적용하는 모습을 볼 수 있다.

```tsx
/* 중략 */
const Menu: React.FC<MenuProps> = (props: ScopedProps<MenuProps>) => {
  const {
    __scopeMenu,
    open = false,
    children,
    dir,
    onOpenChange,
    modal = true,
  } = props;
  const popperScope = usePopperScope(__scopeMenu);
  const [content, setContent] = React.useState<MenuContentElement | null>(null);
  const isUsingKeyboardRef = React.useRef(false);
  const handleOpenChange = useCallbackRef(onOpenChange);
  const direction = useDirection(dir);

  React.useEffect(() => {
    // Capture phase ensures we set the boolean before any side effects execute
    // in response to the key or pointer event as they might depend on this value.
    const handleKeyDown = () => {
      isUsingKeyboardRef.current = true;
      document.addEventListener("pointerdown", handlePointer, {
        capture: true,
        once: true,
      });
      document.addEventListener("pointermove", handlePointer, {
        capture: true,
        once: true,
      });
    };
    const handlePointer = () => (isUsingKeyboardRef.current = false);
    document.addEventListener("keydown", handleKeyDown, { capture: true });
    return () => {
      document.removeEventListener("keydown", handleKeyDown, { capture: true });
      document.removeEventListener("pointerdown", handlePointer, {
        capture: true,
      });
      document.removeEventListener("pointermove", handlePointer, {
        capture: true,
      });
    };
  }, []);

  return (
    <PopperPrimitive.Root {...popperScope}>
      <MenuProvider
        scope={__scopeMenu}
        open={open}
        onOpenChange={handleOpenChange}
        content={content}
        onContentChange={setContent}
      >
        <MenuRootProvider
          scope={__scopeMenu}
          onClose={React.useCallback(
            () => handleOpenChange(false),
            [handleOpenChange]
          )}
          isUsingKeyboardRef={isUsingKeyboardRef}
          dir={direction}
          modal={modal}
        >
          {children}
        </MenuRootProvider>
      </MenuProvider>
    </PopperPrimitive.Root>
  );
};

Menu.displayName = MENU_NAME;
```

## Content

Content 영역은 이 컴포넌트의 핵임 로직인데, Menu의 구현부를 추상화하여 단계를 주어진다.

`MenuContent` -> `MenuRootContentModal` -> `MenuContentImpl`순으로 구성된다.

### MenuContent

Content의 첫번째 영역이다. 여기서는 `foreMount`는 받을 수가 있는데, 더 많은 제어를 강제로 수행할때 필요된다. 얘를 들면애니메이션 라이브러리를 활용하여 제어할때.

이제 전반적인 코드를 살펴보자.

```tsx
type ItemData = { disabled: boolean; textValue: string };
const [Collection, useCollection, createCollectionScope] = createCollection<
  MenuItemElement, // MenuContent의 Item 영역의 요소
  ItemData
>(MENU_NAME);

const CONTENT_NAME = "MenuContent";

type MenuContentContextValue = {
  onItemEnter(event: React.PointerEvent): void;
  onItemLeave(event: React.PointerEvent): void;
  onTriggerLeave(event: React.PointerEvent): void;
  searchRef: React.RefObject<string>;
  pointerGraceTimerRef: React.MutableRefObject<number>;
  onPointerGraceIntentChange(intent: GraceIntent | null): void;
};
const [MenuContentProvider, useMenuContentContext] =
  createMenuContext<MenuContentContextValue>(CONTENT_NAME);
type MenuContentElement = MenuRootContentTypeElement;
interface MenuContentProps extends MenuRootContentTypeProps {
  forceMount?: true;
}

const MenuContent = React.forwardRef<MenuContentElement, MenuContentProps>(
  (props: ScopedProps<MenuContentProps>, forwardedRef) => {
    const portalContext = usePortalContext(CONTENT_NAME, props.__scopeMenu);
    const { forceMount = portalContext.forceMount, ...contentProps } = props;
    const context = useMenuContext(CONTENT_NAME, props.__scopeMenu);
    const rootContext = useMenuRootContext(CONTENT_NAME, props.__scopeMenu);

    return (
      <Collection.Provider scope={props.__scopeMenu}>
        <Presence present={forceMount || context.open}>
          <Collection.Slot scope={props.__scopeMenu}>
            {rootContext.modal ? (
              <MenuRootContentModal {...contentProps} ref={forwardedRef} />
            ) : (
              <MenuRootContentNonModal {...contentProps} ref={forwardedRef} />
            )}
          </Collection.Slot>
        </Presence>
      </Collection.Provider>
    );
  }
);
```

처음 `createCollection`을 통해, 실제 Content의 Item이 될 요소를 모아 내부적으로 item을 모두 ref로 참조하여 Map 자료구조를 구성한 Provider와 Slot, ItemSlot을 반환한 `Collection`을 이용하여, scope 영역의 `Provider`와 커스텀하여 컴포넌트가 추가될 수 있는 `Slot`을 이용하였고, `Presence`를 이용했는데 아직 잘 모르는 영역이다. 그리고 `MenuRootContentModal`을 받아 실제 구현체가 되는 부분이 Modal의 형식을 이어받는 다는 것을 유추할 수 있다.

### MenuRootContentModal, MenuRootContentNonModal

useEffect를 보면 독특한 부분이 있는데, `hideOthers`를 통해 MenuContent 한에서 ARIA를 숨기는 작업을 진행한다. 접근성을 우회하려는 목적으로 보인다.

`MenuContentImpl`에 주입되는 항목을 살펴보면, Content가 열려있을 때만 포인터 이벤트를 비활성화하고, 이를 통해 애니메이션화하는 동안 상호작용을 차단하지 않도록 한다. 스크롤도 마찬가지.

또한 포커스가 MenuContent에 갇혀있으면 포커스 아웃 이벤트가 계속 발생할수 있어 event.preventDefault()하는 모습을 볼 수 있다. 반면에 모달이 아닌 경우 `MenuRootContentNonModal` 모든 옵션을 해제하여 이벤트가 연쇄적으로 일어나지 않게 한다.

```tsx
type MenuRootContentTypeElement = MenuContentImplElement;
interface MenuRootContentTypeProps
  extends Omit<MenuContentImplProps, keyof MenuContentImplPrivateProps> {}

const MenuRootContentModal = React.forwardRef<
  MenuRootContentTypeElement,
  MenuRootContentTypeProps
>((props: ScopedProps<MenuRootContentTypeProps>, forwardedRef) => {
  const context = useMenuContext(CONTENT_NAME, props.__scopeMenu);
  const ref = React.useRef<MenuRootContentTypeElement>(null);
  const composedRefs = useComposedRefs(forwardedRef, ref);

  // Hide everything from ARIA except the `MenuContent`
  React.useEffect(() => {
    const content = ref.current;
    if (content) return hideOthers(content);
  }, []);

  return (
    <MenuContentImpl
      {...props}
      ref={composedRefs}
      // we make sure we're not trapping once it's been closed
      // (closed !== unmounted when animating out)
      trapFocus={context.open}
      // make sure to only disable pointer events when open
      // this avoids blocking interactions while animating out
      disableOutsidePointerEvents={context.open}
      disableOutsideScroll
      // When focus is trapped, a `focusout` event may still happen.
      // We make sure we don't trigger our `onDismiss` in such case.
      onFocusOutside={composeEventHandlers(
        props.onFocusOutside,
        (event) => event.preventDefault(),
        { checkForDefaultPrevented: false }
      )}
      onDismiss={() => context.onOpenChange(false)}
    />
  );
});

const MenuRootContentNonModal = React.forwardRef<
  MenuRootContentTypeElement,
  MenuRootContentTypeProps
>((props: ScopedProps<MenuRootContentTypeProps>, forwardedRef) => {
  const context = useMenuContext(CONTENT_NAME, props.__scopeMenu);
  return (
    <MenuContentImpl
      {...props}
      ref={forwardedRef}
      trapFocus={false}
      disableOutsidePointerEvents={false}
      disableOutsideScroll={false}
      onDismiss={() => context.onOpenChange(false)}
    />
  );
});
```

### MenuContentImpl

이 컴포넌트의 핵심이자 사실상 구현부이다. 타입와 구현 로직, return 영역을 나누어 살펴보자. 여기서부턴 내용이 많아 GPT의 요약 내용을 위주로 작성한다.

`MenuContentImp`l는 Radix UI의 Menu 컴포넌트의 핵심 로직을 포함하고 있으며, 메뉴의 포커스 제어, 상호작용 처리, 스크롤 잠금, 포인터 방향 추적 등 다양한 UX 요소를 관리한다.

#### 타입

- `MenuContentImplElement`: `PopperPrimitive.Content`의 DOM 엘리먼트 타입이다. 이는 Popper 컴포넌트의 Content를 참조하여 타입을 지정한다.
- `FocusScopeProps, DismissableLayerProps, RovingFocusGroupProps, PopperContentProps`: 각각 `FocusScope, DismissableLayer, RovingFocusGroup.Root, PopperPrimitive.Content`의 컴포넌트 속성을 사용하기 위한 타입으로, 다양한 상호작용 이벤트와 포커스 제어 속성을 제공한다
- `MenuContentImplPrivateProps`
  - `onOpenAutoFocus, onCloseAutoFocus`: 포커스가 메뉴로 진입하거나 메뉴가 닫힐 때 자동 포커스를 제어.
  - `disableOutsidePointerEvents`: 메뉴 외부에서 발생하는 포인터 이벤트를 비활성화할지 여부를 제어.
  - `disableOutsideScroll`: 메뉴 외부의 스크롤을 막는 기능.
  - `trapFocus`: 포커스를 메뉴 내에 고정하는 기능.

```ts
type MenuContentImplElement = React.ElementRef<typeof PopperPrimitive.Content>;
type FocusScopeProps = React.ComponentPropsWithoutRef<typeof FocusScope>;
type DismissableLayerProps = React.ComponentPropsWithoutRef<
  typeof DismissableLayer
>;
type RovingFocusGroupProps = React.ComponentPropsWithoutRef<
  typeof RovingFocusGroup.Root
>;
type PopperContentProps = React.ComponentPropsWithoutRef<
  typeof PopperPrimitive.Content
>;
type MenuContentImplPrivateProps = {
  onOpenAutoFocus?: FocusScopeProps["onMountAutoFocus"];
  onDismiss?: DismissableLayerProps["onDismiss"];
  disableOutsidePointerEvents?: DismissableLayerProps["disableOutsidePointerEvents"];
  disableOutsideScroll?: boolean;
  trapFocus?: FocusScopeProps["trapped"];
};
interface MenuContentImplProps
  extends MenuContentImplPrivateProps,
    Omit<PopperContentProps, "dir" | "onPlaced"> {
  onCloseAutoFocus?: FocusScopeProps["onUnmountAutoFocus"];
  loop?: RovingFocusGroupProps["loop"];
  onEntryFocus?: RovingFocusGroupProps["onEntryFocus"];
  onEscapeKeyDown?: DismissableLayerProps["onEscapeKeyDown"];
  onPointerDownOutside?: DismissableLayerProps["onPointerDownOutside"];
  onFocusOutside?: DismissableLayerProps["onFocusOutside"];
  onInteractOutside?: DismissableLayerProps["onInteractOutside"];
}
```

#### 선언 및 로직

1. 포커스 및 이벤트 핸들러 구성

- `onOpenAutoFocus, onCloseAutoFocus, onEntryFocus` 등 여러 이벤트 핸들러가 정의되어 있으며, 메뉴가 열릴 때 포커스를 메뉴 내용으로 이동시키고, 닫힐 때 외부 포커스로 돌아가도록 제어합니다.
- `composeEventHandlers` 유틸리티로 이벤트 핸들러를 조합하여 기본 핸들러와 사용자 정의 핸들러를 연결합니다.
- `trapFocus`: FocusScope 내의 trapped 속성으로, 메뉴가 열려 있을 때 포커스를 메뉴 안으로 제한합니다.

2. 포인터 방향 및 서브메뉴 이동 처리

- `pointerDirRef, lastPointerXRef`를 사용하여 포인터 이동 방향을 기록합니다.
- `isPointerMovingToSubmenu` 함수는 포인터가 서브메뉴로 이동하는지를 확인하여 메뉴 닫기 또는 서브메뉴로 이동을 결정합니다.
- `pointerGraceIntentRef`: 서브메뉴로 포인터가 이동할 때 잠시 대기 시간을 제공해 서브메뉴로 이동 시 실수로 메뉴가 닫히지 않도록 합니다.

3. 타입어헤드 검색 (Typeahead Search)

- 사용자가 키보드를 통해 빠르게 항목을 찾을 수 있도록 `handleTypeaheadSearch` 함수가 구현되어 있습니다. 키 입력을 통해 아이템 검색을 수행하고, 일치하는 항목이 있으면 포커스를 이동시킵니다.
- `searchRef`와 `timerRef`는 검색 문자열을 저장하고 일정 시간 후 초기화하여 다음 검색을 준비합니다.

4. Roving Focus

- `RovingFocusGroup`은 키보드로 메뉴 항목을 순환하며 포커스를 이동할 수 있게 합니다. loop 속성을 통해 키보드 네비게이션이 마지막 항목에서 첫 항목으로 순환할 수 있게 합니다.
- `onEntryFocus`에서는 메뉴가 열릴 때 특정 아이템에 포커스를 두도록 제어하며, 키보드를 사용하여 처음 포커스가 이동하도록 설정합니다.

5. 스크롤 잠금 및 `Dismissable Layer`

- `ScrollLockWrapper`는 메뉴가 열려 있을 때 외부 스크롤을 막기 위해 RemoveScroll을 사용하여 스크롤 잠금을 활성화할 수 있습니다.
- `DismissableLayer`는 메뉴 외부에서 발생하는 이벤트를 감지하여 닫기 동작을 제어합니다. disableOutsidePointerEvents, onEscapeKeyDown, onPointerDownOutside, onFocusOutside 등의 속성으로 제어하여 외부 클릭, 포커스 이동 시 메뉴를 닫도록 처리합니다.

6. 메뉴 포커스 가드 (Focus Guards)

- `useFocusGuards`를 통해 Portal 내에 포커스 보호 영역을 생성합니다. 이는 MenuContent가 DOM의 끝에 위치할 때 포커스가 DOM에서 벗어나지 않도록 해주는 역할을 합니다.

```tsx
const MenuContentImpl = React.forwardRef<
  MenuContentImplElement,
  MenuContentImplProps
>((props: ScopedProps<MenuContentImplProps>, forwardedRef) => {
  const {__scopeMenu,loop = false,trapFocus,onOpenAutoFocus,onCloseAutoFocus,disableOutsidePointerEvents,onEntryFocus,onEscapeKeyDown,onPointerDownOutside,onFocusOutside,onInteractOutside,onDismiss,disableOutsideScroll,
    ...contentProps
  } = props;
  const context = useMenuContext(CONTENT_NAME, __scopeMenu);
  const rootContext = useMenuRootContext(CONTENT_NAME, __scopeMenu);
  const popperScope = usePopperScope(__scopeMenu);
  const rovingFocusGroupScope = useRovingFocusGroupScope(__scopeMenu);
  const getItems = useCollection(__scopeMenu);
  const [currentItemId, setCurrentItemId] = React.useState<string | null>(null);
  const contentRef = React.useRef<HTMLDivElement>(null);
  const composedRefs = useComposedRefs(
    forwardedRef,
    contentRef,
    context.onContentChange
  );
  const timerRef = React.useRef(0);
  const searchRef = React.useRef("");
  const pointerGraceTimerRef = React.useRef(0);
  const pointerGraceIntentRef = React.useRef<GraceIntent | null>(null);
  const pointerDirRef = React.useRef<Side>("right");
  const lastPointerXRef = React.useRef(0);

  const ScrollLockWrapper = disableOutsideScroll
    ? RemoveScroll
    : React.Fragment;
  const scrollLockWrapperProps = disableOutsideScroll
    ? { as: Slot, allowPinchZoom: true }
    : undefined;

  const handleTypeaheadSearch = (key: string) => {
    const search = searchRef.current + key;
    const items = getItems().filter((item) => !item.disabled);
    const currentItem = document.activeElement;
    const currentMatch = items.find(
      (item) => item.ref.current === currentItem
    )?.textValue;
    const values = items.map((item) => item.textValue);
    const nextMatch = getNextMatch(values, search, currentMatch);
    const newItem = items.find((item) => item.textValue === nextMatch)?.ref
      .current;

    // Reset `searchRef` 1 second after it was last updated
    (function updateSearch(value: string) {
      searchRef.current = value;
      window.clearTimeout(timerRef.current);
      if (value !== "")
        timerRef.current = window.setTimeout(() => updateSearch(""), 1000);
    })(search);

    if (newItem) {
      /**
       * Imperative focus during keydown is risky so we prevent React's batching updates
       * to avoid potential bugs. See: https://github.com/facebook/react/issues/20332
       */
      setTimeout(() => (newItem as HTMLElement).focus());
    }
  };

  React.useEffect(() => {
    return () => window.clearTimeout(timerRef.current);
  }, []);

  // Make sure the whole tree has focus guards as our `MenuContent` may be
  // the last element in the DOM (because of the `Portal`)
  useFocusGuards();

  const isPointerMovingToSubmenu = React.useCallback(
    (event: React.PointerEvent) => {
      const isMovingTowards =
        pointerDirRef.current === pointerGraceIntentRef.current?.side;
      return (
        isMovingTowards &&
        isPointerInGraceArea(event, pointerGraceIntentRef.current?.area)
      );
    },
    []
  );
  ...
  });
```

#### 렌더링 부분

1. `MenuContentProvider`
   - `MenuContentProvider`는 하위 컴포넌트들에 메뉴의 상호작용 상태와 포인터 관련 데이터를 공유하는 Context를 제공합니다.
   - `scope, searchRef` 등을 넘겨 주어 `MenuContentImpl` 내부에서 사용할 수 있도록 합니다.
   - `onItemEnter, onItemLeave, onTriggerLeave`는 항목에 대한 진입 및 나가기 이벤트를 처리하는 핸들러로, 서브메뉴로의 포인터 이동 여부를 판단하고, 필요한 경우 메뉴의 포커스를 조정합니다.
2. `ScrollLockWrapper`
   - `disableOutsideScroll` 속성에 따라 스크롤 잠금 여부를 제어합니다.
   - `disableOutsideScroll`이 true인 경우 `RemoveScroll`로 메뉴 외부의 스크롤을 잠그고, false인 경우에는 React.Fragment를 사용하여 래핑합니다.
   - `allowPinchZoom`을 통해 스크롤 잠금 시에도 핀치 줌을 허용하도록 설정합니다.
3. `FocusScope`
   - `FocusScope`는 trapFocus 속성에 따라 포커스를 메뉴 내에 고정합니다.
   - `onMountAutoFocus`와 `onUnmountAutoFocus`로 메뉴가 열리고 닫힐 때 포커스 동작을 제어합니다.
   - `onMountAutoFocus` 핸들러에서 event.preventDefault()를 사용하여 메뉴 내용이 열릴 때 특정 요소에 포커스를 이동하지 않도록 하고, contentRef를 통해 콘텐츠 자체에 포커스를 설정합니다.
4. `DismissableLayer`
   - `DismissableLayer`는 메뉴 외부에서 발생하는 이벤트를 감지하여 메뉴 닫기 동작을 제어합니다.
   - `disableOutsidePointerEvents, onEscapeKeyDown, onPointerDownOutside, onFocusOutside, onInteractOutside, onDismiss` 속성을 통해 외부 클릭이나 ESC 키, 포커스 이동 등에 반응하여 메뉴를 닫을 수 있습니다.
5. `RovingFocusGroup.Root`
   - `RovingFocusGroup`은 키보드로 메뉴 항목을 순환하며 포커스를 이동할 수 있도록 해줍니다.
   - `loop` 속성은 메뉴의 포커스가 끝에서 다시 처음으로 돌아가도록 설정합니다.
   - `currentTabStopId`는 현재 포커스가 위치한 항목을 나타내며, setCurrentTabStopId로 이를 업데이트합니다.
   - `onEntryFocus` 핸들러는 키보드를 사용할 때만 첫 항목에 포커스가 이동하도록 제어합니다.
6. `PopperPrimitive.Content`
   - `PopperPrimitive.Content`는 메뉴의 메인 콘텐츠 요소로, Popper를 이용해 메뉴를 기준 위치에 띄우고 스타일링합니다.
   - `role="menu", aria-orientation="vertical"`로 메뉴의 접근성 속성을 설정합니다.
   - `data-state, data-radix-menu-content`는 메뉴 상태와 Radix UI의 데이터 속성을 추가하여 상태와 스타일을 반영합니다.
   - `ref={composedRefs}`는 `contentRef`와 `forwardedRef`를 합성하여 필요한 여러 참조를 결합합니다.
7. 핸들러
   - `onKeyDown`: 메뉴에서 Tab 키 이동을 막고, 캐릭터 입력 시 `handleTypeaheadSearch`를 호출하여 사용자가 항목을 검색할 수 있도록 합니다. 첫/마지막 항목에 대해 키 이벤트에 따라 포커스를 이동하는 기능을 수행합니다.
   - `onBlur`: 메뉴를 벗어날 때 검색 문자열을 초기화하여 새로운 검색을 준비합니다.
   - `onPointerMove`: 마우스 포인터 이동을 감지하여 `pointerDirRef`에 현재 포인터 방향을 저장합니다. 이는 서브메뉴 진입 시 포인터 방향을 확인하기 위함입니다.

```tsx
const MenuContentImpl = React.forwardRef<
  MenuContentImplElement,
  MenuContentImplProps
>((props: ScopedProps<MenuContentImplProps>, forwardedRef) => {
...
  return (
    <MenuContentProvider
      scope={__scopeMenu}
      searchRef={searchRef}
      onItemEnter={React.useCallback(
        (event) => {
          if (isPointerMovingToSubmenu(event)) event.preventDefault();
        },
        [isPointerMovingToSubmenu]
      )}
      onItemLeave={React.useCallback(
        (event) => {
          if (isPointerMovingToSubmenu(event)) return;
          contentRef.current?.focus();
          setCurrentItemId(null);
        },
        [isPointerMovingToSubmenu]
      )}
      onTriggerLeave={React.useCallback(
        (event) => {
          if (isPointerMovingToSubmenu(event)) event.preventDefault();
        },
        [isPointerMovingToSubmenu]
      )}
      pointerGraceTimerRef={pointerGraceTimerRef}
      onPointerGraceIntentChange={React.useCallback((intent) => {
        pointerGraceIntentRef.current = intent;
      }, [])}
    >
      <ScrollLockWrapper {...scrollLockWrapperProps}>
        <FocusScope
          asChild
          trapped={trapFocus}
          onMountAutoFocus={composeEventHandlers(onOpenAutoFocus, (event) => {
            // when opening, explicitly focus the content area only and leave
            // `onEntryFocus` in  control of focusing first item
            event.preventDefault();
            contentRef.current?.focus({ preventScroll: true });
          })}
          onUnmountAutoFocus={onCloseAutoFocus}
        >
          <DismissableLayer
            asChild
            disableOutsidePointerEvents={disableOutsidePointerEvents}
            onEscapeKeyDown={onEscapeKeyDown}
            onPointerDownOutside={onPointerDownOutside}
            onFocusOutside={onFocusOutside}
            onInteractOutside={onInteractOutside}
            onDismiss={onDismiss}
          >
            <RovingFocusGroup.Root
              asChild
              {...rovingFocusGroupScope}
              dir={rootContext.dir}
              orientation="vertical"
              loop={loop}
              currentTabStopId={currentItemId}
              onCurrentTabStopIdChange={setCurrentItemId}
              onEntryFocus={composeEventHandlers(onEntryFocus, (event) => {
                // only focus first item when using keyboard
                if (!rootContext.isUsingKeyboardRef.current)
                  event.preventDefault();
              })}
              preventScrollOnEntryFocus
            >
              <PopperPrimitive.Content
                role="menu"
                aria-orientation="vertical"
                data-state={getOpenState(context.open)}
                data-radix-menu-content=""
                dir={rootContext.dir}
                {...popperScope}
                {...contentProps}
                ref={composedRefs}
                style={{ outline: "none", ...contentProps.style }}
                onKeyDown={composeEventHandlers(
                  contentProps.onKeyDown,
                  (event) => {
                    // submenu key events bubble through portals. We only care about keys in this menu.
                    const target = event.target as HTMLElement;
                    const isKeyDownInside =
                      target.closest("[data-radix-menu-content]") ===
                      event.currentTarget;
                    const isModifierKey =
                      event.ctrlKey || event.altKey || event.metaKey;
                    const isCharacterKey = event.key.length === 1;
                    if (isKeyDownInside) {
                      // menus should not be navigated using tab key so we prevent it
                      if (event.key === "Tab") event.preventDefault();
                      if (!isModifierKey && isCharacterKey)
                        handleTypeaheadSearch(event.key);
                    }
                    // focus first/last item based on key pressed
                    const content = contentRef.current;
                    if (event.target !== content) return;
                    if (!FIRST_LAST_KEYS.includes(event.key)) return;
                    event.preventDefault();
                    const items = getItems().filter((item) => !item.disabled);
                    const candidateNodes = items.map(
                      (item) => item.ref.current!
                    );
                    if (LAST_KEYS.includes(event.key)) candidateNodes.reverse();
                    focusFirst(candidateNodes);
                  }
                )}
                onBlur={composeEventHandlers(props.onBlur, (event) => {
                  // clear search buffer when leaving the menu
                  if (!event.currentTarget.contains(event.target)) {
                    window.clearTimeout(timerRef.current);
                    searchRef.current = "";
                  }
                })}
                onPointerMove={composeEventHandlers(
                  props.onPointerMove,
                  whenMouse((event) => {
                    const target = event.target as HTMLElement;
                    const pointerXHasChanged =
                      lastPointerXRef.current !== event.clientX;

                    // We don't use `event.movementX` for this check because Safari will
                    // always return `0` on a pointer event.
                    if (
                      event.currentTarget.contains(target) &&
                      pointerXHasChanged
                    ) {
                      const newDir =
                        event.clientX > lastPointerXRef.current
                          ? "right"
                          : "left";
                      pointerDirRef.current = newDir;
                      lastPointerXRef.current = event.clientX;
                    }
                  })
                )}
              />
            </RovingFocusGroup.Root>
          </DismissableLayer>
        </FocusScope>
      </ScrollLockWrapper>
    </MenuContentProvider>
  );
});

MenuContent.displayName = CONTENT_NAME;
```

## 요약

이 구조와 이벤트 핸들러를 통해 `MenuContentImpl`은 사용자와 상호작용하며 포커스, 스크롤, 외부 클릭, 키보드 내비게이션 등 다양한 사용자 이벤트에 대응하는 역할을 수행합니다. 각 컴포넌트는 특정 상호작용 및 UX 기능을 담당하며, 이를 통해 직관적이고 접근성 높은 메뉴를 구성합니다.

## 출처

- [Menu.tsx](https://github.com/radix-ui/primitives/blob/main/packages/react/menu/src/Menu.tsx#L41)
