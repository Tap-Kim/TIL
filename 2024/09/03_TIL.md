# 지연 로드 이미지 및 `<iframe>` 요소

## 정리

- loading 속성을 사용해서 이미지를 지연 로드해서 초기 뷰포트 또는 일정 거리 내 로딩 지연을 통해 네트워크 대역폭을 조절하여 INP에 대응할 수 있다.
- 초기 뷰포트에 존재하는 이미지를 지연 로드하여 뷰포트에 의해 다양한 환경(위치, 크기, 장치)로 INP에 영향이 가지 않도록 한다.
- `<iframe>` 지연 로드하여 리소스가 큰 임베디트 리소스의 대역폭을 줄여 중요 리소스의 개선하자
- Facade를 사용하여 필요에 의해 또는 사용자 상호작용시 사용해야할 리소스를 다운받고 교체하자,
- Javascript 지연 로딩 라이브러리를 활용하자.

### 참고

- 주요 키워드: LCP, INP, viewport, lazy, eager,
- 관련 기술: Facade, lazy loading

## 무엇을 알았는지

이미지 로딩이 느리면 초기 뷰포트 외부에 있는 이미지 로딩을 연기하면 초기 뷰포트 내에서 더 중요한 리소스에대한 네트워크 대역폭에 대한 race-condition을 줄이는데 도움이 된다. 이는 LCP 개선과, 재할당된 대역폭은 LCP 후보가 더 빨리 로드되고 페인트되는데 도움이 된다.

`<iframe>` 요소에 관한 페이지의 INP는 시작시 지연 로딩을 통해 개선이 된다. 그 자체가 하위 리소스를 가진 완전히 별도의 HTML 문서이기 때문이다. `<iframe>`은 별도의 프로세스에서 실행될수 있지만, 다른 스레드와 프로세스를 공유하는 경우가 드물지 않아서 페이지가 사용자 입력에 적게 반응하는 상황이 발생할 수 있다.

따라서 오프스크린 이미지와 `<iframe>`의 로딩을 지연하는 것은 좋은 방식이고, 성능에서도 나은 이점을 가진다.

### loading 속성을 사용해서 이미지를 지연 로드하기

