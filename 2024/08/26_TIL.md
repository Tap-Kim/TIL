# 리소스 힌트로 브라우저 지원 및 UX 개선하기

## 정리

- 여러 리소스를 로드를 하기위해 다양한 최적화 방법이 있고 그중에서 리소스 힌트를 적절하게 사용한다면 UX를 크게 향상 시키는게 가능하다. 그 중 `preconnect, dns-prefetch, preload, prefetch, fetchpriority` 순으로 추가적인 기능이 도입되었다.
- 리소스 힌트는 브라주어가 특정 작업을 미리 수행하도록 지시하는 속성이다. 이로 인해 리소스 로드의 성능을 개선이 가능하다.
- 리소스 힌트는 미리 서버에 연결하고, DNS 조회를 수행이 가능하며, 브라우저가 해당 리소스를 발견하기 전에 미리 가져오는 것이 가능하다.
- 리소스 힌트는 HTML로 지정이 가능하며 `<head>`로 초기에 지정이 가능하다.
- `요약하자면`: 각자 상황에 맞게 최적화가 가능하다. 예를들어 리소스 다운로드의 대역폭을 최소화하여 사전 로드를 진행하거나, 브라우저 리소스를 받기전에 사전 연결을 진행하여 속도를 증가시키거나, 우선순위를 직접 정해 사용자의 이동에 따라 리소스 우선순위를 조절하여 다운 받거나 페이지 리소스를 사전 패칭하여 페이지 이동을 자연스럽게 행하는 등 리소스 힌트 만으로 최적화를 극대화 시킬 수 있다.

### 참고

- 주요 키워드: resource hint, html, browser, preconnect, dns-prefetch, preload, prefetch, fetchpriority
- 관련 기술: DNS, CORS, browser rendering, UX

## 무엇을 알았는지

### preconnect

`preconnect` 힌트는 중요한 리소스를 가져오는 다른 오리진에 대한 연결을 설정하는데 사용된다. 예를들어 이미지나 에셋은 CDN 또는 다른 교차 출처에서 호스팅하고 있을 수 있다.

이는 브라우저가 가까운 시일내에 특정 교차 출처 서버에 연결할 계획이 있고, 브라우저는 HTML 파서나 preload 스캐너가 연결 완료될 때까지 `기다리기 전`에 가능한 빨리 연결을 열어야하고, 페이지에 많은 양의 교차 출처 리소스가 있다면 현재 페이지에 가장 중요한 리소스에 대해 `preconnect`를 사용한다.

