# ResizeObserver API(feat. window resize)

## GPT의 간략한 설명

`ResizeObserver`는 DOM 요소의 크기의 변경을 관찰할 수 있는 API이다. 이는 요소의 크기가 변경할 때마다 감지하여 콜백 함수를 실행한다. 주로 창의 크기 변화에 반응해야하는 `window.addEventListener('resize')`와는 차이가 있다.

### 사용법

`ResizeObserver` 객체 생성 후 관찰 요소에 연결한다.

```js
const resizeObserver = new ResizeObserver((entries) => {
  for (let entry of entries) {
    console.log("Content Rect:", entry.contentRect);
    console.log("Width:", entry.contentRect.width);
    console.log("Height:", entry.contentRect.height);
  }
});

const element = document.querySelector(".resizable-element");
resizeObserver.observe(element);
```

`entries`는 관찰 대상 요소들의 배열이다. 각 `entry` 객체는 `contentRect`를 통해 요소와 새로운 크기 정보를 제공한다.

### ResizeObserver와 window.addEventListener('resize')의 차이점

1. ResizeObserver

- **지정한 요소의 크기 변화만 감지**: 특정 DOM 요소의 크기(너비 또는 높이) 변화에 대해서만 반응한다. 그로인해 요소 자체의 크기가 변할 때만 트리거 된다. 이는 화면 크기나 레이아웃에 변화가 있을 때만 실행한다는 의미
- **더 정확한 반응**: 요소 크기 변경이 실제로 발생한 후에 정확하게 콜백을 실행한다. `window.resize` 이벤트와는 다르게 브라우저 창 크기 변경뿐만 아니라 요소 크기 변화를 직접적으로 관찰이 가능하다.
- **비동기 호출**: 비동기적으로 콜백 호출을 통해 성능 부하를 줄인다. 따라서 동기화 처리나 과도한 이벤트 호출을 방지할 수 있다.

2. window.addEventListener('resize')

- **적용 대상**: `resize` 이벤트는 브라우저 창 크기가 변할 때 트리거 되고 요소 크기에 대해 직접적인 관찰은 불가능하다. 이는 브라우저 전체 크기 변화 감지에 사용된다.
- **성능 효율성**: 모든 크기 조정이 끝날 때까지 연속적으로 트리거될 수 있어 성능에 영향을 줄 수 있다.

### 언제 ResizeObserver를 사용할까?

- 특정 요소의 크기 변경을 감지해야 할 때
- UI 컴포넌트가 내부 콘텐츠에 따라 크기가 동적으로 변해야 할 때
- 성능 부하를 최소화하면서 필요한 요소 크기 변경에 반응해야 할 때

이러한 경우 `ResizeObserver`가 더 적합하며, 반대로 전체 화면의 크기 변화를 트래킹하려면 `window.addEventListener('resize')`를 사용하는 것이 효율적이다.

## ResizeObserver: 요소에 대한 document.onresize와 유사점

`ResizeObserver` 이전에는 document 크기 조정 이벤트에 리스너를 연결하여 뷰포트의 크기 변경에 대해 알림을 받아야 했고, 해당 변경의 영향을 받는 요소를 파악하고 특정 루틴을 호출하여 적절하게 반응해야 했다. 크기 조정 후 요소의 새 치수가 필요한 경우, 모든 읽기/쓰기를 일괄 처리하지 않으면 레이아웃 thrashing이 발생할 수 있는 `getBoundingClientRect()` 또는 `getComputedStyle()`을 호출해야한다.

기본 창 크기가 조정되지 않은 상태에서 요소의 크기가 변경되는 경우는 포함되지 않는다. 예를 들어, 새 자식을 추가하거나 요소의 display 스타일을 none으로 설정하거나 이와 유사한 작업을 수행하면 요소, 형제 또는 상위 요소의 크기가 변할 수 있다.

