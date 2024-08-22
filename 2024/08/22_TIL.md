# Critical Path(주요 경로)의 이해

## 정리

- 주요 렌더링 경로(critical rendering path)란 브라우저에서 렌더링을 시작할때까지 관련된 단계. 페이지를 렌더링하려면 브라우저에 HTML 문서뿐만아니라 렌더링하는데 필요한 모든 리소스가 필요함
- 주요 렌더링 경로 리소스: HTML의 일부, `<head>` 요소의 렌더링 차단 CSS/Javascript
- Render-blocking 리소스(렌더링 차단 리소스): CSS와 같이 중요도가 높은 것으로 간주되는 것들은 브라우저가 처리할 때까지 페이지 렌더링을 중지함.
- Parser-blocking 리소스(파서 차단 리소스): HTML을 계속 파싱하여 브라우저가 실행할 다른 작업을 찾지 못하게하는 리소스.

### 참고

- 주요 키워드: Web Core Vital, browser, Rendering, Optimize, HTML, CSS, Javascript, Parser, TTFB, LCP, FCP, CLS
- 관련 기술: browser rendering, UX

## 무엇을 알았는지

주요 렌더링 경로(critical rendering path)란 브라우저에서 렌더링을 시작할때까지 관련된 단계이다. 페이지를 렌더링하려면 브라우저에 HTML 문서뿐만아니라 렌더링하는데 필요한 모든 리소스가 필요하다.

### Progressive Rendering(점진적 렌더링)

웹은 자연적으로 배포되어있고 설치형 네이티브 앱과 달리 브라우저는 페이지 렌더링에 필요한 모든 리소스가 있는 웹사이트에 의존할 수 밖에 없다. 즉, 브라우저는 점진적 렌더링에 매우 능숙하다.

### 네이티브 / 웹 단계

- 네이티브 앱: 설치 단계 > 실행 단계
- 웹 페이지/웹 앱: 네이티브에 비해 경계가 모호함.(브라우저는 이를 염두하여 특별히 설계 되었다)

브라우저에 일부 HTML만 존재시(css 또는 javascript가 있는 경우) 최대한 빨리 렌더링되면 페이지가 잠시 깨져서 최종 렌더링을 위해 크게 변경된다. 이는 더나은 UX를 제공하는 초기 렌더링에 필요한 리소스를 더 많이 확보할 때까지 처음에 빈 화면을 표시하는 것보다 좋지 않다. 반면 순차적 렌더링을 수행하는 대신 모든 리소스를 대기하면 이또한 좋지 않은 UX를 제공한다.

즉, 브라우저는 최소한의 리소스 수를 알아야하고, 사용자가 오래 기다리지 않는 상태에서 콘텐츠를 표시하면 안된다 이를 수행하기 전에 거치는 단계가 `주요 렌더링 경로(critical rendering path)`이다. 이를 이해하면 필요 이상으로 차단되지 않도록 웹 성능을 개선함과 동시에 초기에 필요한 리소스를 삭제하여 너무 이른 렌더링이 일어나지 않도록 하는 것이 중요하다.

#### 주요 렌더링 경로

