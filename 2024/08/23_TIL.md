# 브라우저 리소스 로딩 최적화

## 정리

- 페이지 로드 시 HTML 내에서 많은 리소스가 참조되어 CSS로 인한 스타일과 레이아웃, 상호작용까지 갖춪 Javascript를 다루게 되는데 이 모든 리소스가 페이지 로드 시간에 영향을 주게 된다. 어떤 방식들이 리소스를 들게하는지, 확인하고 개선하는 방법을 보자.(렌더링 및 파서 차단)
- `preload 스캐너`: 보조 HTML 형식의 브라우저 최적화 방식이다. `<img>` 요소에 지정된 CSS와 Javascript 같은 리소스를 브라우저에서 HTML이 차단도니 경우에도 미리 가져와서 처리가 가능하여 UX에 큰 이점을 준다.
- `CSS`: `CSS - Minification(축소), 사용하지 않는 CSS 삭제, CSS @import 선언 피하기, inline critical CSS` 방식으로 렌더링 차단을 최적화하고 UX를 개선하는 방법을 알수 있다.
- `Javascript`: `async와 defer를 사용하여 동기 및 지연처리, Javascript - Minification(축소)`를 활용하여 렌더링 차단을 우회하거나 불필요한 리소스를 최소화하여 UX 개선에 큰 도움을 줄수 있다.

### 참고

- 주요 키워드: preload, render-blocking, parser-blocking, csr, @import, minification, javascript, css, html
- 관련 기술: preload, browser rendering, UX

## 무엇을 알았는지

