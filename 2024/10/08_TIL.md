# 400줄의 코드로 나만의 React.js 구축하기

## JSX와 createElement

JSX를 사용하면 DOM을 설명하고 자바스크립트 로직을 쉽게 적용할 수 있다. 브라우저는 JSX를 이해하지 못하므로, 이를 이해할 수 있는 자바스크립트 컴파일이 된다.

![](https://miro.medium.com/v2/resize:fit:700/1*XoekqpARAW_Nt6PsSiseew.png)

babel를 사용했지만 대부분 비슷할 것이고, 터미널에 `React.createElement`를 호출하는 것을 볼 수 있다.

1. **type**: 현재 노듸 유형을 나타낸다(예: div)
2. **config**: 현재 엘리먼트 노드의 속성을 나타낸다(예: { id: "test" })
3. **children**: 자식 엘리멘트로 여러 엘리먼트, 간단한 텍스트 또는 `React.createElement`로 생성된 여러 노드일 수 있다.

React 18 이전에는 JSX를 작성하기 위해 `import React from 'react`를 해야했지만 18부터는 작성하지 않아도 되나 여전히 React.createElement가 호출된다.

![](https://media.beehiiv.com/cdn-cgi/image/fit=scale-down,format=auto,onerror=redirect,quality=80/uploads/asset/file/27d9b5fd-4d33-471d-b0a9-11321ba99ed0/image.png?t=1715234057)

단순화한 React 구현의 경우 JSX를 `React.createElement` 구현으로 직접 컴파일하도록 [Vite](https://github.com/ZacharyL2/mini-react/blob/master/vite.config.ts?utm_source=webdeveloper.beehiiv.com&utm_medium=referral&utm_campaign=build-your-own-react-js-in-400-lines-of-code#L9-L13)를 `withreact({ jsxRuntime: 'classic' })`로 구성해야 한다.

구현한다면.([코드 참조](https://github.com/ZacharyL2/mini-react/blob/master/src/mini-react.ts?utm_source=webdeveloper.beehiiv.com&utm_medium=referral&utm_campaign=build-your-own-react-js-in-400-lines-of-code#L82-L107))

```js
// 텍스트 요소는 특별한 처리가 필요하다.
const createTextElement = (text: string): VirtualElement => ({
  type: "TEXT",
  props: {
    nodeValue: text,
  },
});

// 커스텀 자바스크립트 데이터 구조를 생성한다.
const createElement = (
  type: VirtualElementType,
  props: Record<string, unknown> = {},
  ...child: (unknown | VirtualElement)[]
): VirtualElement => {
  const children = child.map((c) =>
    isVirtualElement(c) ? c : createTextElement(String(c))
  );

  return {
    type,
    props: {
      ...props,
      children,
    },
  };
};
```

## Render

앞서 만든 데이터 구조를 기반으로 렌더링 함수의 단수화 버전을 구현하여 JSX를 실제 DOM에 렌더링한다.

```js
// mini-react.js
// DOM 속성을 수정, 간단하게 하기 위해 이전 속성을 모두 제거 후 다음 속성을 추가.
const updateDOM = (DOM, prevProps, nextProps) => {
  const defaultPropKeys = "children";

  for (const [removePropKey, removePropValue] of Object.entries(prevProps)) {
    if (removePropKey.startsWith("on")) {
      DOM.removeEventListener(
        removePropKey.substr(2).toLowerCase(),
        removePropValue
      );
    } else if (removePropKey !== defaultPropKeys) {
      DOM[removePropKey] = "";
    }
  }

  for (const [addPropKey, addPropValue] of Object.entries(nextProps)) {
    if (addPropKey.startsWith("on")) {
      DOM.addEventListener(addPropKey.substr(2).toLowerCase(), addPropValue);
    } else if (addPropKey !== defaultPropKeys) {
      DOM[addPropKey] = addPropValue;
    }
  }
};

// 노드 유형에 따라 DOM을 생성
const createDOM = (fiberNode) => {
  const { type, props } = fiberNode;
  let DOM = null;

  if (type === "TEXT") {
    DOM = document.createTextNode("");
  } else if (typeof type === "string") {
    DOM = document.createElement(type);
  }

  // 생성 후 props을 기반으로 속성을 업데이트
  if (DOM !== null) {
    updateDOM(DOM, {}, props);
  }

  return DOM;
};

const render = (element, container) => {
  const DOM = createDOM(element);
  if (Array.isArray(element.props.children)) {
    for (const child of element.props.children) {
      render(child, DOM);
    }
  }

  container.appendChild(DOM);
};

export default {
  render,
  createElement,
};
```

```js
// main.jsx
import React from "./mini-react";

const App = (
  <div id="test">
    <h1>Hello</h1>
  </div>
);

// eslint-disable-next-line react/no-deprecated
React.render(App, document.getElementById("root"));
```

현재 JSX는 한 번만 렌더링하므로 상태 업데이트 처리 되지 않는다. [구현 링크](https://stackblitz.com/edit/vitejs-vite-x1hklu?file=src%2Fmain.jsx&utm_campaign=build-your-own-react-js-in-400-lines-of-code&utm_medium=referral&utm_source=webdeveloper.beehiiv.com)

## Fiber 아키텍처와 동시성 모드

Fiber 아키텍처와 동시성 모드는 주로 완전한 요소 트리가 재귀되면 중단할 수 없어서 메인 스레드가 장시간 차단될 수 있는 문제를 해결하기 위해 개발되어있다. 사용자 입력이나 애니메이션과 같이 우선순위가 높은 task는 적시에 처리되지 않을 수 있다.

![](https://media.beehiiv.com/cdn-cgi/image/fit=scale-down,format=auto,onerror=redirect,quality=80/uploads/asset/file/bbb170d1-b24c-432b-90ff-8ab471a934f2/image.png?t=1715916980)

소스 코드에서 task는 작은 단위로 나뉜다. 브라우저가 idle 상태일 때마다 작은 task 단위를 처리해서 우선순위가 높은 task에 즉시 응답할 수 있도록 메인 스레드의 제어권을 포기한다. task의 모든 작은 단위가 완료되면 결과가 실제 DOM에 매핑된다.

![](https://media.beehiiv.com/cdn-cgi/image/fit=scale-down,format=auto,onerror=redirect,quality=80/uploads/asset/file/0fa3535b-709d-4eb5-9f28-757fef09882f/image.png?t=1715917058)

그리고 실제 React에는 useTransition 또는 useDeferredValue와 같은 제공된 API를 사용해서 업데이트의 우선순위를 명시적으로 낮출 수 있다.

요약하자면 두 가지 핵심으로 메인 스레드를 포기하는 방법과 작업을 관리 가능한 단위로 세분화하는 방법이다.

## requestIdleCallback

`requestIdleCallback`은 브라우저가 idle 상태일 때 콜백을 실행하는 실험적인 API다. 아직 [모든 브라우저](https://caniuse.com/requestidlecallback)에선 지원되진 않는다. React에선 작업 우선순위 업데이트 등 requestIdleCallback 보다 더 복잡한 스케줄링 로직이 있는 [스케줄러 패키지](https://github.com/facebook/react/tree/main/packages/scheduler?utm_source=webdeveloper.beehiiv.com&utm_medium=referral&utm_campaign=build-your-own-react-js-in-400-lines-of-code)에서 사용된다.

하지만 여기서는 비동기 중단성만 고려하기 때문에 이것이 React를 모방한 기본 구현이다.

```js
// 향상된 requestIdleCallback.
((global: Window) => {
  const id = 1;
  const fps = 1e3 / 60;
  let frameDeadline: number;
  let pendingCallback: IdleRequestCallback;
  const channel = new MessageChannel();
  const timeRemaining = () => frameDeadline - window.performance.now();

  const deadline = {
    didTimeout: false,
    timeRemaining,
  };

  channel.port2.onmessage = () => {
    if (typeof pendingCallback === "function") {
      pendingCallback(deadline);
    }
  };

  global.requestIdleCallback = (callback: IdleRequestCallback) => {
    global.requestAnimationFrame((frameTime) => {
      frameDeadline = frameTime + fps;
      pendingCallback = callback;
      channel.port1.postMessage(null);
    });
    return id;
  };
})(window);
```

### 왜 MessageChannel인가?

주로 매크로 태스크를 사용해서 각 단위 task를 처리한다. 왜 매크로 태스크인가?

매크로 태스크를 사용하면 메인 스레드의 제어권을 포기하고 브라우저가 idle 기간 동안 DOM을 업데이트하거나 이벤트를 수신할 수 있도록 해야한다. 브라우저가 별도의 task로 DOM을 업데이트하므로 이때 자바스크립트는 실행되지 않는다.

메인 스레드는 한 번에 한 가지 작업(자바스크립트 실행 또는 DOM 계산, 스타일 계산, 입력 이벤트 처리 등)만이 실행된다. 그러나 마이크로 태스크(예: `Promise.then`)는 메인 스레드에 대한 제어권을 포기하지 않는다.

### 왜 setTimeout을 사용하지 않는지?

최신 브라우저는 중첩된 setTimeout 호출이 5회 이상 발생하면 차단으로 간주하고 최소 지연 시간을 4ms로 설정하므로 충분히 정확하지 않기 때문이다.

## 알고리즘

React는 계쏙 발전하고 있고, 이 알고리즘은 최신이 아닐 수 있긴하지만 기본을 이해하긴 충분하다.

![](https://miro.medium.com/v2/resize:fit:694/1*LkXFsSMU8Q82MD9T39bfzA.png)

React에는 각 task 단위를 Fiber 노드라고 부른다. 이것들은 링크된 목록과 같은 구조를 사용해서 서로 연결된다.

1. **child**: 부모 노드에서 첫 번째 자식 요소에 대한 포인터다.
2. **return/parent**: 모든 자식 요소에는 부모 요소로 돌아가는 포인터가 있다.
3. **sibling**: 첫 번째 자식 요소에서 다음 sibling 요소로 이동된다.

[구체적인 구현 코드](https://github.com/ZacharyL2/mini-react/blob/master/src/mini-react.ts?utm_source=webdeveloper.beehiiv.com&utm_medium=referral&utm_campaign=build-your-own-react-js-in-400-lines-of-code#L162-L382)

렌더링 로직을 확장해서 호출 시퀀스를 `workLoop -> performUnitOfWork -> reconcileChildren -> commitRoot`

1. **workLoop**: `requestIdleCallback`을 지속적으로 호출해서 idle 시간을 가져온다. 현재 idle 상태고 실행할 단위 task가 잇는 경우 각 단위 task를 실행한다.
2. **performUnitOfWork**: 수행된 특정 단위 task이다. 링크된 목록 아이디어의 구체화이다. 구체적으로, 한 번에 하나의 Fiber 노드만 처리되고 처리할 다음 노드가 반환된다.
3. **reconcileChildren**: 현재 Fiber 노드(실제로는 가상 DOM 비교)를 조정하고 변경 사항을 기록한다. 이제 각 Fiber 노드에서 직접 수정하고 저장한 것을 볼 수 있는데, 이는 자바스크립트 객체에 대한 수정일 뿐 실제 DOM을 건드리지 않기 때문이다.
4. **commitRoot**: 현재 업데이트가 필요하고(`wipRoot`에 따라) 처리할 다음 단위 task가 없는 경우(`!nextUnitOfWork`에 따라), 이는 가상 변경 사항을 실제 DOM에 매핑해야 함을 의미한다. `commitRoot`는 Fiber 노드의 변경 사항에 따라 실제 DOM을 수정하는 것이다.

이를 통해 인터럽트 가능한 DOM 업데이트를 위해 Fiber 아키텍처를 실제로 사용할 수 있지만 여전히 트리거가 부족하다.

## 트리거 업데이트

React에서 일반적인 트리거는 가장 기본적인 업데이트 메커니즘인 `useState`이다. 이를 구현하여 Fiber 엔진을 보자.

```js
// 훅을 Fiber 노드와 연결한다.
function useState<S>(initState: S): [S, (value: S) => void] {
  const fiberNode: FiberNode<S> = wipFiber;
  const hook: {
    state: S,
    queue: S[],
  } = fiberNode?.alternate?.hooks
    ? fiberNode.alternate.hooks[hookIndex]
    : {
        state: initState,
        queue: [],
      };

  while (hook.queue.length) {
    let newState = hook.queue.shift();
    if (isPlainObject(hook.state) && isPlainObject(newState)) {
      newState = { ...hook.state, ...newState };
    }
    if (isDef(newState)) {
      hook.state = newState;
    }
  }

  if (typeof fiberNode.hooks === "undefined") {
    fiberNode.hooks = [];
  }

  fiberNode.hooks.push(hook);
  hookIndex += 1;

  const setState = (value: S) => {
    hook.queue.push(value);
    if (currentRoot) {
      wipRoot = {
        type: currentRoot.type,
        dom: currentRoot.dom,
        props: currentRoot.props,
        alternate: currentRoot,
      };
      nextUnitOfWork = wipRoot;
      deletions = [];
      currentRoot = null;
    }
  };

  return [hook.state, setState];
}
```

Fiber 노드에서 훅의 상태를 유지하고 큐를 통해 상태를 수정한다. 이때 React 훅 호출 순서가 변경되지 않아야하는 이유도 확인할 수 있다.

## 결론

종속성 없이 비동기 및 중단 없는 업데이트를 지원하는 최소한의 React 모델을 구현했고, 주석을 제거하면 400 줄 미만임을 확인할 수 있다.

## 출처

- [build-react-400-lines-code](https://webdeveloper.beehiiv.com/p/build-react-400-lines-code)
