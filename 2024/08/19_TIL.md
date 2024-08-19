# React에서 Streaming SSR 비동기 렌더링 살펴보기

## 정리

- React v18 부터 SSR 구현 내부 구조를 살표보면 알수 있듯이 Streaming SSR을 내부적으로 Suspense를 기준으로 매직 리터럴을 식별자로 사용하고, 먼저 작업되는 부분부터 `Streaming HTML`과 `선택적 Hydrating`을 진행하여 완료된 부분을 `replaceContent`로 갈아끼우는 작업을 하는 것을 볼 수 있다.

- 일련의 작업이 가능한 부분은 Node.js에서 `Transfer-Encoding: chunked` 스펙을 적용하여 기존 동기적 처리 밖에 할 수 없었던 `renderToString` API의 한계를 개선해 `renderToReadableStream` API를 사용하여 React에서도 Streaming SSR이 가능해짐을 확인해 볼 수 있다.

### 참고

- 주요 키워드: Server Side Rendering, hydrate, HTML, Stream, chunk
- 관련 기술: React, Node.js

## 무엇을 알았는지

### 싱글 스레드 + 이벤트 루프(with. Node.js)

![](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Ft1.daumcdn.net%2Fcfile%2Ftistory%2F997A063B5AD8B56E05)

위 사진은 Node.js가 싱글 스레드 환경 하에서 콜 스택과 이벤트 루프를 사용하여 동기/비동기 코드를 처리하는 방식을 단순화한 설명이다.

기본적으로 Node.js는 setTimeout, Promise와 같은 비동기 처리는 콜백 함수 형태로 이벤트 루프로 넘겨져 처리된다. 또한 이벤트 루프는 `Phase(Microtask Queue)`와 2개의 `Microtask Queue`를 가지고 있다는 점고 짚어두고 가면 좋다. 동기 처리를 담당하는 `콜 스택`과 비동기 처리를 담당하는 `이벤트 루프`가 서로 상호작용하면서 싱글 스레드 만으로 비동기 처리가 가능하다.