![](https://web.dev/static/learn/performance/understanding-the-critical-path/image/fig-1-v2.svg)

- HTML에서 DOM (Document Object Model) 생성
- CSS에서 CSS 개체 모델 (CSSOM) 생성
- DOM 또는 CSSOM을 변경하는 자바스크립트 적용
- DOM 및 CSSOM에서 렌더링 트리 생성
- 페이지에서 스타일 및 레이아웃 작업을 실행하여 어떤 요소가 어디에 적합한지 확인
- 메모리에 있는 요소의 픽셀을 페인트
- 픽셀이 겹치는 경우 합성
- 모든 결과 픽셀을 화면에 물리적으로 페인트

위 렌더링 프로세스는 여러 번 발생한다. 초기 렌더링에 이 프로세스를 호출하지만 페이지 렌더링에 영향을 미치는 리소스가 더 많이 사용되면 브라우저는 이중 일부만 다시 실행하여 업데이트한다.

### 주요 렌더링 경로의 리소스 종류

브라우저는 초기에 렌더링 완료하기 위해서 아래 리소스가 다운로드 될 때까지 기다린다.

- HTML의 일부
- `<head>` 요소의 렌더링 차단 CSS
- `<head>` 요소의 렌더링 차단 JavaScript

요점은 브라우저가 HTML을 `스트리밍`으로 처리한다는 것. HTML의 일부를 가져와 즉시 처리하게 된다.

초기 렌더링 이후에는 브라우저는 일반적으로 아래 작업을 기다리지는 않는다.

- 모든 HTML
- 글꼴
- 이미지
- `<head>` 요소 외부의 _Non-render-blocking JavaScript_ (예: HTML 끝에 배치된 `<script>` 요소)
- `<head>` 요소 외부의 _Non-render-blocking CSS_ 또는 _현재 표시 영역에 적용되지 않는 [media 속성 값이 포함된 CSS](https://developer.mozilla.org/ko/docs/Web/HTML/Element/link#conditionally_loading_resources_with_media_queries)_

글꼴과 이미지는 브라우저 렌더링 중에 채워지는 콘텐츠로 간주되는 경우가 많아 초기 렌더링에는 표함할 필요는 없다. 하지만 텍스트가 숨겨진 상태에서 글꼴이 사용 가능하거나 이미지를 사용할 때까지 초기 렌더링시 빈 공간이 남는 경우가 있따. 이는 CLS 측정시 좋지 않는 점수를 얻게 된다.

`<head>`는 주요 렌더링 경로를 처리시 핵심 역할을 하고, 이 요소의 내부 콘텐츠를 최적화하는 것이 **웹 성능의 핵심**이다. 하지만 `<head>`에 참조된 모든 리소스가 초기 렌더링에 반드시 필요한 것은 아니고 브라우저는 필요한 리소스만 기다린다. 중요한 리소스를 식별하려면 `렌더링 차단(render-blocking), 파서 차단(parser-blocking) CSS, JavaScript`를 이해해야한다.

### 렌더링 차단(render-blocking) 리소스

CSS는 기본적으로 주요 리소스로 판단되기 때문에 렌더링을 차단시키는 요소중 하나이다. 브라우저에서 CSS(`<style>`요소의 인라인 CSS 또는 `link rel=stylesheet href="...">`요소에 의해 지정된 `외부 참조 리소스`)를 확인하면 다운로드 완료가 될 때까지 콘텐츠를 렌더링하지 않는다.

> 참고: CSS는 기본적으로 렌더링을 차단하지만, `<link>` 요소의 media 속성을 변경하여 현재 조건(`<link rel=stylesheet href="..." media=print>`)과 일치하지 않는 값을 지정하면 렌더링을 차단하지 않는 리소스로 전환할 수 있다.

최근 브라우저의 혁신적인 기능으로 인해 [`blocking=render`](https://html.spec.whatwg.org/multipage/urls-and-fetching.html#blocking-attributes) 속성과 같이 개발자는 요소 처리가 될때까지 `<link>, <script>` 또는 `<style>` 요소를 렌더링 차단으로 명시적 표시가 가능하고 그동안에도 파서가 계속하서 문서를 처리하도록 허용할 수 있게 되었다.

### 파서 차단(parser-blocking) 리소스

Javascript 실행시 DOM 또는 CSSOM을 변경할 수 있기 때문에 Javascript는 파서를 차단하여 브라우저가 실행할 다른 작업을 찾지 못하게하는 리소스이다.([asynchronous](https://developer.mozilla.org/ko/docs/Web/HTML/Element/script#async) 또는 [deferred](https://developer.mozilla.org/ko/docs/Web/HTML/Element/script#defer)으로 표시된 경우 제외)

파서를 차단하면 단순히 렌더링 차단 이상으로 비용이 많이 발생할수 있다. 기본 HTML 파서가 차단된 동안에는 브라우저에서 [preload scanner](https://web.dev/articles/preload-scanner?hl=ko)라고하는 보조 HTML 파서를 사용하여 향후 리소스를 다운하여 비용을 줄수 있다. 실제로 HTML 파싱하는 것보다 좋지 않지만 네트워크 함수가 차단된 파서보다 먼저 작동하여 다시 차단될 가능성이 낮다.

### 주요 렌더링 경로 최적화 정의

주요 렌더링 경로를 최적화하려면 TTFB를 수신사하는데 걸리는 시간을 줄이고 렌더링 차단 리소스의 영향을 줄여야한다.

오랬동안 주요 렌더링 경로 자체는 초기 렌더링과 관련이 있었지만 이젠 더 `사용자 중심 측정 항목`([user-centric metrics](https://web.dev/articles/user-centric-performance-metrics))이 등장하여 중요한 렌더링 경로의 엔드포이트가 첫 번째 페인즈인지 그 이후 페인트인지 의문이 생길수 있다.

대안으로는 LCP 또는 FCP의 시간에 집중하는 방법이다. 이경우 반드시 차단하지 않지만 콘텐츠가 포함된 페인트를 렌더링하는데 필요한 리소스를 포함해야할 수 있다.

첫 번째 페인트는 사용자에게 "모든 것"을 렌더링할 수 있는 첫 번째 기회를 측정한다. 이상적으로는 배경 색상이 아닌 의미 있는 것이지만 콘텐츠가 포함되지 않더라도 사용자에게 표시하는 것은 가치가 있다고 판단한다.

### 출처

- [understanding-the-critical-path](https://web.dev/learn/performance/understanding-the-critical-path)
