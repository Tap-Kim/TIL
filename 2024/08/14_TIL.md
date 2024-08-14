# React 서버 API 알아보기 (`renderToString`, `renderToStaticMarkup`, `renderToNodeStream`)

## 정리

- Node.js의 등장으로 인해 React에서도 SSR이 가능해졌다.
- `renderToString`가 가장 근간이 되는 함수이며, 루트와 React 컴포넌트를 인자로 받아 우리가 알고 있는 HTML로 렌더링이 된다.
- `renderToString`은 streaming 또는 waiting을 지원하지 않는다.
- 서버용 React DOM API가 생각보다 목적이 다양하게 주어진다.
- `renderToNodeStream`이 이전까지 SSR에서 스트리밍 SSR이 되기까지 근간이 되는 모델이었고, React 18로 넘어가면서부터 모든 출력값을 버퍼링 제공하여 실제로 스트리밍 이점을 제공하지 않아도 되어 `renderToPipeableStream`를 사용하기를 권장하고 있다. 주의할 점은 Node.js 전용이라는 점!

### 참고

- 주요 키워드: Server Side Rendering, hydrate, HTML, Stream, chunk
- 관련 기술: React, Node.js

## 무엇을 알았는지

### renderToString

renderToString은 React 서버 사이드 구현시 가장 기초적인 API이며, 최초 페이지를 `HTML 문자열`로 렌더링한다.

`const html = renderToString(reactNode, options)`

```js
import { renderToString } from "react-dom/server";

// React 서버 컴포넌트 렌더링(renderToString)
const html = ReactDOMServer.renderToString(
  React.createElement("div", { id: "root" }, <Component />)
);

// 결과물
<div id="root" data-reactroot="">
  ...
</div>;
```

renderToString은 인수로 주어진 React 컴포넌트를 빠르게 브라우저가 렌더링할 수 있는 HTML을 제공하는데 목적이 있는 함수일 뿐이다. 즉, 클라이언트에서 실행되는 자바스크립트를 포함하지 않고 렌더링 역할을 하지 않는다.

마지막 `data-reactroot`속성은 이후에 자바스크립트를 실행하기 위한 `hydrate` 함수에서 `root를 식별`하는 기준점이다.

### renderToStaticMarkup

```js
import { renderToStaticMarkup } from "react-dom/server";

// React 서버 컴포넌트 렌더링(renderToStaticMarkup)
const html = ReactDOMServer.renderToStaticMarkup(
  React.createElement("div", { id: "root" }, <Component />)
);

// 결과물
<div id="root">...</div>;
```

renderToString와 유사하며, `data-reactroot`인 추가적인 DOM 속성을 만들지 않는다.(순수한 HTML 문자열 반환)

즉, renderToStaticMarkup의 결과로 반환된 컴포넌트는 React에서 제공하는 API를 사용하지 못하며, hydrate를 수행하지 못하는 `순수한 HTML 문자열 반환`만 반환받게 된다. 이는 블로그 글이나 상품 약관 같은 HTML만 필요한 정적인 내용만 필요한 경우 유용하다.

### renderToNodeStream

> 🚨`주의`: 주요 버전 부터는 해당 함수는 deprecated 될 예정이며, [`renderToPipeableStream`](https://react.dev/reference/react-dom/server/renderToPipeableStream)을 사용한다.
>
> 해당 함수는 제거될 예정이지만 `renderToPipeableStream`의 기본 모델이 되기 때문에 정리하고자 한다.

```js
import { renderToNodeStream } from "react-dom/server";

// React 서버 컴포넌트 렌더링(renderToNodeStream)
const stream = renderToNodeStream(<App />);
stream.pipe(response);
```

renderToNodeStream의 결과물은 `Node.js`의 `ReadableStream`이다. 이는 utf-8로 코딩된 `바이트 스트림`으로 Node.js 환경에서만 실행이 가능하다.

`ReadableStream`는 브라우저에서도 사용이 가능한 객체이지만 만드는 과정에서 브라우저에서 사용이 불가능하게 구현되어있다. Why??

#### 스트림

이를 이해하려면 스트림 개념을 이해해야한다. `스트림이란?` 큰 데이터를 다룰 때 데이터를 청크(chunk)로 분할해 조금씩 가져오는 방식이다.

renderToString의 HTML 결과물이 작다면 상관없지만, 큰 문자열을 한번에 메모리에 올려두고 응답을 수행해야 하는 경우 Node.js가 실행되는 서버에 큰 부담이 된다.

이때 스트림을 활용하면 데이터를 청크 단위로 분리해 순차적으로 처리할 수 잇는 장점이 있다.

때문에 널리 알려진 React SSR 프레임워크는 모두 renderToString 대신 renderToNodeStream을 채택했다.

하지만

> 🚨 "React 18부터 이 메서드는 모든 출력값을 버퍼링하므로 실제로 스트리밍 이점을 제공하지 않습니다. 따라서 [`renderToPipeableStream`](https://ko.react.dev/reference/react-dom/server/renderToPipeableStream)으로 마이그레이션할 것을 권장합니다."

#### 번외) React 트리를 HTML로 Node.js Readable Stream에 렌더링하기

```js
import { renderToNodeStream } from "react-dom/server";

// 라우트 핸들러 구문은 백엔드 프레임워크에 따라 다릅니다.
app.use("/", (request, response) => {
  const stream = renderToNodeStream(<App />);
  stream.pipe(response);
});
```

### 출처

- [모던 리액트 Deep Dive](https://m.yes24.com/Goods/Detail/123161563)
- [renderToString](https://react.dev/reference/react-dom/server/renderToString)
- [renderToStaticMarkup](https://react.dev/reference/react-dom/server/renderToStaticMarkup)
- [renderToNodeStream](https://react.dev/reference/react-dom/server/renderToNodeStream)