`<img>`에 [loading](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/iframe#lazy) 속성을 추가한다.

- `eager`: 브라우저 이미지가 초기 뷰포트 밖에 있더라도 즉시 로드되는 것을 알리며, `loading` 속성의 기본 값이다.
- `lazy`: 이미지가 보이는 뷰포트에서 일정 거리 내에 있을 때까지 이미지 로딩을 지연한다. 이는 브라우저마다 다르지만 종종 사용자가 스크롤할 때까지 이미지가 로드될 만큼 충분히 크게 설정된다.

또한 `<picture>` 사용시 자식 요소에 적용해야한다. 이는 `<picture>` 요소가 서로 다른 이미지 후보를 가리키는 추가 `<source>` 요소를 포함하는 컨테이너이고 브라우저가 선택한 후보가 하위 `<img>` 요소에 직접 적용되기 때문이다.

### 초기 뷰포트에 있는 이미지는 지연 로드하지 말것

페이지 렌더링 전 뷰포트 내에 요소의 정확한 위치를 아는 것은 복잡할 수 있고, 다양한 뷰포트 크기, 종횡비 및 장치를 고려해야하기 때문에 초기 뷰포트의 이미지는 지연 로드하지 않는 것이 좋다.

지연 로드된 이미지는 브라우저가 [레이아웃](https://web.dev/articles/howbrowserswork#layout)을 완료할 떄까지 기다려야 이미지의 최종 위치가 뷰포트 내에 있는지 확인이 가능하다. 즉, 초기 뷰포트의 `<img>` 요소에 `loading='lazy'` 속성이 있는 경우, [원시 마크업에서 preload 스캐너에 의해 발견되는 즉시](https://web.dev/articles/preload-scanner#whats_a_preload_scanner) 페이지로 가져오는 것이 아니라 모든 CSS가 다운로드, 파싱 및 적용된 후에만 요청된다.

`<img>` 요소의 `loading` 속성은 모든 주요 브라우저에 지원이되므로 javascript를 사용해서 이미지를 따로 지연 로드할 필요는 없다. 브라우저가 이미 제공하는 기능을 제공하기 위해 페이지에 javascript를 추가하면 INP와 같은 페이지 성능의 다른 측면에 영향을 미치기 때문이다.

> 참고: 속성 loading은 이미지의 네트워크 우선순위에 영향을 미치지 않는다. 초기 뷰포트에 있는 이미지는 모든 CSS가 다운되고, 파싱될 때까지 계속 대기한다.

### `<iframe>` 지연 로드 요소

뷰포트에 `<iframe>`이 표시될 때까지 지연 로딩하면 상당한 데이터 절약과 최상위 페이지 로드시 중요한 리소스 로딩시 개선이 가능하다.

예를들어, 유튜브 비디오 임베디드와 같이 큰 리소스를 지연 로드하게되면 초기 페이지 로드시 500KiB 이상 절약되는 반면, 페이스북의 좋아요 버튼 플러그인을 지연 로딩하면 200KiB 이상 절약이 된다.

### `<iframe>` 요소의 loading 속성

`<iframe>`의 `loading` 속성은 모든 브라우저에서 지원된다.

- `eager`: 기본값. 브라우저 `<iframe>` 요소의 HTML과 하위 리소스를 즉시 로드한다,
- `lazy`: `<iframe>` 뷰포트로부터 미리 정의된 거리내에 들어올 때까지 요소의 HTML과 하위 리소스의 로드를 지연한다.

> 참고: 크롬은 공간은 예약하고 `<iframe>` 지연 로드가 여전히 페칭되고 있을 때, 레이아웃 이동을 피하기 위해 플레이스 홀더를 표시한다. 그러나 레이아웃 이동을 최소화하기 위해 CSS에서 `<iframe>` 요소의 width, height 속성과 추가 스타일을 사용하는 것을 고려해야한다.

### Facade

페이지 로드 중 임베드 즉시 로드 대신 사용자 상호작용에 대한 응답으로 필요에 따라 로드가 가능하다. 이는 사용자가 상호작용할 때까지 이미지나 다른 적절한 HTML 요소를 표시하여 수행 가능하다. 사용자가 요소와 상호작용하면 타사 임베드로 바꿀 수 있다. 이 기술을 [Facade](https://web.dev/articles/embed-best-practices#use_click-to-load_to_enhance_facades)라 한다.

이는 비디오 임베드와 시각적으로 유사한 정적 이미지를 보여주고 그 과정에서 상당한 대역폭을 절약할 수 있는 기회다. 사용자가 이미지를 클릭하면 실제 `<iframe>` 임베드로 대체되고, 그러면 타사 `<iframe>`의 HTML과 하위 리소스가 다운된다.

초기 페이지 로드를 개선하는 것 외에도 사용자가 비디오를 재생하지 않으면 비디오를 제공하는데 필요한 리소스가 다운되지 않는 다는 점에서 장점을 가진다.

채팅 위젯을 facade 기술의 좋은 사례이다. 대부분의 채팅 위젯은 페이지 로드와 사용자 입력에 대한 반응성에 부정적인 영향을 미칠 수 있는 상당한 양의 javascript를 다운로드한다. 무엇이든 미리 로드하는 것과 마찬가지로 비용은 로드 시간에 발생하지만 채팅 위젯은 모든 사용자각 꼭 상호 작용할 의도가 없는 것은 아니다. 따라서 "채팅 시작"과 같은 가짜 버튼으로 대체가능하고, 적정 시간에 포인터를 가져다 놓거나 클릭시 실제 기능적 채팅 위젯이 그 자리에 삽입되게 하는 것이다.

### Javascript 지연 로딩 라이브러리

`<video>`의 `poster`로 로드한 이미지, `background-image` CSS 속성으로 로드한 이미지 등 지연 로드해야 하는 경우 [lazysizes](https://github.com/aFarkas/lazysizes) 또는 [yall.js](https://github.com/malchata/yall.js)를 사용하면 된다.

특히 오디오 트랙이 없는 자동 재생 반복시 `<video>`를 사용하는 것보다 [애니메이션 GIF를 사용하는 것보다 훨씬 효율적인 대안](https://web.dev/articles/replace-gifs-with-videos)이다.

이러한 라이브러리는 [Intersection Observer API](https://developer.mozilla.org/docs/Web/API/Intersection_Observer_API)를 사용해서 작동하고, 페이지의 HTML이 초기 로드 후 변경되는 경우 [Mutation Observer API](https://developer.mozilla.org/docs/Web/API/MutationObserver)를 사용하여 사용자의 뷰포트에 들어오는 시점을 인식하는 라이브러리르 상용하는 것이 좋다.

```html
<!--자동 재생, 반복, 음소거 및 재생 인라인 속성은 사용자 개입 없이 동영상이 자동 재생되도록 하기 위한 것입니다. -->
<video class="lazy" autoplay loop muted playsinline width="320" height="480">
  <source data-src="video.webm" type="video/webm" />
  <source data-src="video.mp4" type="video/mp4" />
</video>
```

### 출처

- [Lazy load images and `<iframe>` elements](https://web.dev/learn/performance/lazy-load-images-and-iframe-elements)
