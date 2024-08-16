# React 서버 API 알아보기 - `renderToPipeableStream`

## 정리

- `renderToPipeableStream`는 `Node.js` 스트림으로 렌더링하기 때문 `서버` 환경에서만 작동되는 API이다.
- Node.js환경이 아닌 환경에선 `Web` 스트림 렌더링을 하는 `renderToReadableStream`을 사용할 것
- `renderToPipeableStream` 호출시 출력을 처리할 pipe(res) 메서드와 이를 중단하는 abort() 메서드가 내장되어 있는 스트림을 반환한다.
- 이는 나중에 `<script>` 태그를 통해서 HTML의 Suspense와 스트리밍을 지원하게 된다.

### 참고

- 주요 키워드: Server Side Rendering, hydrate, HTML, Stream, chunk
- 관련 기술: React, Node.js

## 무엇을 알았는지

- v18 이전 SSR 렌더링 방식 (Streaming X)
  ![](https://camo.githubusercontent.com/7bf0e3b75d3036a486bce76b968c98197d94e8c0e5305e2e661608ab1e442bfb/68747470733a2f2f717569702e636f6d2f626c6f622f5963474141416b314234322f39656b30786570614f5a653842764679503244652d773f613d6131796c464577695264317a79476353464a4451676856726161375839334c6c726134303732794c49724d61)

  ![](https://camo.githubusercontent.com/0097afcc0db8e1aaf9b8381bfddd5e32212fff7efcf475e61135e2fa722e8e09/68747470733a2f2f717569702e636f6d2f626c6f622f5963474141416b314234322f534f76496e4f2d73625973566d5166334159372d52413f613d675a6461346957316f5061434668644e36414f48695a396255644e78715373547a7a42326c32686b744a3061)

- v18 이후 SSR 렌더링 방식 (Streaming O)

  v18이후 Streaming UI 지원으로 앱을 chunk 단위로 분할하여 독립적으로 렌더링이 진행이 가능해졌다.

  그에 따라 서버 환경에서 `Suspense`를 사용이 가능해졌으며, 컴포넌트의 로딩 상태가 분기 되어 `html streaming, hydrating`이 진행 가능해졌다.

  ![](https://camo.githubusercontent.com/07b4ae5252a66b946420ae5d70fc800d5c542642607185a8e30ec92e955f9db2/68747470733a2f2f717569702e636f6d2f626c6f622f5963474141416b314234322f704e6550316c4253546261616162726c4c71707178413f613d716d636f563745617955486e6e69433643586771456961564a52637145416f56726b39666e4e564646766361)

### renderToPipeableStream

이전 renderToString과 renderToNodeStream으로는 해결하지못한 스트리밍 UI를 해당(`renderToPipeableStream, renderToReadableStrea`) API로 활용이 가능해졌다.

`renderToPipeableStream`는 `Node.js` 전용 API이며, Deno 및 엣지 런타임과 같은 [Web Stream](https://developer.mozilla.org/ko/docs/Web/API/Streams_API) 환경에선 [renderToReadableStrea](https://ko.react.dev/reference/react-dom/server/renderToReadableStream)을 사용해야한다.

```js
const { pipe, abort } = renderToPipeableStream(reactNode, options);
```

이 API는 Node.js 환경에서 구동되는 메서드이며, React Element를 초기 HTML로 렌더링 된다.

출력을 처리할 pipe(res) 메서드와 이를 중단하는 abort() 메서드가 내장되어 있는 스트림을 반환한다. 이는 나중에 `<script>` 태그를 통해서 HTML의 Suspense와 스트리밍을 지원하게 된다.

- `pipe`: HTML을 제공하는 쓰기 가능한 Node.js 스트림으로 출력한다. 이때 스트리밍을 활성화하기 위해서는 `onShellReady`에서 크롤러와 정적 생성을 사용하려면 `onAllReady`에서 pipe를 호출하면 된다.

- `abort`: AbortController API를 사용하는 것 같고, 서버 렌더링을 강제로 중단하여 나머지는 클라이언트 렌더링이 가능하다.

### React 트리를 HTML로 Node.js 스트림에 렌더링하기

`renderToPipeableStream`을 호출하여 React 트리를 HTML로 Node.js 스트림에 렌더링한다.

```js
import { renderToPipeableStream } from "react-dom/server";

// 경로 핸들러 문법은 백엔드 프레임워크에 따라 다릅니다.
app.use("/", (request, response) => {
  const { pipe } = renderToPipeableStream(<App />, {
    bootstrapScripts: ["/main.js"],
    onShellReady() {
      response.setHeader("content-type", "text/html");
      pipe(response);
    },
  });
});
```

루트 컴포넌트와 함께 `bootstrapScript`에 경로를 제공해야 하는데 루트 컴포넌트는 `<html>`태그를 포함한 전체 문서를 반환해야한다.

```js
export default function App() {
  return (
    <html>
      <head>
        <meta charSet="utf-8" />
        <meta name="viewport" content="width=device-width, initial-scale=1" />
        <link rel="stylesheet" href="/styles.css"></link>
        <title>My app</title>
      </head>
      <body>
        <Router />
      </body>
    </html>
  );
}
```

React는 doctype과 bootstrapScript의 결과를 HTML에 삽입한다.

```html
<!DOCTYPE html>
<html>
  <!-- ... 컴포넌트의 HTML ... -->
</html>
<script src="/main.js" async=""></script>
```

클라이언트에서 bootstrapScript는 [hydrate](https://react.dev/reference/react-dom/client/hydrateRoot#hydrating-an-entire-document)를 호출하여 전체 document를 dydrate해야한다.

```js
import { hydrateRoot } from "react-dom/client";
import App from "./App.js";

hydrateRoot(document, <App />);
```

### 출처
