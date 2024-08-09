# React hook으로 전역 상태 관리 만들어보기

## 정리

- 리액트의 훅과 상태관리 라이브러리의 동작은 클로저를 기반으로 동작
- useState는 useReducer로 작성되었으며, 서로서로 각 훅의 기능 구현이 가능하다.
- 전역 상태 관리에서 중점은 전역 상태값을 알리는 `store`, 값의 변경을 알리기 위한 `callback`, 이를 등록하는 `subscribe` 함수로 구현된다.
- 특정 상태만 변경하기위해 `selector` 함수 구현과정을 살펴볼 수 있으며, 불필요한 리렌더링을 막기 위한 흐름 파악이 가능하다.
- 위 모든 기능을 망라하는 훅을 메타팀에서 `useSubscription`으로 구현했으며, 현재는 `useSyncExternalStore`훅으로 대체되었다.

### 참고

- 주요 키워드: 상태관리, useReducer, useState, useSyncExternalStore
- 관련 기술: React, Redux

## 무엇을 알았는지

### 지역 상태 관리 - useState, useReducer 만들기(with. preact 코드 기반)

1. **useState -> useReducer 만들기**

   ```ts
   type Initializer<T> = T extends any ? T | ((prev: T) => T) : never

   function useStateWithUseReducer<T>(initialSTate: T) {
      const [state, dispatch] = useReducer(
         (prev: T, action: Initializer<T>) =>
            typeof action === 'function' ?? action(prev) : action,
         initialState
      )

      return [state, dispatch]
   }
   ```

- 실제 `useState`를 구현하기위해 `useReducer`로 구현되어있다.
- `useReducer`의 첫 번째 인수로 reducer, 즉 state와 action을 어떻게 정의할지 넘겨줘야하는데 useState와 동일한 작동한다.
- 즉 `T`를 받거나 `(prev: T) => T`를 받아 새로운 값을 설정한다.

2. **useReducer -> useState 만들기**

   ```ts
   function useReducerWithUseState(reducer, initialState, initializer) {
     const [state, setState] = useState(
       initializer ? () => initializer(initialState) : initialState
     );

     const dispatch = useCallback(
       (action) => setState((prev) => reducer(prev, action)),
       [reducer]
     );

     return [state, dispatch];
   }
   ```

- `useReducer`를 타입스크립트로 작성하기 위해 다양한 형태의 오버로딩이 필요하다.

둘다 구현상의 차이가 있을 뿐 모두 `지역 상태 관리`를 위해 만들어지는 것을 알수 있다.

3. 지역 상태 관리의 한계

- 위 두 훅을 사용할시 선언될 때마다 새롭게 초기화되어, 결론적으로 컴포넌트별로 상태를 `파편화`되어진다.

### 전역 상태 관리 만들어보기

- useState의 한계로는 React가 만든 `클로저 내부`에서 관리되어 지역 상태로 생성되기 때문에 선언된 컴포넌트에서만 사용이 가능하다.
- **useState 상태를 바깥으로 분리하기**
  - 만약 useState가 리액트 클로저가 아닌 `다른 자바스크립트 실행 문맥 어딘가서 초기화`되어서 관리된다면?
  - 그 `상태를 참조하는 유효한 스코프 내부에서는 해당 객체의 값을 공유`해서 사용이 가능하지 않을까?
  - 또한 값`을 참조하는 컴포넌트가 훅에서 그 업데이트된 값을 사용`이 가능하지 않을까?

### 1. 컴포넌트 레벨의 지역 상태를 벗어나는 새로운 상태관리 만들기

- 이 상태는 객체, 원시값일 수도 있으니 범용적인 용어로 `store`로 정의한다.
- `store` 값의 변경을 알리는 `callback` 함수를 실행한다.
- `callback`을 등록하는 `subscribe` 함수를 만든다.

