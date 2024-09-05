# Web Worker의 개요

## 정리

- 단일 스레드로 작업되는 자바스크립트에서 유일하게 멀티 스레딩(Multi Thread) 기능을 허용하는 Web Worker API가 있으며, 이 기능은 메인 스레드에서만 처리할 수 없는 작업시 유용하다.
- Web Worker는 Worker 클래스를 인스턴스화 하여 등록하고, 코드의 위치를 지정(인스턴스 등록)시 브라우저 로드 후 새 스레드를 생성하는데 이 결과를 `Worker Thread`라 한다.
- Web Worker 자체로는 제약사항이 꽤 있따. DOM의 직접 접근, 메시징 파이프라인을 통한 `window` 컨텍스트 접근, 이를 간접을 통해 DOM 접근 또는 API 사용.

### 참고

- 주요 키워드: Web Worker, INP, GPU Thread, Single Thread, Multi Thread
- 관련 기술: Multi Thread, Messaging Pipeline

## 무엇을 알았는지

자바스크립트는 흔히 단일 스레드(Single Thread) 언어로 설명된다. 실제 브라우저에 표시되는 대부분 작업은 단일 스레드이다. 이 작업은 스크립팅, 렌더링 작업, HTML 및 CSS 파싱 등이 포함된다. 실제로 브라우저는 [GPU 스레드](https://www.chromium.org/developers/design-documents/gpu-accelerated-compositing-in-chrome/)와 같이 개발자가 일반적으로 직접 접근할 수 없는 수행을 위해 다른 스레드를 사용한다.

자바스크립트에서 추가 스레드를 등록하고 사용할 수 있는데 멀티 스레딩(Multi Thread)을 허용하는 기능을 [Web Worker API](https://developer.mozilla.org/en-US/docs/Web/API/Web_Workers_API/Using_web_workers)라고 한다. 이 기능은 페이지가 응답하지 않는 긴 작업을 유발하지 않으면서 메인 스레드에서 실행할 수 없는 계산의 비용이 많이 들때 유용하다. 이 작업은 INP에 영향을 줄 수 있으므로 메인 스레드에서 완전히 벗어날 수 있는 작업이 있는지 파악하는 것이 좋다. 이 기능은 메인 스레드에 다른 작업을 위한 더 많은 공간 확보와 사용자 상호 작용 속도를 높일 수있다.

### Web Worker를 시작하는 방법

Web Worker는 [Worker 클래스](https://developer.mozilla.org/ko/docs/Web/API/Worker)를 인스턴스화하여 등록한다. 이러면 Web Worker 코드의 위치를 지정하고 브라우저가 로드한 후 새 스레드를 생성하는데 이 결과를 `Worker Thread`라고 한다.

```js
const myWebWorker = new Worker("/js/my-web-worker.js");
```

워커의 자바스크립트 파일(`my-web-worker.js`)에서 별도의 워커 스레드에서 실행되는 코드를 작성할 수 있다.

### Web Worker의 제약 사항

메인 스레드에서 실행되는 자바스크립트와 달리 Web Worker는 [window 컨텍스트](https://developer.mozilla.org/docs/Web/API/Window)에 직접 접근할 수 없고 제공되는 API에 대한 접근도 제한적이다.

- Web Worker는 DOM에 직접 액세스할 수 없다.
- Web Worker는 메세징 파이프라인을 통해 `window` 컨텍스트와 통신할 수 있고, 이는 Web Worker가 간접적으로 DOM에 접근이 가능하다는 의미이다.
- Web Worker의 범위는 `window`가 아닌 `self`다.
- Web Worker의 범위는 자바스크립트 primitive 및 construct 뿐만아니라 fetch와 같은 API와 상당히 많은 기타 API에 접근할 수 있다.

### Web Worker가 `window`와 대화하는 법

Web Worker는 메시징 파이프라인을 통해 메인 스레드의 `window` 컨텍스트와 통신할 수 있다. 이를 이용하면 메인 스레드와 Web Worker 간에 데이터를 주고받을수 있고 메인 스레드로 데이터를 전송하려면 Web Worker의 컨텍스트(`self`)에서 `message` 이벤트를 설정하면 된다.

```js
// my-web-worker.js
self.addEventListener("message", () => {
  self.postMessage("Hello, window!");
});
```

그런 다음 메인 스레드의 `window` 컨텍스트에 있는 스크립트에서 또 다른 `message` 이벤트를 사용하여 Web Worker 스레드에서 `message`를 받을 수 있다.

```js
// scripts.js
// Web Worker 생성
const myWebWorker = new Worker("/js/my-web-worker.js");

// Web Worker 인스턴스에 메시지를 수신 대기하는 이벤트 리스너 추가합니다
myWebWorker.addEventListener("message", ({ data }) => {
  console.log(data);
});
```

> 참고: 기본적인 작업인 경우 Web Worker의 메시징 파이프라인을 직접 사용해도 좋다. 하지만 일이 복잡해지기 시작하면 이 작업을 단순화할 방법을 찾아야하는데 [Comlink](https://web.dev/articles/off-main-thread#comlink_making_web_workers_less_work)와 같은 추상화를 이용하자.

Web Worker의 메시징 파이프라인은 Web Worker 컨텍스트에서 일종의 탈출구다. 이를 사용하면 Web Worker에서 `window`로 데이터를 전송하여 DOM을 업데이트하거나 메인 스레드에서 수행해야 하는 다른 작업을 수행할 수 있다.

### 출처

- [An overview of web workers](https://web.dev/learn/performance/web-worker-overview)