때문에 `ResizeObserver`가 유용한 이유 인데, 변경의 원인과 관계 없이 관찰된 요소의 크기 변경에 반응하여 관찰된 요소의 새로운 크기에 대한 액세스도 제공한다.

## 언제 보고될까?(1)

일반적으로 [ResizeObserverEntry](https://developer.mozilla.org/en-US/docs/Web/API/ResizeObserverEntry)는 `contentRect` 속성을 통해 요소의 `Content Box`를 보고한다. 이 속성은 [DOMRectReadOnly](https://developer.mozilla.org/en-US/docs/Web/API/DOMRectReadOnly) 객체를 반환한다. `Content Box`는 콘텐츠를 배치할 수 있는 상자이고, `Border Box`에서 패딩을 뺀값이다.

![](https://web.dev/static/articles/resize-observer/image/a-diagram-the-css-box-mo-f526084ad034e_960.png)

`ResizeObserver`는 `contentRect`의 크기와 패딩을 모두 보고하지만, `contentRect`만 감시한다는 점을 유의하자. `contentRect`를 요소의 bounding box와 혼동하지말자. `getBoundingClientRect()`가 보고하는 bounding box는 전체 요소와 그 하위 요소를 포함하는 박스이다.

> 예외: SVG는 예외로 `ResizeObserver`가 bounding box의 치수를 보고한다.

## 언제 보고될까?(2)

스펙은 `ResizeObserver`가 페인트 전과 레이아웃 후의 모든 크기 조정 이벤트를 처리하도록 규정한다. 따라서 페이지 레이아웃을 변경할 때 `ResizeObserver`의 콜백이 **이상적인 장소**가 된다. `ResizeObserver` 처리는 레이아웃과 페인트 사이에서 이루어지므로 이렇게 하면 페인트가아닌 레이아웃에서만 무효화한다.

## 확인

콜백 내에서 관찰된 요소 크기를 `ResizeObserver`로 변경하면 어떻게 될까? 정답을 **콜백에 대한 또 다른 호출이 바로 트리거 된다는 점**. `ResizeObserver`는 무한 콜백 루프와 순환 종속성을 피할 수 있는 메커니즘이 있다. 크기가 조정된 요소가 이전 콜백에서 처리된 가장 얕은 요소보다 DOM 트리에서 더 깊은 곳에 있는 경우에만 변경사항이 동일한 프레임에서 처리된다. 그렇지 않으면 다음 프레임으로 연기된다.

## 애플리케이션

### 특정 요소 상자만 관찰

`ResizeObserver`로 할 수 있는 한 가지는 요소별 미디어 쿼리를 구현하는 것. 요소를 관찰하여 디자인 중단점을 필수적으로 정의하고 요소의 스타일을 변경할 수 있다. 다음
예제에서 두 번째 상자의 너비에 따라 테두리 반경이 변경된다.

[테스트 주소](https://googlechrome.github.io/samples/resizeobserver/)

```js
const ro = new ResizeObserver((entries) => {
  for (let entry of entries) {
    entry.target.style.borderRadius =
      Math.max(0, 250 - entry.contentRect.width) + "px";
  }
});
// 두번째 상자만 관찰
ro.observe(document.querySelector(".box:nth-child(2)"));
```

### 채팅창

채팅창은 일반적으로 위에서 아래로 내려오는 대화 레이아웃에서 스크롤 위치가 문제가 된다. UX를 고려해 최신 메시지가 표시되는 하단에 고정이 되는 것이 좋다. 또 여러 종류의 레이아웃 변경도 마찬가지다.

`ResizeObserver`를 사용하면 두 가지 시나리오를 모두 처리하는 단일 코드를 작성할 수 있다. 창 크기 조정을 `ResizeObserver`가 정의상 캡처할 수 있는 이벤트이지만, 새로운 요소를 위한 공간을 만들어야 하므로 `appendChild()`를 호출해 해당 요소의 크기도 조정하면 된다.(`overflow: hidden`설정이 되지 않은 경우)

[![영상 재생](data:image/svg+xml;base64,PHN2ZyBmaWxsPSJXaW5kb3ciIGhlaWdodD0iMjQiIHZpZXdCb3g9IjAgMCAyNCAyNCIgd2lkdGg9IjI0IiB4bWxucz0iaHR0cDovL3d3dy53My5vcmcvMjAwMC9zdmciPgogICAgPHBhdGggZD0iTTggNXYxNGwxMS03eiIvPgogICAgPHBhdGggZD0iTTAgMGgyNHYyNEgweiIgZmlsbD0ibm9uZSIvPgo8L3N2Zz4K)](https://web.dev/static/articles/resize-observer/video/webfundamentals-assets/resizeobserver/chat_vp8.webm) 채팅 테스트 영상 재생

```js
const ro = new ResizeObserver((entries) => {
  document.scrollingElement.scrollTop = document.scrollingElement.scrollHeight;
});

// 창 크기 조정시 scrollingElement 관찰
ro.observe(document.scrollingElement);

// 타임라임 관찰하여 새 메시지 처리
ro.observe(timeline);
```

사용자가 수동으로 위로 스크롤 후 새 메시지가 들어올 때 스크롤이 해당 메시지에 고정되길 원하는 경우를 코드 추가가 가능하다. 또 다른 사용 사례는 자체 레이아웃을 수행하는 모든 종류의 사용자 정의 요소에 대한 것. `ResizeObserver`가 나오기 전까지는 크기 변경시 알림을 받아 자식들으 다시 레이아웃할 수있는 신뢰할 수 있는 방법이 없었다.

## INP에 미치는 영향

INP는 사용자 상호작용에 대한 페이지의 전반적인 응답석을 측정하는 지표다. 페이지의 INP가 '양호' 임계값, 즉 200밀리초 이하인 경우 페이지가 사용자의 상호 작용에 의해안정적으로 반응하는 것을 볼 수 있다.

상호작용 응답으로 이벤트 콜백이 실행되는 데 걸리는 시간을 상호작용의 총 지연 시간에 크게 영향을 미칠 수 있지만 INP에서 고려할만한 측면은 아니다.

`ResizeObserver`와 관련해서 `ResizeObserver` 인스턴스가 실행하는 콜백이 렌더링 작업 직전에 발생하기 때문에 중요하다. 이는 콜백에서 발생하는 작업의 결과로 사용자 인터페이스의 변경이 필요할 가능성이 높기 때문에 고려해야한다.

과도한 렌더링 작업은 브라우저에 중요한 작업을 지연시키는 요소이므로 `ResizeObserver` 콜백에서 필요한 만큼만 렌더링 작업을 수행하도록 주의하자.

- [과도한 스타일 재계산 작업을 피하려면](https://web.dev/articles/reduce-the-scope-and-complexity-of-style-calculations) CSS 선택기를 최대한 단순하게 만들 것. 스타일 재계산은 레이아웃 직전에 발생하며 복잡한 CSS 선택기는 레이아웃 작업을 지연시킬 수 있다.
- 강제 리플로우를 트리거할 수 있는 `ResizeObserver` 콜백에 어떤 작업도 하지 말것
- 페이지의 레이아웃을 업데이트하는 데 필요한 시간은 일반적으로 페이지의 DOM 요소 수에 따라 증가한다. 이는 페이지가 `ResizeObserver`를 사용하는지 여부와 관계없이 페이지의 구조가 복잡해지면 `ResizeObserver` 콜백에서 수행되는 작업은 커질 수 있다.

## 결론

`ResizeObserver`는 모든 주요 브라우저에서 사용가능하고 요소 수준에서 요소 크기 변경을 효율적으로 모니터링 할 수 있는 방법을 제공하나. 이로인해 렌더링이 너무 지연되지 않도록 주의할 것.

## 출처

- [GPT - ResizeObserver vs window.addEventListner('resize']()
- [ResizeObserver: it's like document.onresize for elements](https://web.dev/articles/resize-observer)
