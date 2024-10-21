# [카카오페이] RTK(Redux-Toolkit)에서 React-Query의 전환 사례

이번 글은 카카오 페이에서 RTK(Redux-Toolkit) 사용 중 많은 보일러 플레이트와 일관 되지 않은 개발자 및 사용자 경험을 해소하기 위해 React-Query를 도입하게된 이유와 전환 사례를 보여준다.

## RTK의 불편한점

Redux를 사용하여 API 통신과 비동기 데이터 관리에 있어서 많은 불편함. Redux는 "전역 상태 관리 Library"이다. 따라서 API 데이터에 대한 관리에 있어서 적합하지 않았고, 비동기 프로세스를 다루기 위해선 Redux-Saga를 사용하여 Middleware를 두어 API 사이드 이펙트 관리를 진행해야 한다.

따라서 전역 상태 관리인 Redux를 사용하여 비동기 데이터 관리시 컴포넌트의 생명주기와 관계없이 전역에서 비동기 데이터가 관리 되기 때문에 캐싱과 같은 최적화 작업을 쉽게 수행하고 복잡한 사용자 시나리오 대응에도 용이하다. 하지만 그에 따르는 단점이 너무 많다.

### 많은 보일러 플레이트

[Redux 기본 원칙](https://redux.js.org/understanding/thinking-in-redux/three-principles)인 *단일 책임 원칙, 상태는 읽기 전용 상태 변경은 순수 함수로 생성*의 원칙으로 인해, 많은 보일러 플레이트가 요구된다. 그에 따라 RTK가 등장했지만 여전히 비동기 데이터를 다루기 위해 많은 코드가 필요하다.

```tsx
// Todo.tsx
import { useEffect } from "react";
import { selectTodoList } from "features/todos/todos.selector";
import {
  requestFetchTodos,
  requestPostTodos,
} from "features/todos/todos.slice";
import { useForm } from "react-hook-form";
import { useDispatch, useSelector } from "react-redux";

function Todo() {
  const dispatch = useDispatch();
  const data = useSelector(selectTodoList);

  const { register, handleSubmit } = useForm<{
    contents: string;
  }>();

  useEffect(() => {
    // 컴포넌트가 마운트 될 때 서버에 저장된 Todo 정보를 불러옵니다.
    dispatch(requestFetchTodos());
  }, [dispatch]);

  const onSubmit = handleSubmit((value) => {
    // 사용자가 Form을 Submit하면 서버에 새로운 Todo 정보를 저장합니다.
    dispatch(requestPostTodos(value.contents));
  });

  return (
    <div>
      <header>
        <form onSubmit={onSubmit}>
          <input
            type="text"
            placeholder="What needs to be done?"
            autoComplete="off"
            {...register("contents")}
          />
        </form>
      </header>
      <div>
        <ul>
          {data?.map(({ id, contents }) => (
            <li key={id}> {contents} </li>
          ))}
        </ul>
      </div>
    </div>
  );
}

export default Todo;
```

```tsx
// features/todos/todos.slice.ts
import { createSlice, PayloadAction } from "@reduxjs/toolkit";
import { TodoItem } from "types/todo";

export interface TodoListState {
  fetchTodos: {
    data?: TodoItem[];
    isLoading: boolean;
    error?: Error;
  };
  postTodos: {
    isLoading: boolean;
    error?: Error;
  };
}

const initialState: TodoListState = {
  fetchTodos: {
    data: undefined,
    isLoading: false,
    error: undefined,
  },
  postTodos: {
    isLoading: false,
    error: undefined,
  },
};

export const todoListSlice = createSlice({
  name: "todoList",
  initialState,
  reducers: {
    // 이전 State의 값을 바탕으로 다음 State의 값을 새로 만드는 순수함수 "Reducer"
    // redux-toolkit은 immer를 내부적으로 사용하므로 조금 더 자연스럽게 Reducer를 구성할 수 있게끔 도와줍니다.
    requestFetchTodos: (state) => {
      state.fetchTodos.isLoading = true;
    },
    successFetchTodos: (state, action: PayloadAction<TodoItem[]>) => {
      state.fetchTodos.data = action.payload;
      state.fetchTodos.isLoading = false;
      state.fetchTodos.error = undefined;
    },
    errorFetchTodos: (state, action: PayloadAction<string>) => {
      state.fetchTodos.data = undefined;
      state.fetchTodos.isLoading = false;
      state.fetchTodos.error = action.payload;
    },
    requestPostTodos: (state, _: PayloadAction<string>) => {
      state.postTodos.isLoading = true;
    },
    successPostTodos: (state) => {
      state.postTodos.isLoading = false;
    },
    errorPostTodos: (state, action: PayloadAction<string>) => {
      state.postTodos.isLoading = false;
      state.postTodos.error = action.payload;
    },
  },
});

export const {
  requestFetchTodos,
  successFetchTodos,
  errorFetchTodos,
  requestPostTodos,
  successPostTodos,
  errorPostTodos,
} = todoListSlice.actions;

export default todoListSlice.reducer;
```

```tsx
// features/todos/todos.saga.ts
import { PayloadAction } from "@reduxjs/toolkit";
import axios from "axios";
import { call, put, takeEvery } from "redux-saga/effects";
import { TodoItem } from "../../types/todo";
import {
  errorFetchTodos,
  errorPostTodos,
  requestFetchTodos,
  requestPostTodos,
  successFetchTodos,
  successPostTodos,
} from "./todos.slice";

async function getTodoList() {
  const { data } = await axios.get<TodoItem[]>("./todos");
  return data;
}

function* requestFetchTodoTask() {
  try {
    const data: TodoItem[] = yield call(getTodoList);
    yield put(successFetchTodos(data));
  } catch (e) {
    yield put(errorFetchTodos(e.message));
  }
}

async function postTodoList(contents: string) {
  await axios.post("/todos", { contents });
}

function* requestPostTodoTask(action: PayloadAction<string>) {
  try {
    yield call(postTodoList, action.payload);
    yield put(successPostTodos());
  } catch (e) {
    yield put(errorPostTodos(e.message));
  }
}

function* successPostTodoTask() {
  // 서버에 새로운 Todo 추가 요청 성공 시
  // 서버에서 Todo 목록을 다시 받아오기 위해 Action Dispatch
  yield put(requestFetchTodos());
}

function* todoListSaga() {
  yield takeEvery(requestFetchTodos.type, requestFetchTodoTask);
  yield takeEvery(requestPostTodos.type, requestPostTodoTask);
  yield takeEvery(successPostTodos.type, successPostTodoTask);
}

export default todoListSaga;
```

상당한 분량의 TodoList 앱 코드이다. 하나의 API 요청을 위해 여러개의 Action과 Reducer가 필요하고, 비동기 Action 처리를 위해 redux-saga를 통해야한다. 이로써 비동기 Action 처리를 위한 복잡성이 높아지게 된다.

### API 요청 수행을 위한 규격화된 방식 부재

Redux는 API 통신과 비동기 상태 관리를 위한 라이브러리가 아니다. 따라서 상태 저장을 위한 코드를 모두 정의해줘야하는데, 아래와 같이 인터페이스를 정의해줘야한다.

```ts
// 로딩 상태를 관리하는 방법도 개발자에 따라 다르게 구현됩니다.
interface ApiState {
  data?: Data;
  isLoading: boolean;
  error?: Error;
}

interface ApiState {
  data?: Data;
  status: "IDLE" | "LOADING" | "SUCCESS" | "ERROR";
  error?: Error;
}
```

이는 범용적으로 사용할 수 있는 전역 상태 관리이기 때문이기도 하며, Redux Middleware로 비동기를 불러오고 내부 구현을 각 개발자가 구현하다보니 데이터 관리 방법도 모두 달라지게 된다.

이처럼 `API 상태를 관리하기 위한 규격화된 방식`이 필요했다는점

### UX를 위해 필요 이상의 코드 작업

웹뷰 작업시 웹뷰 특성상 앱을 Background로 내리고 시간이 지난 뒤 Foreground로 올리는 형태를 취하는데, 이때 앱이 Foreground로 올라온 시점에 데이터 동기화를 다시 수행해야하는 시나리오가 존재한다.

```tsx
// Todo.tsx
function Todo() {
  // ...전략

  useEffect(() => {
    const handleVisibilityChange = () => {
      if (document.visibilityState === 'visible') {
        dispatch(requestFetchTodos());
      }
    }

    // window focus 이벤트 발생시 Todo API 요청
    document.addEventListener('visibilitychange', handleVisibilityChange);
    return () => document.removeEventListener('visibilitychange', handleVisibilityChange);
  }, [dispatch]);

  return (
    // ...후략
  );
}

export default Todo;
```

이런 코드는 직접 구현시 리소스와 복잡도도 높아져 유지보수에 큰 부담이 생기게 된다.

## 대망의 React-Query

React-Query는 리액트 앱에서 서버 상태를 불러오고, 지속적 동기화를 해주는 라이브러리이다. 여기선 React-Query에 대한 사용방법과 예시는 다르지 않는다.

### RTK로 작성된 TodoList를 React-Query로 전환하기

위 RTK의 보일러 플레이트에서 아래 React-Query로 작성하게되면??

```tsx
// quires/useTodosQuery.ts
// API 상태를 불러오기 위한 React Query Custom Hook
import axios from "axios";
import { useQuery } from "react-query";
import { TodoItem } from "types/todo";

// useQuery에서 사용할 UniqueKey를 상수로 선언하고 export로 외부에 노출합니다.
// 상수로 UniqueKey를 관리할 경우 다른 Component (or Custom Hook)에서 쉽게 참조가 가능합니다.
export const QUERY_KEY = "/todos";

// useQuery에서 사용할 `서버의 상태를 불러오는데 사용할 Promise를 반환하는 함수`
const fetcher = () => axios.get<TodoItem[]>("/todos").then(({ data }) => data);

const useTodosQuery = () => {
  return useQuery(QUERY_KEY, fetcher);
};

export default useTodosQuery;
```

Redux의 기본 원칙을 위한 조건으로 State와 수많은 Action, 그리고 API를 통한 사이드 이펙트 관리를 위해 redux-saga를 통한 미들웨어 코드 작성을 React-Query에선 한두줄로 처리 되었다.

### API 요청 수행에 대한 규격화된 인터페이스 제공

Redux는 비동기 데이터 관리를 위한 라이브러리가 아니다. 따라서 Middleware 부터 State 구조까지 다양한 부분을 설계하고 구현해야했다. 이는 불필요한 고민과 커뮤니케이션 비용을 증가 시키는 요인이기도 했다.

React-Query는 `비동기 데이터 관리를 위한 라이브러리`이면서 상당히 잘만들어진 인터페이스를 제공한다. 위 Redux를 사용할때 정의했던 ApiState 인터페이스와 같은 방식이 여기선 이미 정의된 인터페이스를 제공하여 많은 상태와 데이터 제공을 해준다.

```ts
const {
  data,
  dataUpdatedAt,
  error,
  errorUpdatedAt,
  failureCount,
  isError,
  isFetched,
  isFetchedAfterMount,
  isFetching,
  isIdle,
  isLoading,
  isLoadingError,
  isPlaceholderData,
  isPreviousData,
  isRefetchError,
  isRefetching,
  isStale,
  isSuccess,
  refetch,
  remove,
  status,
} = useQuery(queryKey, queryFn);
```

### UX를 위한 기능 제공

위에서 `웹뷰 환경에서의 사용자 경험 향상을 위해 Window Focus 이벤트 발생 시 서버 상태를 동기화`해야 하는 시나리오가 있다고 가정했을 때, Redux로 비동기 데이터 관리 시에서는 React Component 단에서 Window Focus 이벤트에 `Dispatch Action을 직접 바인딩`하여 구현해야했다. 이런 기능은 `refetchOnWindowFocus` 옵션으로 한방에 구현된다.

```ts
// quires/useTodosQuery.ts
// API 상태를 불러오기 위한 React Query Custom Hook

// ...전략

const useTodosQuery = () => {
  return useQuery(QUERY_KEY, fetcher, { refetchOnWindowFocus: true });
};

export default useTodosQuery;
```

옵션하나만으로도 `Window Focus 이벤트 발생 시 서버 상태 동기화` 시나리오를 구현하고, 직접 이벤트 바인딩 방식보다 유지보수하기 훨씬 좋은 코드가 된다.

## 마무리

React에선 React-Query와 SWR와 같은 비동기 상태관리 라이브러리가 있다. 일반적인 전역 상태관리만 사용한다면 비동기 상태관리를 위한 코드와 인터페이스를 구현하기 위해 보일러플레이트와 규격화된 인터페이스를 제공해줘야하는데 이미 잘 만들어진 라이브러리를 사용하게 된다면, 훨씨더 견고하고 중요한 비지니스 로직에 집중할 수 있게 도와준다는 것을 알 수 있다.

## 출처

- [카카오페이 프론트엔드 개발자들이 React Query를 선택한 이유](https://tech.kakaopay.com/post/react-query-1/#user-content-fn-2)