앞서 [Critical Path(주요 경로)의 이해](https://github.com/Tap-Kim/TIL/blob/main/2024/08/22_TIL.md)에서 렌더링 및 파서 차단에 대해 종류와 역할을 이해했다. 이번에는 어떻게하면 브라우저 최적화를 위해 렌더링 및 파서를 차단을 막고, 어떤 이유로 인해 최적화에 영향을 주고 개선하는지 공부한다.

### Blocking(차단)

#### 렌더링 차단

CSS는 렌더링 차단 리소스다. 따라서 CSSOM이 요청될 때까지 브라우저는 콘텐츠를 차단하여 기다리게된다. 이는 [FOCU(Flash of unstyled content)](https://en.wikipedia.org/wiki/Flash_of_unstyled_content)를 차단하게 되는데 UX가 좋지 않다.

![](https://web.dev/static/learn/performance/optimize-resource-loading/video/fouc.webm?hl=ko)

#### 파서 차단

파서 차단은 `<script>`와 같은 HTML 파서를 차단하는 리소스고 `async`, `defer`를 사용한다. `<script>`인 경우 브라우저에서 스크립트를 평가하고 실행하는데 계속 파싱한다. 이건 의도된 것이고 스크립트가 DOM이 생성중인 동안 DOM을 수정하거나 접근할 수 없다.

```html
<!-- This is a parser-blocking script: -->
<script src="/script.js"></script>
```

### preload 스캐너

이는 보조 HTML 형식의 브라우저 최적화 방식이다. `<img>` 요소에 지정된 CSS와 Javascript 같은 리소스를 브라우저에서 HTML이 차단도니 경우에도 미리 가져와서 처리가 가능하다.

preload 스캐너가 찾을 수 없는 경우는 모두 늦게 검색된 리소스이므로 preload의 이점을 누리지 못한다.

- CSS에 `background-image`로 참조한 이미지는 검색 불가
- Javascript 또는 `dynamic import()`를 사용하여 로드된 모듈을 사용하여 DOM에 삽입된 `<script>` 요소 마크업 형식의 동적으로 로드된 스크립트이다.
- JavaScript를 사용하여 클라이언트에서 렌더링된 HTML. 이러한 마크업은 JavaScript 리소스의 `문자열`에 포함되어 있으며 사전 로드 스캐너로 검색할 수 없다.
- CSS `@import` 선언된 마크업

### CSS

앞서 설명한 것처럼 렌더링 차단 요소이며 전체 페이지 로드에 영향을 미치는 리소스이다. 이를 개선하기 위해 4가지 방식을 소개한다.

#### CSS - Minification(축소)

파일 크기 자체를 축소하면 리소스가 줄어들어 더 빠른 다운이 된다. 주로 콘텐츠를 삭제하는 방식이고 아래와 같이 진행한다.

```css
/* 축소 전 CSS: */

/* Heading 1 */
h1 {
  font-size: 2em;
  color: #000000;
}

/* Heading 2 */
h2 {
  font-size: 1.5em;
  color: #000000;
}
```

```js
/* 축소 후 CSS: */
h1,h2{color:#000}h1{font-size:2em}h2{font-size:1.5em}
```

이 방식은 FCP, LCP까지 개선 가능하며, 번들러에서 자동으로 수행이 가능하다.

#### 사용하지 않는 CSS 삭제

콘텐츠 렌더링전 브라우저는 모든 스타일 시트를 다운로드하고 파싱(구문을 분석)해야한다. 완료에 필요한 시간은 사용되지 않는 스타일도 포함되어 모든 CSS를 단일 파일로 결합하는 번들러를 사용시 더 많은 CSS를 다운할 수 있다.

개발자 도구에서 [Coverage tool](https://developer.chrome.com/docs/devtools/css/reference/#coverage) 도구를 사용하자.

![](https://web.dev/static/learn/performance/optimize-resource-loading/image/fig-1_856.png)

> 참고: 이는 현재 페이지에서 사용되지 않는 CSS 및 Javascript를 감지하는데 유용하다. 이 도구를 사용하면 렌더링을 지연시키는 요소를 확인하여 해결할 수 있을 것이다.

#### CSS @import 선언 피하기

`@import` 선언은 편리하지만 피해야한다. `<link>` 요소가 작동하는 방식과 마찬가지로 스타일 시트 내부에서 외부의 CSS를 가져올 수 있게 된다. 차이점은 `<link>` 요소가 HTML 응답에 포함되므로 CSS 보다 `@import`로 선언한 것을 훨씬 빨리 발견하기 때문에 미리 로드하여 렌더링 차단 리소스가 더욱 늦게 발견 될 수 있게 된다.

```css
/* 이렇게 쓰지 말것 */
@import url("style.css");
```

```html
<!-- 이렇게 쓰자 -->
<link rel="stylesheet" href="style.css" />
```

> 참고: `@import`를 사용해야하는 경우(cascade layers 또는 서드 파티 스타일 시트) 가져온 스타일 시트에 preload 지시문을 사용하여 지연을 완화라 수 있다. 또한 SAAS나 LESS와 같은 CSS 전처리기는 개발자 환경 개선읠 일환으로 `@import` 구문을 사용해서 소스 파일을 분리하고 모듈화하는 것이 일반적이다. 하지만 CSS 전처리기에서 `@import`를 만나면 `참조된 파일이 번들로 묶여 단일 스타일 시트로 작성`되어 일반 CSS에서 `@import`로 인한 연속 요청에 대한 불이익을 피할 수 있다.

#### inline critical CSS

CSS 리소스 시간으로 FCP가 증가할 수 있다. 문서 `<head>`에 중요한 스타일을 인라인 처리하면 CSS 리소스에 대한 네트워크 요청이 제거되고 올바르게 수행하면 사용자 브라우저 캐시가 준비되지 않은 상태에서 초기 화면(스크롤 없이 볼 수 있는 부분 포함) 노출에 대한 초기 로드 시간을 개선할 수 있다. 나머지 CSS는 비동기적으로 로드하거나 `<body>` 요소 끝에 추가할 수 있다.

```html
<head>
  <title>Page Title</title>
  <!-- ... -->
  <style>
    h1,
    h2 {
      color: #000;
    }
    h1 {
      font-size: 2em;
    }
    h2 {
      font-size: 1.5em;
    }
  </style>
</head>
<body>
  <!-- Other page markup... -->
  <link rel="stylesheet" href="non-critical.css" />
</body>
```

단점은 많은 양의 CSS를 인라인 처리하게 되면 초기 HTML 응답에 더 많은 바이트가 추가됩니다. 이는 HTML 리소스를 오래 또는 전혀 캐시가 안되는 경우가 많기 때문에 외부 스타일시트에서 동일한 CSS를 사용할 수 있는 후속 페이지에는 인라인 CSS가 캐시되지 않는다. 상황에 맞게 쓰도록 하자.

### Javascript

#### 렌더링 차단 Javascript

`defer`나 `async` 속성 없이 `<script>` 요소를 로드하면 스크립트가 다운로드, 파싱, 실행될 때까지 *파싱 및 렌더링을 차단*한다. 마찬가지로 인라인 스크립트는 스크립트가 파싱되고 실행될 때까지 파서를 차단한다.

#### async와 defer 비교

`defer`나 `async` 사용시 외부 스크립트를 HTML 파서에서 차단하지않고 로드가 되고 유형이 `"module"`인 스크립트(인라인 스크립트 포함)은 자동 지연이 된다.

![](https://web.dev/static/learn/performance/optimize-resource-loading/image/fig-2.svg)

차이를 보게 되면

- `async`로 로드된 스크립트는 다운로드 즉시 파싱되어 실행한다.
- `defer`로 로드된 스크립트는 HTML 문서 파싱이 완료될 때 실행하며, 브라우저의 `DOMContentLoaded` 이벤트와 동시에 발생한다.
- `async`는 순서 없이 실행될 수 있지만, `defer`는 마크업에 표시된 순서대로 실행된다.

> 참고: `type="module"` 속성은 기본적으로 지연되지만 DOM에 `<script>` 마크업을 삽입하여 로드된 스크립트 사용하면 `async` 스크립트처럼 작동한다.

#### CSR(Client-Side-Rendering)

일반적으로는 콘텐츠 렌더링하거나 페이지의 LCP 요소를 렌더링할때 Javascript를 사용하면 안된다. 이러한 방식을 CSR이라하며 SPA에선 광범위하게 사용되는 기술이다.

이렇게 렌더링된 마크업은 리소스 검색이 안되기 때문에 preload 스캐너를 우회하게된다. 이로 인해 LCP 이미지와 같은 중요 리소스 다운로드가 지연될 수 있고, 브라우저는 스크립트가 실행하고 DOM에 추가 되어야 LCP 이미지를 다운 한다. 결과적으로 스크립트는 검색, 다운로드 및 파싱이 완료된 후에만 실행 가능하다. 이를 [critical-request-chains](https://developer.chrome.com/en/docs/lighthouse/performance/critical-request-chains/)이라고하고 피해야하는 방식이다.

또한 이렇게 마크업 렌더링을 진행하면 탐색 요청에 대한 응답이 서버로 다운된 마크업보다 오래걸리게 될 가능성이 높다. 페이지의 DOM이 매우 커서 javascript가 DOM을 수정할 때 상당한 렌더링 작업이 발생할 경우 인터렉션 시간에 부정적인 영향을 끼칠수 있게 된다.

#### Javascript - Minification(축소)

CSS와 마찬가지로 크기를 축소하면 다운로드 속도가 빨라져 자바스크립트 파싱 및 컴파일 프로세스로 더 빨리 이동이 가능하다. 이는 CSS 축소보다 더 한 단계 나아가는 방식이다.

```js
// 축소전 JavaScript 코드:
export function injectScript() {
  const scriptElement = document.createElement("script");
  scriptElement.src = "/js/scripts.js";
  scriptElement.type = "module";

  document.body.appendChild(scriptElement);
}
```

```c
// 축소후 JavaScript 운영 코드:
export function injectScript(){const t=document.createElement("script");t.src="/js/scripts.js",t.type="module",document.body.appendChild(t)}
```

스크립트 명칭이 단축되고 운영에 영향을 주지 않으면서 상당한 절감 효과를 얻을 수 있다. 이또한 번들러를 사용하여 프로덕션 빌드시 자동으로 번들링 할 수 있다.

### 출처

- [Optimize resource loading](https://web.dev/learn/performance/optimize-resource-loading)
