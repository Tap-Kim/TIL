# CSS 미디어 쿼리 가이드(1) - 미디어 쿼리에 사용되는 다양한 요소들

미디어 쿼리는 사용자 기기, 브라우저 또는 시스템 설정에 대한 조건들의 집합에 따라 웹사이트나 앱의 모양(동작까지) 적용가능하다.

가장 일반적인 방법으로 특정 `뷰포트 범위`를 타겟팅하고 사용자 지정 스타일을 적용하는 방법이다.

```css
/* 브라우저가 최소 600px 이상인 경우 */
@media screen and (min-width: 600px) {
  .element {
    ...;
  }
}
```

뷰포트 너비 외에도 타겟팅할 수 있는 많은 것들이 있다. 여기에는 화면 해상도, 기기 방향, OS 환경 설정 또는 우리가 쿼리하고 콘텐츠 스타일을 지정하는데 사용하는 모든 것들 중에 있다.

> 참고: [스니펫 컬렉션](https://css-tricks.com/snippets/css/media-queries-for-standard-devices/): 표준 기기의 뷰포트 기반으로 한 미디어 쿼리 집합들

## 미디어 쿼리 사용

### HTML

`<head>`에 `<link>` 요소. 이 예로 브라우저에 다른 뷰포트 크기에서 다른 스타일시트를 사용하고 싶을 때.

```html
<html>
  <head>
    <!-- Served to all users -->
    <link rel="stylesheet" href="all.css" media="all" />
    <!-- Served to screens that are at least 20em wide -->
    <link rel="stylesheet" href="small.css" media="(min-width: 20em)" />
    <!-- Served to screens that are at least 64em wide -->
    <link rel="stylesheet" href="medium.css" media="(min-width: 64em)" />
    <!-- Served to screens that are at least 90em wide -->
    <link rel="stylesheet" href="large.css" media="(min-width: 90em)" />
    <!-- Served to screens that are at least 120em wide -->
    <link rel="stylesheet" href="extra-large.css" media="(min-width: 120em)" />
    <!-- Served to print media, like printers -->
    <link rel="stylesheet" href="print.css" media="print" />
  </head>
  <!-- ... -->
</html>
```

이렇게 스타일을 분할하면 필요한 기기에 다운로드 제공과 사이트 성능을 미세 조정할 수 있는 좋은 방법이다.(그저 낮은 로딩 수선순위 수준을 할당하긴한다)

반응형 이미지 또한 `<source>` 요소에 쿼리사용이 가능하여 `<picture>` 요소에 브라우저가 이미지 옵션 세트에서 어떤 버전의 이미지를 사용하는지 알려준다.

```html
<picture>
  <!-- Use this image if the screen is at least 800px wide -->
  <source srcset="cat-landscape.png" media="(min-width: 800px)" />
  <!-- Use this image if the screen is at least 600px wide -->
  <source srcset="cat-cropped.png" media="(min-width: 600px)" />

  <!-- Use this image if nothing matches -->
  <img src="cat.png" alt="A calico cat with dark aviator sunglasses." />
</picture>
```

`<style>` 태그에서도 직접 사용가능하다.

```html
<style>
  p {
    background-color: blue;
    color: white;
  }
</style>

<style media="all and (max-width: 500px)">
  p {
    background-color: yellow;
    color: blue;
  }
</style>
```

### CSS

CSS에서 사용하는 것이 가장 일반적이고 브라우저에서 해당 조건과 일치할 때 스타일 집합을 언제, 어디서 적용할지 조건으로 요소를 감싸는 `@media` 규칙이 스타일시트에 적용된다.

```css
/* 뷰포트 320px에서 480px 사이*/
@media only screen and (min-device-width: 320px) and (max-device-width: 480px) {
  .card {
    background: #bada55;
  }
}
```

import 스타일 시트로 범위 지정이 가능하지만 `@import`는 성능이 좋지 않아 사용하지 않는 것이 좋다.

```css
/* Avoid using @import if possible! */

/* Base styles for all screens */
@import url("style.css") screen;
/* Styles for screens in a portrait (narrow) orientation */
@import url("landscape.css") screen and (orientation: portrait);
/* Print styles */
@import url("print.css") print;
```

### Javascript

자바스크립트에서도 미디어 쿼리 사용이 가능하다. `window.matchMedia()` 메서드를 사용해서 조건을 정의하고, 브라우저 너비가 768px 이상일 때 콘솔에 메시지를 기록한다고 하자.

```js
// 최소 768px 너비의 뷰포트를 대상으로하는 미디어 조건 생성
const mediaQuery = window.matchMedia("( min-width: 768px )");

//  `matches` 속성에 주목하자
if (mediaQuery.matches) {
  console.log("Media Query Matched!");
}
```

안타깝게도 이 기능은 한 번만 실행되므로 알림이 해제된 경우 화면 너비를 변경하고 새로 고침하지 않고 다시 시도하면 다시 실행되지 않는다. 때문에 업데이트를 확인하는 리스너를 사용하는 것이 좋다.

```js
// 최소 768px 너비의 뷰포트를 대상으로하는 미디어 조건 생성
const mediaQuery = window.matchMedia("(min-width: 768px)");

function handleTabletChange(e) {
  // 미디어 쿼리 조건이 true 라면
  if (e.matches) {
    // 콘솔 로그 메세지 확인
    console.log("Media Query Matched!");
  }
}

// 리스너 이벤트 등록
mediaQuery.addListener(handleTabletChange);

// 초기 체크
handleTabletChange(mediaQuery);
```

미디어 쿼리 사용과 window.innerWidth 또는 window.innerHeight를 확인하여 변경 사항을 실행하는 크기 조정 이벤트 리스너를 바인딩하는 이전 자바스크립트 접근 방식을 비교하는 등에 대해 자세 알아보려면 ["자바스크립트 미디어 쿼리로 작업하기"](https://css-tricks.com/working-with-javascript-media-queries/)를 살펴보자.

## 미디어 쿼리 해부

![](https://i0.wp.com/css-tricks.com/wp-content/uploads/2020/09/media-query-anatomy.jpg?resize=1000%2C66&ssl=1)

### @media

@media는 CSS at-rule 중 하나이다.

```css
@media [media-type] ([media-feature]) {
  /* Styles! */
}
```

### Media types

```css
@media screen {
  /* Styles! */
}
```

- all: 모든 기기와 일치
- print: print 미리 보기에서 보는 문서나 콘텐츠를 인쇄용 페이지로 분할하는 모든 미디어와 일치
- screen: 화면이 있는 기기와 일치
- speech: 화면 리더기와 같이 콘텐츠를 소리로 읽는 디바이스와 일치. 이는 미디어 쿼리 레벨 4이후 더 이상 사용되지 않는 청각 유형을 대체한다.

### Media features

일치시키려는 미디어 유형을 정의하고 나면 어떤 기능에 일치시킬지 정의할 수 있다. 화면과 너비를 매칭하는 많은 예시 중, 화면은 유형이고 최소 너비와 최대 너비는 모두 특정 값을 가진 기능이다.

#### Viewport/Page 특성

| Feature         | Summary                                                                                                                                                                                                              | Values                 | Added                       |
| --------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ---------------------- | --------------------------- |
| width           | 뷰포트의 너비를 정의합니다. 특정 숫자(예: 400px) 또는 범위(최소 너비 및 최대 너비 사용)일 수 있습니다.                                                                                                               | `<length>`             |                             |
| height          | 뷰포트의 높이를 정의합니다. 특정 숫자(예: 400px) 또는 범위(최소 높이 및 최대 높이 사용)일 수 있습니다.                                                                                                               | `<length>`             |                             |
| aspect-ratio    | 뷰포트의 너비 대 높이 종횡비를 정의합니다.                                                                                                                                                                           | `<ratio>`              |
| orientation     | 기기 회전 방식에 따라 세로(가로) 또는 가로(세로) 등 화면 방향이 결정됩니다.                                                                                                                                          | portrait, landscape    |                             |
| overflow-block  | 뷰포트를 블록 방향으로 넘기는 콘텐츠를 장치에서 어떻게 처리할지 확인하며, 스크롤(스크롤 허용), 선택적 페이지(스크롤 및 수동 페이지 나누기 허용), 페이지(페이지로 나누기), 없음(표시되지 않음) 중 선택할 수 있습니다. | scroll, optional-paged | paged Media Queries Level 4 |
| overflow-inline | 인라인 축을 따라 뷰포트를 가득 채우는 콘텐츠를 스크롤할지 여부(없음(스크롤 안 함) 또는 스크롤(스크롤 허용))를 확인합니다.                                                                                            | scroll, none           | Media Queries Level 4       |

#### Display Quality

| Feature              | Summary                                                                                                                                             | Values                                      | Added                 |
| -------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------- | --------------------- |
| resolution           | 장치의 목표 픽셀 밀도를 정의합니다                                                                                                                  | `<resolution>`, infinite                    |                       |
| scan                 | 장치가 이미지를 화면에 그리는 과정을 정의합니다 (interlace는 홀수와 짝수 라인을 번갈아 그리고, progressive는 순차적으로 모두 그립니다).             | interlace, progressive                      |                       |
| grid                 | 장치가 그리드(1) 또는 비트맵(0) 화면을 사용하는지 결정합니다                                                                                        | 0 = Bitmap, 1 = Grid                        | Media Queries Level 5 |
| update               | 장치가 콘텐츠의 외관을 변경할 수 있는 빈도를 확인합니다 (없거나 none, 느리거나 slow, 빠른 fast 값이 포함됨).                                        | slow, fast, none                            | Media Queries Level 4 |
| environment-blending | 장치의 외부 환경(어두운 환경 또는 과도하게 밝은 장소)을 결정하는 방법입니다.                                                                        | opaque, additive, subtractive               |                       |
| display-mode         | 장치의 표시 모드를 테스트합니다 (전체 화면, 독립 실행형 응용 프로그램, 최소 UI로 탐색이 있는 독립 실행형 응용 프로그램, 또는 일반적인 브라우저 창). | fullscreen, standalone, minimal-ui, browser | Web App Manifest      |

#### Color

| Feature         | Summary                                                                                                                                                                        | Values            | Added                 |
| --------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ | ----------------- | --------------------- |
| color           | 장치의 색상 지원을 정의하며, 숫자로 표현됩니다. 예를 들어, 값이 12이면 12비트 색상을 지원하는 장치에 해당하며, 값이 0이면 색상 지원이 없음을 의미합니다.                       | `<integer>`       |                       |
| color-index     | 장치가 지원하는 값의 수를 정의합니다. 이는 특정 숫자(e.g. 10000) 또는 범위(e.g. min-color-index: 10000, max-color-index: 15000)로 지정할 수 있습니다. 이는 width와 유사합니다. | `<integer>`       |                       |
| monochrome      | 장치의 흑백 지원을 위해 픽셀당 비트 수를 정의하며, 0은 흑백 지원이 없음을 의미합니다.                                                                                          | `<integer>`       |                       |
| color-gamut     | 브라우저와 장치에서 지원하는 색상 범위를 정의하며, srgb, p3 또는 rec2020이 될 수 있습니다.                                                                                     | srgb, p3, rec2020 | Media Queries Level 4 |
| dynamic-range   | 브라우저와 사용자 장치의 비디오 영역에서 지원하는 밝기, 색상 깊이 및 대비 비율의 조합을 정의합니다.                                                                            | standard, high    |                       |
| inverted-colors | 브라우저나 운영체제가 색상을 반전하도록 설정되었는지 확인합니다 (이는 색상 관련 시각적 장애를 해결하기 위한 접근성 최적화에 유용할 수 있음).                                   | inverted, none    | Media Queries Level 5 |

#### Interaction

| Feature     | Summary                                                                                                                                                                     | Values             | Added                 |
| ----------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------------ | --------------------- |
| pointer     | `any-pointer`와 유사하지만, 주 입력 장치가 포인터인지, 그렇다면 얼마나 정확한지를 확인합니다 (coarse는 덜 정확하고, fine은 더 정확하며, none은 포인터가 없음을 의미합니다). | coarse, fine, none | Media Queries Level 4 |
| hover       | `any-hover`와 유사하지만, 주 입력 장치(예: 마우스나 터치)가 사용자가 요소 위로 호버링하는 것을 허용하는지 확인합니다.                                                       | hover, none        | Media Queries Level 4 |
| any-pointer | 장치가 포인터(예: 마우스나 스타일러스)를 사용하는지, 그리고 얼마나 정확한지를 확인합니다 (coarse는 덜 정확하고 fine은 더 정확합니다).                                       | coarse, fine, none | Media Queries Level 4 |
| any-hover   | 장치가 마우스나 스타일러스처럼 요소 위로 호버링할 수 있는지 확인합니다. 드문 경우로, 터치 장치도 호버링이 가능할 수 있습니다.                                               | hover, none        | Media Queries Level 4 |

그 외 Video Prefixed, Scripting, User Preference, Deprecated가 있다.

### Operators

미디어 쿼리는 논리 연산자를 지원하고 특정 조건에 따라 미디 유형을 맞춘다.

#### `and`

```css
/* 320px에서 768px 사이*/
@media screen (min-width: 320px) and (max-width: 768px) {
  .element {
    /* Styles! */
  }
}
```

#### `or` 또는 콤마 분리

```css
/* 사용자가 어두운 모드를 선호하거나 화면 너비가 1200px 이상인 화면과 일치합니다  */
@media screen (prefers-color-scheme: dark), (min-width 1200px) {
  .element {
    /* Styles! */
  }
}
```

#### `not`

지원하지 않거나 일치하지 않는 것을 기준으로 디바이스를 타겟팅하고 싶다면 예를 들어

- 디바이스가 print
- 한 가지 생상만 표시할 수 있는 경우
- 본문의 배경 색을 제거

```css
@media print and (not(color)) {
  body {
    background-color: none;
  }
}
```

## 출처

- [CSS Media Queries Guide](https://css-tricks.com/a-complete-guide-to-css-media-queries/)
