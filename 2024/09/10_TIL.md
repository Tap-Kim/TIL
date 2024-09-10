# CSS 성능 최적화를 위한 이유와 방법(1)

## 정리

- 모든 곳에서 CSS 최적화를 적용하지 말것(불필요하고 시간 낭비다). 실제로 최적화가 필요한 곳을 파악하고 "사이트 성능 측정" 도구를 사용하여 필요한 곳에 적용하자.
- 브라우저는 CSS를 다운로드하고 CSSOM을 빌드 후 페이지를 페인팅 할 수 있다. 이 구성을 최적화하기 위해 아래 방법들 중 하나 이상을 수행하자.
- 애니메이션은 UX에 좋은 기술이다. 하지만 무분별하게 사용되면 디바이스의 성능 저해한다.
- 애니메이션은 리소스를 GPU로 위임하여 사용이 가능한 요소들이 있고, 이를 활용하면 훨씬 빠른 속도로 적용이 가능하나 이 또한 무분별하게 작성하면 성능이 떨어질 수 있다.

### 참고

- 주요 키워드: CSS, CSSOM, Repaint/Reflow, universal selector, animation, GPU
- 관련 기술: Stripe CSS, split CSS, asset preload

## 무엇을 알았는지

웹 개발시 CSS로 인해 발생할 수 있는 성능 문제를 완화하려면 CSS 최적화가 필요하고, [렌더링 차단](https://developer.mozilla.org/en-US/docs/Glossary/Render_blocking)을 완화하고 리플로우 횟수를 최소화하기 위해 CSS를 최적화하는 방식 등으로 최적화 기법을 적용해야한다.

### "무엇을 최적화해야하는가?"

CSS 최적화는 모든 프로젝트에 적용하는 것을 불필요하고 시간 낭비다. 실제로 어떤 곳에 성능 최적화가 필요한지 파악해야한다. 이를 위해서는 "사이트의 성능을 측정"을 해야한다.

성능 측정 중 [performance API](https://developer.mozilla.org/en-US/docs/Web/API/Performance_API)이 포함되기도한다. 하지만 가장 좋은 방법을 기본 제공하는 브라우저 [네트워크](https://developer.mozilla.org/en-US/docs/Learn/Performance/Measuring_performance#network_monitor_tools) 및 [성능](https://developer.mozilla.org/en-US/docs/Learn/Performance/Measuring_performance#performance_monitor_tools) 도구와 같은 도구를 사용하여 페이지 로딩 중 어느 부분에서 시간이 오래 걸리고 최적화가 필요한지 확인이 필요하다.

### 렌더링 최적화

브라우저는 특정 렌더링 경로를 따르고, 렌더 트리 생성 후 발생하는 레이아웃 이후에만 페인트가 발생하므로 DOM과 CSSOM 트리 모두 필요하다.

사용자에게 스타일이 지정되지 않은 페이지를 표시한 다음 CSS 스타일이 파싱 된 후에 다시 페인트하면 사용자 경험에 좋지 않다. 그래서 브라우저에서 CSS가 필요하다고 판단할 때까지 CSS는 렌더링이 차단된다.

브라우저는 CSS를 다운로드하고 [CSS 객체 모델(CSSOM)](https://developer.mozilla.org/en-US/docs/Glossary/CSSOM)을 빌드한 후 페이지를 페인팅 할 수 있다.

CSSOM 구성을 최적화하고 페이지 성능 개선을 위해 아래 중 하나를 수행하자

- **불필요한 스타일 제거**: 당연하지만 많은 개발자가 CSS 규칙을 정리하는 것을 잊어버린다. 이는 레이아웃 및 페인팅 중 사용 여부와 관계없이 모든 스타일이 파싱되어 CSS 파싱중 렌더링 속도 개선이 가능하다.
  > 참고: [사용하지 않은 CSS는 어떻게 제거하나요?](https://css-tricks.com/how-do-you-remove-unused-css-from-a-site/)
- **CSS 별도 모듈 분리**: CSS를 모듈화하면 페이지 로드시 필요하지 않는 CSS는 나중에 로드하여 초기 CSS 렌더링 차단 및 로딩 시간을 줄일 수 있다.
  ```html
  <!-- styles.css 로드 및 파싱이 렌더링 차단 -->
  <link rel="stylesheet" href="styles.css" />
  <!-- print.css 로드 및 파싱은 렌더링 차단 대상이 아니다. -->
  <link rel="stylesheet" href="print.css" media="print" />
  <!-- 대형 화면에서 mobile.css 로드 및 파싱이 렌더링 차단되지 않음 -->
  <link
    rel="stylesheet"
    href="mobile.css"
    media="screen and (max-width: 480px)"
  />
  ```
- **CSS 축소 및 압축**: CSS 축소는 코드가 프로덕션에 적용되면 사람이 읽기 쉽도록 파일에서 모든 공백을 제거하는 작업을 포함하며 로딩 시간을 크게 줄일 수 있다. 축소는 일반적으로 빌드 프로세스시 일부 수행한다. 그 외에도 gzip과 같은 압축을 사용하면 호스팅되는 서버가 파일을 제공전에 최적화하여 CSS를 파싱할수 있다.
- **선택기(selector) 단순화**: 복잡한 선택기를 사용하면 파일 크기가 커질 뿐만아니라 선택기의 파싱 시간도 늘어난다.
  ```css
  /* 구체적인 선택기 */
  body div#main-content article.post h2.headline {
    font-size: 24px;
  }
  /* 이것만 있으면 충분 */
  .headline {
    font-size: 24px;
  }
  ```
  선택기를 덜 복잡하고 구체적으로 만드는게 유지 보수에도 좋다. 또한 이해하기 쉽고 덜 구체적일 경우 나중에 스타일 재정의하기가 쉽다.
- **필요 이상으로 많은 요소에 스타일 적용하지 말것**: 일반적으로하는 실수는 [universal selector](https://developer.mozilla.org/en-US/docs/Web/CSS/Universal_selectors)를 사용하여 모든 요소에 스타일을 적용하거나 적어도 필요 이상으로 많은 요소에 스타일을 적용하는 것이다. 이는 대규모 사이트에서 성능에 좋지 않은 영향을 미친다.
  ```css
  /* <body> 내부의 모든 요소를 선택*/
  body * {
    font-size: 14px;
    display: flex;
  }
  ```
  글꼴 크기와 같이 많은 속성은 부모로부터 값을 상속받으므로 모든 곳에 적용할 필요는 없다. 그리고 flexbox와 같은 강력한 도구는 아껴서 사용하자. 모든 곳에서 사용하면 예기치 않은 동작이 발생할 수 있다.
- **CSS Stripe로 이미지 HTTP 요청 줄이기**: [CSS 스트라이프](https://css-tricks.com/css-sprites/)는 사이트에서 사용하려는 여러 개의 작은 이미지를 하나의 이미지 파일에 배치하여 서로 다른 배경 위치 값을 사용하여 표시하는 이미지 덩어리를 사용하는 것이다. 이는 HTTP 요청 횟수를 크게 줄일 수 있다.
- **중요 assets preload 하기**: `rel="preload"`를 사용해서 `<link>` 요소를 asset에 대한 preload로 전환할 수 있따. 여기에는 CSS 파일, 글꼴 및 이미지가 포함된다.

  ```html
  <link rel="preload" href="style.css" as="style" />

  <link
    rel="preload"
    href="ComicSans.woff2"
    as="font"
    type="font/woff2"
    crossorigin
  />

  <link
    rel="preload"
    href="bg-image-wide.png"
    as="image"
    media="(min-width: 601px)"
  />
  ```

  preload를 사용하면 브라우저는 참조된 리소스를 최대한 빨리 가져와 브라우저 캐시에서 사용할 수 있도록 하여 후속 코드에 참조될 때 더 빨리 사용하도록 준비한다. 초반에 접하게 될 우선순위가 높은 리소스를 미리 로드하여 원활한 UX를 제공하는 것이 유용하다.

### 애니메이션 핸들링

애니메이션은 체감 성능을 개선하여 인터페이스가 더 빠르게 느껴지고 사용자가 로딩을 기다리는 동안 진행 상황을 느끼게 해준다. 그러나 애니메이션의 크기가 크고 애니메이션 수가 많으면 처리하는데 더 많은 처리 능력이 필요하여 성능이 저하될 수도 있다.

간단한건 불필요한 애니메이션을 줄이는 것. 저전력 디바이스나 배터리가 제한된 모바일 디바이스 사용시 애니메이션을 끌 수 있는 환경설정을 제공할 수 있다. 자바스크립트를 사용해서 페이지에 애니메이션 적용 및 제어도 가능하다. 또한 애니메이션에 대한 사용자의 OS 수준에 따라 기본 설정의 애니메이션 스타일을 선택적으로 제공 또는 제공하지 않는데 사용할 수 있는 [prefers-reduced-motion](https://developer.mozilla.org/en-US/docs/Web/CSS/@media/prefers-reduced-motion) 미디어 쿼리도 있다.

필수 DOM 애니메이션의 경우 가능하면 자바스크립트 애니메이션보다는 [CSS 애니메이션](https://developer.mozilla.org/en-US/docs/Web/CSS/CSS_animations/Using_CSS_animations)을 사용하는 것이 좋다([웹 애니메이션](https://developer.mozilla.org/en-US/docs/Web/API/Web_Animations_API) API는 자바스크립트를 사용하여 CSS 애니메이션에 직접 연결할 수 있는 방법을 제공한다.)

### 애니메이션을 적용할 속성(property) 선택

애니메이션 성능은 적용하는 속성에 따라 크게 달라진다. 특정 프로퍼티는 애니에미션 적용시 [리플로우](https://developer.mozilla.org/en-US/docs/Glossary/Reflow)(그리고 [리페인트](https://developer.mozilla.org/en-US/docs/Glossary/Repaint))를 트리거하므로 피해야한다.

- `width, height, border, padding` 등 요소의 _치수를 변경_
- `margin, top, bottom, left, right` 등 요소의 _위치를 변경_
- `align-content, align-items, flex` 등 요소의 레이아웃을 변경
- `box-shadow` 등 요소의 형상을 변경하는 시각 효과 추가

최신 브라우저는 전체 페이지가 아닌 변경된 영역만 리페인트할 만큼 똑똑하다. 따라서 애니메이션이 클수록 비용이 더 많이 든다.

아래는 가능하면 리플로우/리페인트를 유발하지 않는 프로퍼티에 애니메이션을 적용하는 것이 좋다.

- [Transforms](https://developer.mozilla.org/en-US/docs/Web/CSS/CSS_transforms)
- [opacity](https://developer.mozilla.org/en-US/docs/Web/CSS/opacity)
- [filter](https://developer.mozilla.org/en-US/docs/Web/CSS/filter)

### GPU에서 애니메이팅하기

성능을 더욱 향상시키려면 작업을 메인 스레드에서 벗어나 디바이스의 GPU로 옮기는 것(또는 compositing이라고 함)을 고려하자. 이 작업은 브라우저가 자동으로 GPU로 전송해서 처리할 특정 유형의 애니메이션을 선택하면 된다.

- [trasnform: translateZ()](https://developer.mozilla.org/en-US/docs/Web/CSS/transform)와 [rotate3d()](https://developer.mozilla.org/en-US/docs/Web/CSS/transform-function/rotate3d)와 같은 3D transform 애니메이션
- [position:fixed](https://developer.mozilla.org/en-US/docs/Web/CSS/position)와 같이 애니메이션이 적용된 다른 특정 속성을 가진 요소
- [will-change](https://developer.mozilla.org/en-US/docs/Web/CSS/will-change)가 적용된 요소
- `<video>`, `<canvas>`, `<iframe>` 등 자체 레이어에서 렌더링 되는 특정 요소

GPU에서 애니메이션을 사용하면 특히 모바일에서 성능이 향상된다. 하지만 애니메이션을 GPU로 옮기는 것이 항상 간단하진 않다. [CSS GPU 애니메이션](https://www.smashingmagazine.com/2016/12/gpu-animation-doing-it-right/)을 읽어보자.

### 출처

- [MDN - Performance/CSS](https://developer.mozilla.org/en-US/docs/Learn/Performance/CSS)
