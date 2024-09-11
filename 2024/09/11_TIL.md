# CSS 성능 최적화를 위한 이유와 방법(2)

## 정리

- CSS 요소 중에서 폰트가 렌더링 차단에 큰 영향을 끼친다. 용량이 상대적으로 크기도하고, 외부에서 받아오거나 서드 파티를 통해 넘어오는 폰트로 인해 사용자 경험이 나빠질 수 있다.
- font-family와 font-display를 적절히 활용하면서 브라우저별로 지원되는 폰트를 순차적으로 적용되도록 제공하고, 글리프를 사용해 디폴트 폰트를 제공하도록 하자.
- CSS containment을 사용해서 페이지의 부분별로 독립적으로 렌더링 최적화를 진행하자. 이를 통해 브라우저는 DOM의 제한된 부분에 대해 레이아웃, 스타일, 페인트, 크기 또는 이들의 조합을 다시 계산할 수 있다.

### 참고

- 주요 키워드: CSS, font, font-face, web safe fonts, CSS containment, 글리프, font-display
- 관련 기술: CSS preload, rendering block

## 무엇을 알았는지

### `will-change`를 통한 요소 변경 최적화

브라어주는 요소가 실제로 변경되기 전에 최적화 설정이 가능하다. 이런 최적화는 잠재적으로 비용이 많이 드는 작업을 필요하기 전에 수행해서 페이지 응답성을 높인다. [`will-change`](https://developer.mozilla.org/en-US/docs/Web/CSS/will-change) 속성은 요소가 변경될 것으로 예상되는 방식을 브라우저에 힌트한다. 이 힌트는 성능 개선의 최후의 수단으로 사용해야한다. 성능 문제를 예상해서 사용하면 안된다.

```css
.element {
  will-change: opacity, transform;
}
```

### 렌더링 차단 최적화

CSS는 미디어 쿼리로 특정 조건 범위 지정이 간으하다. 이는 렌더링 경로를 최적화하는데 도움이 된다. 미디어 쿼리 기반 CSS를 여러 파일로 분할하면 사용하지 않는 CSS를 다운로드하는 동안 렌더링 차단 방지가 가능하다.

```html
<!--  styles.css 로드 및 구문 분석이 렌더링 차단 -->
<link rel="stylesheet" href="styles.css" />

<!-- 큰 화면에서 mobile.css 로드 및 구문 분석이 렌더링 차단되지 않음 -->
<link
  rel="stylesheet"
  href="mobile.css"
  media="screen and (max-width: 480px)"
/>
```

기본적으로 브라우저는 지정된 각 스탕리 시트가 렌더링 차단이라고 가정한다. 미디어 쿼리화 함께 속성을 추가해서 언제 적용해야하는지 브라우저에게 알려주자. 특정 시나리오에만 적용하는 것을 브라우저가 안다면 스타일 시트를 다운로드하지만 렌더링 차단은 하지 않는다.

### 폰트 성능 개선

폰트는 신중하게 생각하자. 일부 폰트는 MB 단위로 클 수 있따. 이렇게 되면 로딩 속도가 크게 느려지기 때문에, 폰트를 두세 개만 사용하는 것이 좋다.[web safe fonts](https://developer.mozilla.org/en-US/docs/Learn/CSS/Styling_text/Fundamentals#web_safe_fonts)

### 폰트 로딩

폰트는 [`@font-face`](https://developer.mozilla.org/en-US/docs/Web/CSS/@font-face) 사용하여 처음 참조할 때가 아니라 [`font-family`](https://developer.mozilla.org/en-US/docs/Web/CSS/font-family) 속성을 사용하여 요소에 실제로 적용될 때만 로드된다는 점에 유의 하자.

```css
/* 여기선 폰트가 로드되지 않는다. */
@font-face {
  font-family: "Open Sans";
  src: url("OpenSans-Regular-webfont.woff2") format("woff2");
}

h1,
h2,
h3 {
  /* 여기서 폰트가 로드된다. */
  font-family: "Open Sans";
}
```

따라서 중요한 폰트를 미리 로드하여 실제로 필요할 때 더 빨리 사용할 수 있도록 `rel="preload"`를 사용하는 것이 유용할 수 있다.

```html
<link
  rel="preload"
  href="OpenSans-Regular-webfont.woff2"
  as="font"
  type="font/woff2"
  crossorigin
/>
```

`font-family` 선언이 큰 외부 스타일시트 안에 숨겨져 있고 구문 분석 프로세스의 상당히 늦은 단계까지 도달하지 않는 경우 이 방법이 더 유리할 수 있다. 그러나 폰트 파일은 상당히 크기 때문에 너무 많이 미리 로드하면 다른 리소스가 지연될 수 있다는 단점이 있다.

다른 방법으로는 [CSS Font Loading API](https://developer.mozilla.org/en-US/docs/Web/API/CSS_Font_Loading_API)를 사용하면 자바스크립트를 통해 로딩 동작을 정의할 수 있다.

### 필요한 글리프(glyphs)만 불러오기

본문 카피에 사용할 폰트를 선택할 때, 특히 사용자 제작 콘텐츠나 여러 언어로 된 콘텐츠를 다루는 경우 폰트가 사용될 글리프를 확신하기가 더 어렵다.

그러나 특정 글리프 세트(예: 제목 또는 특정 구두점 문자 전용 글리프)를 사용하려는 경우 브라우저에서 다운로드해야 하는 글리프 수를 제한할 수 있다. 필요한 하위 집합만 포함된 폰트 파일을 생성하여 이 작업을 수행할 수 있다. 이를 [하위 집합 설정](https://fonts.google.com/knowledge/glossary/subsetting)이라고 한다. 그런 다음 [`unicode-range`](https://developer.mozilla.org/en-US/docs/Web/CSS/@font-face/unicode-range) `@font-face` 설명자를 사용하여 하위 집합 폰트가 사용되는 시기를 지정할 수 있다. 페이지에서 이 범위의 문자를 사용하지 않는 경우 폰트가 다운로드되지 않는다.

```css
@font-face {
  font-family: "Open Sans";
  src: url("OpenSans-Regular-webfont.woff2") format("woff2");
  unicode-range: U+0025-00FF;
}
```

### `font-display` 설명자를 사용해서 폰트 표시 동작 정의하기

`@font-face` at-rule에 적용되는 [`font-display`](https://developer.mozilla.org/en-US/docs/Web/CSS/@font-face/font-display) 설명자는 브라우저에서 폰트 파일을 로드하고 표시하는 방법을 정의하여 폰트가 로드되거나 로드되지 않을 때 *대체 폰트 텍스트를 표시*할 수 있도록 한다. 이렇게 하면 빈 화면 대신 텍스트가 표시되므로 성능이 향상되지만 스타일이 지정되지 않은 텍스트가 표시되는 단점이 있다.

```css
@font-face {
  font-family: someFont;
  src: url(/path/to/fonts/someFont.woff) format("woff");
  font-weight: 400;
  font-style: normal;
  font-display: fallback;
}
```

### CSS containment을 통한 스타일 재계산 최적화

[CSS containment](https://developer.mozilla.org/en-US/docs/Web/CSS/CSS_containment) 모듈에 정의된 속성을 사용하여 브라우저에 페이지의 여러 부분을 분리하고 서로 독립적으로 렌더링을 최적화하도록 지시할 수 있다. 이를 통해 개별 섹션의 렌더링 성능을 향상시킬 수 있다. 예를 들어 특정 컨테이너가 뷰포트에 표시될 때까지 렌더링하지 않도록 브라우저에 지정할 수 있다.

[contain](https://developer.mozilla.org/en-US/docs/Web/CSS/contain) 속성을 사용하면 작성자가 페이지의 개별 컨테이너에 적용할 [containment types](https://developer.mozilla.org/en-US/docs/Web/CSS/CSS_containment/Using_CSS_containment)을 정확하게 지정할 수 있다. 이를 통해 브라우저는 DOM의 제한된 부분에 대해 레이아웃, 스타일, 페인트, 크기 또는 이들의 조합을 다시 계산할 수 있다.

```css
article {
  contain: content;
}
```

[content-visibility](https://developer.mozilla.org/en-US/docs/Web/CSS/content-visibility) 속성은 작성자가 컨테이너 집합에 강력한 컨테이너 집합을 적용하고 필요할 때까지 브라우저가 해당 컨테이너를 레이아웃 및 렌더링하지 않도록 지정할 수 있는 유용한 단축키다.

컨테이너가 containment의 영향을 받는 동안 컨테이너에 플레이스홀더 크기를 지정할 수 있는 두 번째 속성인 [`contain-intrinsic-size`](https://developer.mozilla.org/en-US/docs/Web/CSS/contain-intrinsic-size)도 사용할 수 있다. 즉, 콘텐츠가 아직 렌더링되지 않은 상태에서도 컨테이너가 공간을 차지하므로 요소가 렌더링되어 표시될 때 스크롤 막대가 이동하거나 끊어질 위험 없이 컨테이너가 마법의 성능을 발휘할 수 있다. 이렇게 하면 콘텐츠가 로드될 때 사용자 경험의 품질이 향상된다.

```css
article {
  content-visibility: auto;
  contain-intrinsic-size: 1000px;
}
```

### 출처

- [MDN - Performance/CSS](https://developer.mozilla.org/en-US/docs/Learn/Performance/CSS)