![](https://cdn.builder.io/o/assets%2FYJIGb4i01jvw0SRdL5Bt%2F26702aaabc7845f78315b112015c067d%2Fcompressed?apiKey=YJIGb4i01jvw0SRdL5Bt&token=26702aaabc7845f78315b112015c067d&alt=media&optimized=true)

### `renderToString`의 동기적 구조 문제로 인해 SSR의 초기 모델의 제한된 동시성

최초 SSR의 제한된 동시성 성능 이슈로인해 Node.js 런타임 환경에 부하를 줄수 있고 그 유가 싱글 스레드 환경의 콜스택과 이벤트 루프의 구조적 문제 때문이다.

- Node.js에서 http 모듈로 서버를 띄웠을 때 n개의 요청이 들어올 경우 다른 요청을 대단히 방해하지는 않는다.
- 비동기 처리는 이벤트 루프(`libuv`)로 넘겨져 자바스크립트 런타임의 메인 스레드 이외의 영역에서 처리된 다음, 동기로 실행 가능한 부분만 다시 메인 스레드의 콜스택으로 넘겨져 처리되기 때문이다.
- 그런데 `renderToString()`은 `동기 함수이기 때문에 메인 스레드에서 처리`된다.
- 콜 스택 내에서 실행될 때 많은 시간을 소모할 가능성이 높기 때문에 동시에 많은 요청이 몰리면 메인 스레드를 블락하게 될 수 있다.

### 해결책: StreamingSSR과 선택적 Hydration(React v18)

[New Suspense SSR Architecture in React 18 - Dan abramov](https://github.com/reactwg/react-18/discussions/37)

- `Streaming HTML`을 사용하면 원하는 만큼 일찍 HTML을 전송하기 시작하여 추가 콘텐츠에 대한 HTML을 스트리밍하고 이를 적절한 위치에 배치하는 `<script>` 태그를 함께 전송할 수 있습니다.
- `선택적 Hydration`을 사용하면 나머지 HTML과 JavaScript 코드가 완전히 다운로드되기 전에 가능한 한 빨리 앱에 하이드레이션을 시작할 수 있습니다. 또한 사용자가 상호 작용하는 부분에 우선적으로 수분을 공급하여 즉각적으로 수분이 공급되는 듯한 착각을 불러일으킵니다.

### HTTP 스펙 추가와 3-way handshake

`HTTP/1.1` 스펙의 Header 값 중 `Transfer-Encoding: chunked`로 인해 chunk 단위로 스트리밍 통신이 가능해졌다.(`HTTP/2`부터는 기본적으로 가능함)

[MDN: Transfer-Encoding](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Transfer-Encoding)

![](https://charsyam.wordpress.com/wp-content/uploads/2018/01/3whs.png)

이 스펙은 HTTP 요청을 보낼때 전체 사이즈를 알지 못할때 사용된다. 일반적으로는 Content-Length를 함께 보내지만 스트리밍 요청인 경우에는 불가능하다.

그래서 브라우저에 남은 `Chunk`의 길이가 0이 될 때까지 커넥션을 닫지 않고 기다리면서 TCP/IP 핸드 쉐이크 비용을 절약하게 되었고, 이제부터 React에서 `Streaming SSR` 구현이 가능해졌다.

### Next.js의 App Router에서 Streaming SSR 활용

![](https://saengmotmi.netlify.app/static/cb5ac188e7949b64b4482a4fb6a18283/96220/image-6.png)

사진의 노란색(`received chunk`)가 chunk 단위로 불러와 지는 모습을 볼 수 있다. 이는 Suspense 바운더리가 resolve 된 상태가 아닌 상태기 때문에 fallback 컴포넌트를 보여주게 된다.

```tsx
/** catch */
<Suspense fallback={<div>로딩 중...</div>}>
  {/** try */}
  <Todo />
</Suspense>
```

```tsx
// replaceContent
<script>
    const replaceContent = function(targetID, removeID, newContent) {
        let removeElement = document.getElementById(removeID);
        removeElement.parentNode.removeChild(removeElement);

        let targetElement = document.getElementById(targetID);
        if (targetElement) {
            let prevSibling = targetElement.previousSibling;
            if (newContent) {
                prevSibling.data = "$!";
                targetElement.setAttribute("data-dgst", newContent);
            } else {
                let parentElement = prevSibling.parentNode;
                let nextSibling = prevSibling.nextSibling;
                let balance = 0;
                do {
                    if (nextSibling && 8 === nextSibling.nodeType) {
                        let siblingData = nextSibling.data;
                        if ("/$" === siblingData)
                            if (0 === balance)
                                break;
                            else
                                balance--;
                        else
                            "$" !== siblingData && "$?" !== siblingData && "$!" !== siblingData || balance++;
                    }
                    let nextSiblingCopy = nextSibling.nextSibling;
                    parentElement.removeChild(nextSibling);
                    nextSibling = nextSiblingCopy;
                } while (nextSibling);

                while (removeElement.firstChild) {
                    parentElement.insertBefore(removeElement.firstChild, nextSibling);
                }
                prevSibling.data = "$";
            }

            if(prevSibling._reactRetry) {
                prevSibling._reactRetry();
            }
        }
    };

    replaceContent("B:0", "S:0");
</script>
```

만약 SSR이 동기 함수로 이루어졌다면 "로딩 중"이 아니라 모든 데이터가 채워진 HTML을 받을 것이다(이렇게 되면 첫 응답인 TTFB가 굉장히 느려진다)

이때 첫 번째 `receive chunk` 정보인 `$?` 라는 문자열과 `template#B:0` 태그를 확인해보면 후속 응답으로 들어올 실제 데이터의 `Placeholder` 정보다.

![](https://saengmotmi.netlify.app/static/9bd6367eeb5ab7402ba808822e79b758/da18d/image-10.png)

두 번째 `receive chunk` 정보에선 id 셀렉터가 `S:0`인 정보가 있다.

![](https://saengmotmi.netlify.app/static/2e4c993154e9933ff031f46a1a10309b/1f083/image-8.png)

이는 `replaceContent("B:0", "S:0")` 에서 볼 수 있듯이 id 셀렉터가 `B:0`인 요소(아까 언급했던 placeholder 자리에)와 `S:0` (새로 들어온 데이터)인 HTML 요소를 갈아 끼우는 동작을 한다.

아래 코드를 보면 Placeholder를 생성하는 로직이며

[참고 코드:Github](https://github.com/facebook/react/blob/613e6f5fca3a7a63d115988d6312beb84d37b4db/packages/react-dom-bindings/src/server/ReactFizzConfigDOM.js#L3315-L3355)

```tsx
// Suspense boundaries are encoded as comments.
const startCompletedSuspenseBoundary = stringToPrecomputedChunk("<!--$-->");
const startPendingSuspenseBoundary1 = stringToPrecomputedChunk(
  '<!--$?--><template id="'
);
const startPendingSuspenseBoundary2 = stringToPrecomputedChunk('"></template>');
const startClientRenderedSuspenseBoundary =
  stringToPrecomputedChunk("<!--$!-->");
const endSuspenseBoundary = stringToPrecomputedChunk("<!--/$-->");

const clientRenderedSuspenseBoundaryError1 =
  stringToPrecomputedChunk("<template");
const clientRenderedSuspenseBoundaryErrorAttrInterstitial =
  stringToPrecomputedChunk('"');
const clientRenderedSuspenseBoundaryError1A =
  stringToPrecomputedChunk(' data-dgst="');
const clientRenderedSuspenseBoundaryError1B =
  stringToPrecomputedChunk(' data-msg="');
const clientRenderedSuspenseBoundaryError1C =
  stringToPrecomputedChunk(' data-stck="');
const clientRenderedSuspenseBoundaryError2 =
  stringToPrecomputedChunk("></template>");
```

Suspense 바운더리의 식별의 위한 매직 리터럴이 선언되고 구현하는 모습을 볼 수 있다.

[참고 코드:Github](https://github.com/facebook/react/blob/613e6f5fca3a7a63d115988d6312beb84d37b4db/packages/react-dom-bindings/src/client/ReactFiberConfigDOM.js#L184-L187)

```tsx
const SUSPENSE_START_DATA = "$";
const SUSPENSE_END_DATA = "/$";
const SUSPENSE_PENDING_START_DATA = "$?";
const SUSPENSE_FALLBACK_START_DATA = "$!";
```

### 출처

[Streaming SSR로 비동기 렌더링 하기](https://saengmotmi.netlify.app/react/streaming_ssr/)