```ts
export type State = { counter: number };
type Initializer<T> = T extends any ? T | ((prev: T) => T) : never;
type Store<State> = {
  get: () => State; // 항상 최신값을 가져오기위해 함수로 구현
  set: (action: Initializer<State>) => State; // useState와 동일하게 값 또는 함수를 받기위함
  subscribe: (callback: () => void) => () => void; // store의 변경을 감지하고 싶은 컴포넌트 들이 자신의 callback 함수를 등록함
};
```

```ts
export const createStore = <State extends unknown>(
  initialState: Initializer<State>
): Store<State> => {
  // useState와 동일하게 게이른 초기화를 위한 함수 및 값을 받도록함
  // state의 값은 스토어 내부에서 보관해야하므로 변수 선언함
  let state =
    typeof initialState !== "function" ? initialState : initialState();

  // callbacks는 자료형에 관계없이 유일한 값을 저장할 수 있는 Set을 사용
  const callbacks = new Set<() => void>();

  const get = () => state;
  const set = (nextSTate: State | ((prev: State) => State)) => {
    state =
      typeof nextState === "function"
        ? (nextState as (prev: State) => State)(state)
        : nextState;

    // 값의 설정이 발생시 콜백 목록을 순회하면서 모두 실행
    callbacks.forEach((callback) => callback());

    return state;
  };
  // subscribe는 콜백을 인수로 받음
  const subscribe = (callback: () => void) => {
    // 받은 함수를 콜백 목록에 추가
    callbacks.add(callback);
    // 클린업 실행 시 이를 삭제해서 반복적으로 추가되는 것을 방지함
    return () => {
      callbacks.delete(callback);
    };
  };

  return { get, set, subscribe };
};
```

위 함수를 요약하자면

- `createStore`는 자신이 관리해야하는 상태를 내부 변수로 가짐
- get 함수로 해당 변수를 최신 값을 제공
- set 함수로 내부 변수를 최신화하면서 이 과정에서 등뢱된 콜백을 모조리 실행하는 구조
- `(추가로 필요한 부분)` store의 값을 참조하고 이 값의 변화에 따라 컴포넌트 렌더링을 요할 사용자 정의 훅이 필요!

### 2. store의 변화를 감지하는 useStore 만들기

```ts
// useStore.ts
export const useStore = <State extends unkown>(store: Store<State>) => {
  const [state, setState] = useState<State>(() => store.get());
  useEffect(() => {
    const unsubscribe = store.subscribe(() => {
      setState(store.get());
    });
    return unsubscribe;
  }, [store]);

  return [state, store.set] as const;
};

// Counter.tsx
const store = createStore({ counter: 0 });
function Counter() {
  const [state, setState] = useStore(store);

  function handleClick() {
    setState((prev) => ({ counter: prev.counter + 1 }));
  }

  return (
    <>
      <h3>{state.counter}</h3>
      <button onClick={handleClick}>+</button>
    </>
  );
}
```

### 3. 특정 참조 값만 변경하여 불필요한 렌더링을 막아주는 useStoreSelector 만들기

```ts
// useStoreSelector.ts
export const useStoreSelector = <State extends unknown, Value extends unknown>(
   store: Store<State>,
   selector: (state: State) => Value.
) => {
   const [state, setState] = useState(() => selector(store.get()))
   useEffect(() => {
      const unsubscribe = store.subscribe(() => {
         const value = selector(store.get())
         setState(value);
      });

      return unsubscribe;
   }, [store, selector])

   return state
}

// Counter.tsx
const store = createStore({ counter: 0, text: 'tap' });
function Counter() {
   // useCallback을 하는 이유는 컴포넌트 밖에 selector를 선언하거나 불가능할때, useCallback의 참조를 고정해야한다.
   // 만약 컴포넌트 내에 selector 함수를 생성하고 감싸지 않으면 리렌더링될 때마다 함수가 계속 생성되어 store의 subscribe를 반복적으로 수행하게된다.
  const counter = useStoreSelector(
   store,
   useCallback((state) => state.count, [])
  )

  useEffect(() => console.log("Counter Render"))

  function handleClick() {
    store.set((prev) => ({ ...prev, count: prev.count + 1}))
  }

  return (
    <>
      <h3>{state.counter}</h3>
      <button onClick={handleClick}>+</button>
    </>
  );
}

// TextEditor.tsx
const textSelector = (state: ReturnType<typeof store.get>) => state.text

function TextEditor() {
  const text = useStoreSelector(store, textSelector)

  useEffect(() => console.log("TextEditor Render"))

  function handleChange() {
    store.set((prev) => ({ ...prev, text: e.target.value}))
  }

  return (
    <>
      <h3>{test}</h3>
      <input onChange={handleChange} />
    </>
  );
}

```

