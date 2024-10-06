# CSS 미디어 쿼리 가이드(2) - UX 관점의 미디어 쿼리와 시스템 환경에 따른 설정

## CSS 미디어 쿼리를 정말로 필요로 할까?

미디어 쿼리는 강력한 도구임과 동시에 모든 상황에 적용하게 되면 유지 관리가 복잡하게 되고, CSS는 처음엔 작을지언정 점점 커지면 감당하기 어렵게 된다.

그렇기 때문에 "적응이나 특수 설계 없이도 가능한 모든 사람이 적용 가능하도록 제품을 디자인 하는 것"이라는 Ranald Mace의 유니버셜 디자인 개념을 따르는 것을 추천한다.

Laura Kalbag은 ["모두를 위한 접근성"](https://abookapart.com/products/accessibility-for-everyone)에서 접근성 디자인과 유니버셜 디자인의 차이는 미묘하지만 중요하다고 한다.

Miriam Suzanne의 말을 빌려보면 "운영 체제, 인터페이스 및 언어에 걸쳐 무한하고 알 수 없는 캔버스에서 알 수 없는 콘텐츠의 그래픽 디자인을 시도한느 것이 CSS다. 어느 누구도 우리가 무엇을 하는지 알 수 있는 방법은 없다"

때문에 제품을 디자인, 개발 및 생각할 때 가정을 버리고 미디어 쿼리를 사용하여 콘텐츠가 모든 연락처와 사용자에 올바르게 표시되는지 확인해야 한다.

## 일치하는 값의 범위

미디어 기능에 최소 또는 최대 제약 조건 표시를 위해 min- 또는 max-를 접두사로 붙일 수 있다.

다음 코드에선 뷰포트 너비가 30m보다 넓고 80m보다 좁은 경우 본문 배경을 보라색으로 칠한다. 뷰포트 너비가 해당 값 범위와 일치하지 않으면 흰색으로 폴백된다.

```css
body {
  background-color: #fff;
}

@media (min-width: 30em) and (max-width: 80em) {
  body {
    background-color: purple;
  }
}
```

미디어 쿼리 레벨 4는 <, >, = 연상자를 사용하여 더 간단한 구문을 지정하여 위 코드를 아래와 같이 작성할 수 있다.

```css
@media (30em <= width <= 80em) {
  /* ... */
}
```

## 중첩 및 복잡한 의사 결정

CSS를 사용하면 괄호를 사용하여 at- 규칙이나 그룹 문을 중첩할 수 있으므로 복잡한 연산시 원하는 만큼 깊이 들어갈 수 있다.

```css
@media (min-width: 20em), not all and (min-height: 40em) {
  @media not all and (pointer: none) {
    ...;
  }
  @media screen and ((min-width: 50em) and (orientation: landscape)),
    print and (not (color)) {
    ...;
  }
}
```

> 참고: 이는 복잡해지기 때문에 유지 보수가 어려워지니 잘 생각하고 적용하자.

## 접근성

미디어 쿼리 레벨 4에 추가된 많은 기능은 접근성에 중점을 두고 있다.

### prefers-reduced-motion

prefers-reduced-motion는 사용자 움직임과 애니메이션의 양을 최소화하기 위해 모션 감소 기본 설정을 활성하 했는지 감지한다.

- no-preference: 사용자가 시스템에 기본 설정은 알리지 않았음을 나타낸다.
- reduce: 사용자가 불필요한 움직임이 모두 제거될 정도로 움직임이나 애니메이션의 양을 최소화하는 인터페이스를 선호한다고 시스템에 표시한다.

![](https://i0.wp.com/css-tricks.com/wp-content/uploads/2020/09/macos-preference-motion.png?resize=1024%2C772&ssl=1)

Eric Bailey의 글에선 이 코드로 모든 애니메이션을 중지하는 것을 제안했다.

```css
@media screen and (prefers-reduced-motion: reduce) {
  * {
    /* 매우 짧은 기간에 이벤트에 의존하는 자바스크립트가 여전히 작동한다. */
    animation-duration: 0.001ms !important;
    animation-iteration-count: 1 !important;
    transition-duration: 0.001ms !important;
  }
}
```

부트스트랩과 같은 인기 있는 프레임워크에도 이 기능이 기본적으로 설정되어있다. prefers-reduced-motion을 상요하지 않을 이유가 없다.

### prefers-contrast

prefers-contrast은 시스템 기본 설정 또는 브라우저 설정 대비를 높이거나 낮추도록 선택했는지 여부를 알려준다.

- no-preference: 사용자가 시스템에 기본 설정을 알리지 않은 경우. boolean으로 사용하면 거짓으로 평가된다.
- high: 사용자가 더 높은 수준의 대비를 표시하는 옵션 선택시
- low: 사용자가 더 낮은 수준의 대비를 표시하는 옵션 선택시

![](https://i0.wp.com/css-tricks.com/wp-content/uploads/2020/09/macos-preference-contrast.png?resize=1024%2C772&ssl=1)

이 기능은 어떤 브라우저에도 작동되지 않는다. ms에서는 이전 버전에서 -ms-high-contrast 기능을 비표준으로 구현했지만 Edge v18이하에서만 동작하고 크로미움에는 동작하지 않는다.

```css
.button {
  background-color: #0958d8;
  color: #fff;
}

@media (prefers-contrast: high) {
  .button {
    background-color: #0a0db7;
  }
}
```

### inverted-colors

inverted-colors은 시스템 환경설정 또는 브라우저 설정에서 색상 반전을 했는지 여부를 알려준다.

- none: 색상이 정상적으로 표시되는 경우
- inverted: 사용자가 색상을 반전하는 옵션을 선택한 경우

![](https://i0.wp.com/css-tricks.com/wp-content/uploads/2020/09/macos-preference-invert-colors.png?resize=1024%2C772&ssl=1)

반전된 색상의 문제점은 img, video의 색상이 반전되어 x-ray 이미지 처럼 보이게 된다. CSS 반전 필터를 사용하면 모든 이미지와 동영상을 선택하고 다시 반전시킬 수 있다.

```css
@media (inverted-colors) {
  img,
  video {
    filter: invert(100%);
  }
}
```

이 기능은 safari에서만 지원된다.

### prefers-color-scheme

'다크 모드'색 구성표를 사용하는 것은 더 많이 볼 수 있고, prefers-color-scheme 덕분에 시스템 또는 브라우저 기본 설정을 활용하여 IR 기본 설정에 따라 '다크'나 '라이트'

```css
body {
  --bg-color: white;
  --text-color: black;

  background-color: var(--bg-color);
  color: var(--text-color);
}

@media screen and (prefers-color-scheme: dark) {
  body {
    --bg-color: black;
    --text-color: white;
  }
}
```

Adhuham이 [다크 모드에 대한 전체 가이드](https://css-tricks.com/a-complete-guide-to-dark-mode-on-the-web/)에서 설명했듯이 다크 모드에는 단순히 배경색을 변경하는 것 이상의 의미가 있다. 이는 현명한 구현 전략이 없으면 정말 어려운 코드 베이스를 만들게 된다.

## 출처

- [CSS Media Queries Guide](https://css-tricks.com/a-complete-guide-to-css-media-queries/)