![](https://web.dev/static/learn/performance/resource-hints/image/fig-1_856.png)

만약 이미지가 여러개 있고, 아래 처럼 특정 이미지 하나를 `preconnect`하여 브라우저에서 검색되기 전에 서버에서 사전 연결을 통하여 리소스를 받을수 있게된다.

```html
<head>
  <link rel="preconnect" href="https://cdn.glitch.global" />
</head>
```

![](https://cdn.glitch.global/db01a8e4-9230-4c5c-977d-85d0e0c3e74c/preconnect.png?v=1669249002348)

preconnect를 제거한다면?

![](https://cdn.glitch.global/db01a8e4-9230-4c5c-977d-85d0e0c3e74c/preconnect-before.png?v=1669249000700)

또한 `preconnect`는 일반적으로 font를 사용할때 쓰이는데, `https://fonts.googleapis.com` 도메인에 `preconnect`하는 `@font-face` 선언과 `https://fonts.gstatic.com` 도메인에 글꼴 파일을 제공한다.

```html
<link rel="preconnect" href="https://fonts.googleapis.com" />
<link rel="preconnect" href="https://fonts.gstatic.com" crossorigin />
```

`crossorigin` 속성으로 `preconnect` 힌트 사용시 원본에서 다운로드하는 리소스가 글꼴 파일과 같은 CORS를 사용하는 경우 `preconnect` 힌트에 `crossorigin` 속성을 추가해야한다. 왜냐하면 `crossorigin` 속성을 생략시 브라우저는 글꼴 파일을 다운로드할 때 "new connection"을 하게되고 `preconnect` 힌트로 열린 연결을 `재사용`하지 않게 된다.

### dns-prefetch

교차 출처 서버의 연결을 조기에 열면 개선이 핑료할때 초기 로드 시간이 한 번에 많이 설정하게 되어 과도하게 사용된다면 `dns-prefetch` 힌트를 사용한다.

이는 이름에서도 알 수 있듯이 교차 출처에 대한 `연결을 하지 않고 대신 미리 DNS 조회를 수행`한다. DNS 조회는 도메인 이름이 기본 IP 주소로 확인될 때 발생하고, 장치나 네트워크 수준의 DNS 캐시 계층은 이를 좀더 쉽게 만드는데 일반적으로 빠른 프로세스이나 여전히 시간은 걸린다.

```html
<link rel="dns-prefetch" href="https://fonts.googleapis.com" />
<link rel="dns-prefetch" href="https://fonts.gstatic.com" />
```

`dns-prefetch`는 상당히 저렴하고 `preconnect`에 비해 상대적으로 적은 비용이라 경우에 따라 더 적절하다. 특히 사용자를 따라가는 상황일때 다른 웹사이트로 이동하는 링크시 해당 리소스 힌트를 사용하는 것이 바람직하다.

### preload

`preload` 지시문은 리소스의 조기 요청을 시작하는데 사용된다.

```html
<link rel="preload" href="/lcp-image.jpg" as="image" />
```

`preload`는 늦게 발결된 중요 리소스로 제한해야한다. 일반 적인 사례로는 글꼴 파일, `@import`를 통해 가져온 CSS 파일 또는 LCP 후보가 될 가능성이 높은 CSS `background-image` 리소스이다. 이런 경우 파일을 외부 리소스에서 참조되므로 `preload` 스캐너에서 발견되지 않는다.(참고: [브라우저 리소스 로딩 최적화](https://github.com/Tap-Kim/TIL/blob/main/2024/08/23_TIL.md))

위와 마찬가지로 이미지가 여러개 있고, 아래 처럼 특정 이미지 하나를 `preload`하여 브라우저에서 렌더링 차단이 일어나는 최우선순위에 있는 `js, css` 파일과 동일한 우선순위로 특정 이미지를 사전 로드해올수 있다.

```html
<head>
  <link
    rel="preload"
    href="https://cdn.glitch.global/db01a8e4-9230-4c5c-977d-85d0e0c3e74c/image-1.jpg?v=1669198400523"
    as="image"
  />
</head>
```

`preconnect`과 마찬가지로, 글꼴과 같은 CORS 리소스를 미리 로드하는 경우 `preload` 지시문에는 `crossorigin` 속성이 필요하다. 이를 설정하지 않으면 브라우저에서 리소스를 두 번 다운로드해서 대역폭을 낭비할 수 있게 된다.

![](https://cdn.glitch.global/db01a8e4-9230-4c5c-977d-85d0e0c3e74c/preload-background-image-after.png?v=1669249008939)

preload를 제거했다면?

![](https://cdn.glitch.global/db01a8e4-9230-4c5c-977d-85d0e0c3e74c/preload-background-image-before.png?v=1669249010586)

```html
<link rel="preload" href="/font.woff2" as="font" crossorigin />
```

> 참고1: 지시어의 `<link>` 요소에 `as` 속성이 누락된 경우 `preload` 지시어에 지정된 리소스가 두 번 다운로드 된다.

앞의 HTML 스니펫에서 브라우저는 `/font.woff2`가 동일한 도메인에 있더라도 CORS 요청을 사용하여 `/font.woff2`를 미리 로드하도록 지시받는다.

### prefetch

`prefetch`는 나중에 탐색에서 사용될 가능성이 높은 리소스에 대한 낮은 우선순위 요청을 시작하는데 유용하다.

```html
<link rel="prefetch" href="/next-page.css" as="style" />
```

대체로 `preload`와 동일한 형식을 따르나 `<link>` 요소의 rel 속성만 `prefetch` 값으로 대신사용한다. 그리고 `preload`와 차이점으로는 나중에 발생할 수도 있고 않을수 있는 탐색을 위해 리소스를 가져오기 시작한다는 점에서 `추측` 기반으로 동작한다. 예를 들면 사용자가 웹사이트를 완료하기 위해 사용자 흐름을 파악하고나서 페이지 렌더링에 중요한 리소스를 미리 가져오면 리소스를 줄이는데 도움이 된다.

단점으로는 사용자가 `prefetch`한 페이지로 이동하지 않을 가능성이 있다는 단점이 있다. 이럴땐 사용자 패턴의 서비스를 활용하여 `prefetch`적용 여부를 판단하는 것이 좋다.

### fetchpriority API

`fetchpriority` API는 `<link>, <img>, <script>` 요소와 함께 사용이 가능하고, 리소스의 우선순위를 매길수 있다.

```html
<div class="gallery">
  <div class="poster">
    <img src="img/poster-1.jpg" fetchpriority="high" />
  </div>
  <div class="thumbnails">
    <img src="img/thumbnail-2.jpg" fetchpriority="low" />
    <img src="img/thumbnail-3.jpg" fetchpriority="low" />
    <img src="img/thumbnail-4.jpg" fetchpriority="low" />
  </div>
</div>
```

![](https://cdn.glitch.global/db01a8e4-9230-4c5c-977d-85d0e0c3e74c/fetchpriority.png?v=1669248998980)

이 속성은 LCP 이미지를 사용할때 아주 효과적이다. 이미지 우선순위를 높여 적은 노력으로 LCP 개선이 가능하다.

기본적으로는 낮은 우선순위로 가져온다. 레이아웃 후 이미지가 초기 뷰포트 내에 있는 것으로 확인ㅇ되면 우선순위가 높아진다. 앞의 HTML을 보면 `fetchpriority="high"`는 LCP 이미지를 다운하도록 지시하고, 나머지는 낮은 우선순위로 썸네일 이미지를 다운로드한다.

최신 브라우저는 리소스를 두 단계로 나뉜다.

- 첫 번쨰: 중요한 리소스를 위해 `예약`되어 있고, *모든 차단 스크립트가 다운로드되고 실행되면 종료*한다.
- 두 번쨰: 첫 번째에서 우선순위가 낮은 리소스의 다운이 지연될 수 있고 `fetchpriority="high"` 사용시 리소스의 우선순위를 높여 첫 번째 단계에서 리소스를 할수 있도록 한다.

### 출처

- [Assist the browser with resource hints](https://web.dev/learn/performance/resource-hints)