### 이미 존재했던 훅 useSubscription(현재는 `useSyncExternalStore`)

1. **useSubscription 사용방법**

```ts
function NewCounter() {
  const subscription = useMemo(
    () => ({
      getCurrentValue: () => store.get(),
      subscribe: (callback: () => void) => {
        const unsubscribe = store.subscribe(callback);
        return () => unsubscribe();
      },
    }),
    []
  );
  const value = useSubscription(subscription);
  return <>{JSON.stringify(value)}</>;
}
```

2. **useSubscription 구현하기**

```ts
export function useSubscription<Value>({
  getCurrentValue,
  subscribe,
}: {
  getCurrentValue: () => Value;
  subscribe: (callback: Function) => () => void;
}): Value {
  const [state, setState] = useState(() => ({
    getCurrentValue,
    subscribe,
    value: getCurrentValue(),
  }));

  // 현재 가져올 값을 따로 변수 저장
  let valueToReturn = state.value;

  // useState에 저장된 최신값을 가져오느 함수 getCurrentValue와
  // 함수의 인수로 받은 getCurrentValue가 다르거나
  // useState에 저장돼 있는 subscribe와 함수의 인자로 받은 subscribe가 다르다면
  if (
    state.getCurrentValue !== getCurrentValue ||
    state.subscribe !== subscribe
  ) {
    // 참조가 바뀌었으니 getter로 다시 실행하여 최신화
    valueToReturn = getCurrentValue();

    // 참조가 변경되었으니 일괄 없데이트
    setState({
      getCurrentValue,
      subscribe,
      value: getCurrentValue(),
    });
  }

  // useEffect 실행시 위 if문으로 getCurrentValue와 subscribe의 참조를 최신 상태로 유지한다.
  useEffect(() => {
    // subscribe 취소 여부
    let didUnsubscribe = false;

    // 값의 업데이트 확인 함수
    const checkForUpdate = () => {
      // 이미 subscribe가 취소라면 return
      if (subscribe) {
        return;
      }
      const value = getCurrentValue();
      // useSubscription을 렌더링 시키는 코드
      setState((prevState) => {
        if (
          prevState.getCurrentValue !== getCurrentValue ||
          prevState.subscribe !== subscribe
        ) {
          return prevState;
        }

        if (prevState.value === value) {
          return prevState;
        }

        return { ...prevState, value };
      });
    };

    // 콜백 목록에 변경 여부 체크하는 함수 등록
    const unsubscribe = subscribe(checkForUpdate);
    checkForUpdates();
    return () => {
      didUnsubscribe = true;
      unsubscribe();
    };
  }, [getCurrentValue, subscribe]);

  return valueToReturn;
}
```

- 차이점으로는 selector(여기서는 `getCurrentValue`)와 subscribe에 대한 비교가 추가되었다.
- useStore나 useStoreSelector 모두 useEffect 의존성 배열에 store나 selector가 들어가 있어 객체가 임의로 변경시 불필요한 리렌더가 발생하는 문제가 있었다.
- 이를 방지하기 위해 내부에 예외처리를 추가하여 store나 selector의 벼녁응ㄹ 무시하고 한정적으로 원하는 값을 반환하게끔 작성돼 있다.
- 현재는 `useSyncExternalStore`로 작성되어있다.

### 출처

- [모던 리액트 Deep Dive](https://m.yes24.com/Goods/Detail/123161563)
