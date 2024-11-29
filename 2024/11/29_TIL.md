# DOMContentLoaded 이벤트와 load 이벤트 비교

이 두 가지 이벤트 웹 개발에서 DOMContentLoaded 및 load 이벤트는 웹페이지의 일부가 준비되었을 때를 감지하는 데 사용된다. `DOMContentLoaded 이벤트`는 HTML이 완전히 구문 분석되고 DOM이 준비되었을 때 발생하는 반면, `load 이벤트`는 이미지, 스크립트, 스타일시트와 같은 모든 리소스가 완전히 로드될 때까지 기다린다.

## DOMContentLoaded

`DOMContentLoaded 이벤트`는 `HTML 문서`가 완전히 로드되고 파싱될 때 발생한다. 즉, DOM이 준비되었지만 이미지와 스타일시트와 같은 외부 리소스가 완전히 로드되기 전이다. DOM을 사용할 수 있게 되자마자 모든 지연된 스크립트(`<script defer src="…">와 <script type="module">`)가 다운로드를 실행할 수 있다.

> 참고: 여기에는 경쟁 조건이 없으므로 `if` 검사와 `addEventListener()` 호출 사이에 문서가 로드될 수 없다. 자바스크립트에는 실행-완료 시맨틱이 있으므로 이벤트 루프의 특정 틱에 문서가 로드 중이면 다음 사이클까지 로드될 수 없으며, 이때 `doSomething` 핸들러가 이미 첨부되어 실행될 것이다.

```js
function doSomething() {
  console.info("DOM loaded");
}

if (document.readyState === "loading") {
  // Loading hasn't finished yet
  document.addEventListener("DOMContentLoaded", doSomething);
} else {
  // `DOMContentLoaded` has already fired
  doSomething();
}
```

예: 이 예에서는 DOMContentLoaded 이벤트가 페이지가 로드될 때 메시지를 로깅하고, HTML 페이지에 inline CSS를 사용하여 중앙에 이미지를 표시한다.

```html
<html>
  <head>
    <title>Output of DOMContentLoaded and Load events</title>
    <script type="text/javascript">
      document.addEventListener("DOMContentLoaded", function (e) {
        console.log("GfG 페이지가 로드");
      });
    </script>
  </head>
  <body>
    <img
      src="https://media.geeksforgeeks.org/wp-content/uploads/20190328185307/gfg28.png"
      style="align-content: center"
    />
  </body>
</html>
```

## DOMContentLoaded 이벤트를 사용하는 장점

- **더 빠른 실행**: 스크립트는 이미지나 다른 리소스가 로드될 때까지 기다리지 않고 DOM이 준비되는 즉시 실행된다.
- **향상된 사용자 경험**: 페이지 콘텐츠와의 상호작용이 더 빨라진다.
- **효율적인 DOM 조작**: 스크립트가 요소를 사용할 수 있게 되는 즉시 해당 요소를 조작할 수 있도록 보장한다.
- **지연 감소**: 대용량 이미지와 같은 중요하지 않은 리소스를 기다리는 데 따른 불필요한 지연을 방지하는 데 도움이 된다.

## load 이벤트

로드 이벤트는 모든 지연되는 iframe, 이미지, 스크립트, 스타일시트와 같은 모든 종속 리소스를 포함한 `전체 웹페이지가 완전히 로드될 때` 발생한다. 연관된 자바스크립트 함수나 작업을 실행하기 전에 모든 것이 완전히 사용 가능한지 확인한다.

> 참고: `bubbles`가 참으로 초기화되어 있어도 `load`라는 이름의 모든 이벤트는 `window`으로 전파되지 않는다. `window`에서 `load` 이벤트를 포착하려면 해당 `load` 이벤트를 `window`으로 직접 디스패치해야한다.

> 참고: 메인 문서가 로드될 때 발송되는 `load` 이벤트는 `window`에서 발송되지만 대상은 `document`, `path`는 `undefined`이라는 두 가지 변경된 속성을 가진다. 이 두 속성은 레거시 준수로 인해 변경되었다.

