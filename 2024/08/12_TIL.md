# Zustand 내부 구조 살펴보기(with. VanillaJS, React)

## 정리

- 리덕스에 영감을 받은 zustand. 하나의 스토어를 중앙 집중형으로 활용해 상태 관리하는 라이브러리
- `bottom-up` 방식의 recoil, jotai와 비교해서 `Provider`의 제약이 없어 바닐라에서도 가능하게 설계한 점이 강력하다.
- 타 라이브러리에 비해 굉장히 가볍다.(2.9kb)
- `create`를 외부에 선언하여 getter, setter 구조를 미리 선언하여 작성이 가능하며, selector를 구분해서 작성이 가능하여 최적화 코드도 가능하다.
- 기본적으로 상태관리 라이브러리의 공통 구조인 `state, setState, subscribe, listener`를 선언하고 생성하는 구조가 동일하며, 독특한 점은 이 모두를 호출부에서 다시 재호출해서 사용도 가능하다는 점이다.

### 참고

- 주요 키워드: zustand, VanilaJS, 상태 관리
- 관련 기술: 클로저

## 무엇을 알았는지

### 바닐라 코드 구조

```tsx
const createStoreImpl = (CreateStoreImp = (createState) => {
  type TState = ReturnType<typeof createState>;
  type Listener = (state: TState, prevState: TState) => void;

  // 외부에서 관리하는 스토어의 상태 값
  let state: TState;
  const listener: Set<Listener> = new Set();

  // # partial: state의 일부분만 변경시
  // # replace: state를 완전히 새로운 값으로 변경시
  const setState: SetStateInternal<TState> = (partial, replace) => {
    const nextState =
       typeof partial = 'function'
          ? (partial as (State: TState) => TState)(state)
             : partial;
       if (nextState !== state) {
         const previousState = state;
         state =
            replace ?? typeof nextState !== 'object'
               ? (nextState as TState)
                  : Object.assign({}, state, nextState);
         listeners.forEach((listener) => listener(state, previousState));
       }
  };

  // 클로저의 최신 값을 가져오는 함수
  const getState: () => TState = () => state;

  // listener를 등록하는 함수이며, Set형태로 선언되어 추가,삭제,중복 관리가 용이하게 설계
  // 즉, 상태값이 변경될 때 리렌더링이 필요한 컴포넌트에 전파될 목적으로 만들어짐!
  const subscribe: (listener: Listener) => () => void = (listener) => {
   listeners.add(listener);
   // Unsubscribe
   return () => listeners.delete(listener);
  };

   // listener 초기화
   const destroy: () => void = () => listeners.clear();
   const api = { setState, getState, subscribe, destroy};
   state = (createState as PopArgument<typeof createState>)(
      setState, getState, api
   );
   return api as any;
});
```

> zustand가 바닐라 자바스크립트에서도 사용할 수 있는 이유?

- store는 리액트를 비롯한 그 어떤 프레임워크와는 별개로 완전히 독립적으로 구성됨

#### 활용 예제

```tsx
type CounterStore = {
  count: number;
  increase: (num: number) => void;
};

const store = createStore<CounterStore>((set) => ({
  count: 0,
  increase: (num: number) => set((state) => ({ count: state.count + num })),
}));

store.subscribe((state, prev) => {
  if (state.count !== prev.count) {
    console.log("changed!", state.count);
  }
});

store.setState((state) => ({ count: state.count + 1 }));
store.getState().increase(10);
```

- `createStore`를 만들 때 `set`이라는 인수를 활용해 생성이 가능하다.
- 이는 `createStoreImpl` 함수를 보면 알수 있는데 state 생성시 `setState, getState, api`를 인수로 넘겨주었기 때문에 가능하다.
- `set`을 통해 현재 스토어의 값을 재정할 수도 있고, 두 번째 인수로 get을 추가해 현재 스토어의 값을 받아올 수도 있다.

이렇게 생성된 스토어는 `getState`,`setState`를 통해 현재 스토어의 값을 반환하거나 재정의할 수 있다. 또한 subscribe를 통해 스토어의 값이 변경될 때마다 특정 함수 실행이 가능(최적화 가능)하다.

### 리액트 코드 구조

리액트에선 store를 읽고 리렌더링이 필요하다. 이때 제공되는 함수가 `useStore`와 `create`이다.

```tsx
// useStore.tsx
export function useStore<TState, StateSlice>(
  api: WithReact<StoreApi<TState>>,
  selector: (state: TState) => StateSlice = api.getState as any,
  equalityFn?: (a: StateSlice, b: StateSlice) => boolean
) {
  const slice = useSyncExternalStoreWithSelector(
    api.subscribe,
    api.getState,
    api.getServerState || api.getState,
    selector,
    equalityFn
  );
  useDebugValue(slice);
  return slice;
}
```

`useSyncExternalStoreWithSelector`와 `useSyncExternalStore`와 동일하지만 원하는 값을 가져올 수 있는 selector와 동등 비교를 할 수 있는 equalityFn 함수를 받는다는 차이가 있다.

즉, 객체가 예상되는 외부 상태값에서 일부 값을 꺼내올 수 있또록 `useSyncExternalStoreWithSelector`를 사용한다.

```tsx
// create.tsx
const createImpl = <T,>(createState: StateCreator<T, [], []>) => {
  const api =
    typeof createState == "function" ? createStore(createState) : createState;

  const useBoundStore: any = (selector?: any, equalityFn?: any) =>
    useStore(api, selector, equalityFn);

  Object.assign(useBoundStore, api);
  return useBoundStore;
};

const create = (<T,>(createState: StateCreator<T, [], []> | undefined) =>
  createState ? createImpl(createState) : createImp) as Create;

return default create;
```

바닐라의 `createStore` 기반으로 제작되어 거의 유사하다. 다만 `useStore`를 사용해 해당 스토어 즉시 리액트 컴포넌트에서 사용가능하도록 만들어졌다.

또한 `useBoundStore`에 api를 Object.assign으로 복사하여 `useBoundStore`에 api의 모든 함수를 복사해서 api도 동일하게 사용가능 하도록 제공했다.

#### 활용 예제

```tsx
// createStore 활용
interface Store {
  count: number;
  increase: (count: number) => void;
}
const store = createStore<Store>((set) => ({
  count: 0,
  increase: (n) => set((state) => ({ count: state.count + n })),
}));

const counterSelector = ({ count, increase }: Store) => ({
  count,
  increase,
});

function Counter() {
  const { count, increase } = useStore(store, counterSelector);
  function handleClick() {
    increase(1);
  }
  return (
    <>
      {count} <button onClick={handleClick}>+</button>
    </>
  );
}
```

아래 `create`와 마찬 가지로 외부에서 store 생성도 가능하다.

```tsx
// (추천) create 활용
import { create } from "zustand";
const useCounterStore = create((set) => ({
  count: 1,
  inc: () => set((state) => ({ count: state.count + 1 })),
}));

function Counter() {
  const { count, inc } = useCounterStore();

  return (
    <>
      {count} <button onClick={inc}>+</button>
    </>
  );
}
```

### 출처

- [모던 리액트 Deep Dive](https://m.yes24.com/Goods/Detail/123161563)