예1: 이 예에서는 로드 이벤트가 이미지를 포함한 전체 페이지가 완전히 로드되면 메시지를 로깅하고, HTML 페이지에 중앙에 이미지를 표시한다.

```html
<!DOCTYPE html>
<html>
  <head>
    <title>Load events</title>

    <script type="text/javascript">
      document.addEventListener("load", function (e) {
        console.log("GfG 페이지가 완전히 로드");
      });
      // or
      window.onload = (event) => {
        console.log("page is fully loaded");
      };
    </script>
  </head>

  <body>
    <img
      src="https://media.geeksforgeeks.org/wp-content/uploads/20190328185307/gfg28.png"
      style="align-content: center"
    />
  </body>
</html>
```

예2: "load"라는 textContent가 선언되었기때문에 다른 영역은 이미 가져왔을지라도, "load"가 끝날때까지 나타나지 않는다.

```html
<!DOCTYPE html>
<html>
  <head>
    <script type="text/javascript">
      const log = document.querySelector(".event-log-contents");
      const reload = document.querySelector("#reload");

      reload.addEventListener("click", () => {
        log.textContent = "";
        setTimeout(() => {
          window.location.reload(true);
        }, 200);
      });

      window.addEventListener("load", (event) => {
        log.textContent += "load\n";
      });

      document.addEventListener("readystatechange", (event) => {
        log.textContent += `readystate: ${document.readyState}\n`;
      });

      document.addEventListener("DOMContentLoaded", (event) => {
        log.textContent += `DOMContentLoaded\n`;
      });
    </script>
  </head>

  <body>
    <div class="controls">
      <button id="reload" type="button">Reload</button>
    </div>

    <div class="event-log">
      <label for="eventLog">Event log:</label>
      <textarea
        readonly
        class="event-log-contents"
        rows="8"
        cols="30"
        id="eventLog"
      ></textarea>
    </div>
  </body>
</html>
```

## load 이벤트 사용의 장점

- **전체 콘텐츠 가용성 보장**: 모든 리소스가 완전히 로드된 후 스크립트를 실행한다.
- **오류 방지**: 요소나 리소스 누락으로 인해 발생할 수 있는 잠재적 오류를 줄여준다.
- **미디어가 많은 페이지 지원**: 큰 이미지, 비디오 또는 외부 자산이 있는 페이지에 이상적이다.
- **스크립트 안정성 향상**: 스크립트가 완전히 로드된 모든 리소스에 액세스하고 조작할 수 있도록 한다.

## DOMContentLoaded와 load 이벤트의 비교

| 특징            | DOMContentLoaded                                  | load                                                             |
| --------------- | ------------------------------------------------- | ---------------------------------------------------------------- |
| 발사 시간       | HTML이 완전히 구문 분석되고 DOM이 준비되면 실행.  | 리소스를 포함한 전체 페이지가 완전히 로드되면 발생.              |
| 외부 리소스     | 이미지, 스타일시트 또는 프레임을 기다리지 않는다. | 모든 외부 리소스가 완전히 로드될 때까지 기다린다.                |
| 성능            | 더 빠른 실행, 페이지 상호 작용성 향상             | 모든 리소스가 로딩 완료될 때까지 기다려야 하기 때문에 더 느리다. |
| 사용 사례       | 초기 DOM 조작에 적합                              | 리소스에 의존하는 작업을 처리하는 데 가장 적합                   |
| 일반적인 사용법 | 상호 작용 설정, DOM 관련 스크립트                 | 분석, 로딩 애니메이션, 미디어 처리                               |

## 출처

- [Document: DOMContentLoaded event](https://developer.mozilla.org/en-US/docs/Web/API/Document/DOMContentLoaded_event)
- [Window: load event](https://developer.mozilla.org/en-US/docs/Web/API/Window/load_event)
