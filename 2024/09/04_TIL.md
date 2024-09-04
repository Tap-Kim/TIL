# prefetching, prerendering, precaching

## 정리

- `<link rel="prefetch">`는 브라우저가 낮은 우선순위로 리소스를 미리 가져올 수 있도록 힌트를 제공하며, 문서나 특정 리소스 파일을 대상으로 적용할 수 있다.
- Speculation Rules API는 페이지와 그 하위 리소스를 미리 가져오거나 prerendering할 수 있게 하여 사용자가 이동 시 즉각적인 로드를 지원한다.
- Service Worker를 통해 리소스를 미리 캐싱할 수 있으며, Workbox를 사용해 관리하는 것이 일반적이나 적절한 리소스 선택이 중요하다.

### 참고

- 주요 키워드: prefetching, prerendering, precaching
- 관련 기술: Service Worker Precaching, Workbox, Caching Strategies, Revisioning

## 무엇을 알았는지

### 가까운 시기에 낮은 우선순위로 prefectch

`<link rel="prefetch">` 사용시 브라우저에 가까운 시기에 리소스가 필요한 가능성이 있음을 암시한다. 이때 `가장 낮은 우선순위로 요청`이 가능하다.

```html
<head>
  <link rel="prefetch" as="script" href="/date-picker.js" />
  <link rel="prefetch" as="style" href="/date-picker.css" />
</head>
```

위 코드에서 `date-picker.js`,`date-picker.css`를 prefetch함을 알려준다. 또는 사용자와 상호작용시 동적으로 리소스를 가져와 prefetch도 가능하다.

> 참고: [Safari를 제외한 모든 최신 브라우저 지원](https://caniuse.com/link-rel-prefetch). Safari의 경우 플래그 뒤에 사용이 가능하다.

### 향후 탐색 속도 증가를 위해 페이지를 prefetch

HTML 문서를 가리킬 때 `as="document"` 속성을 지정하여 `페이지와 해당 페이지 내의 모든 하위 리소스를 미리 가져오는 것`도 가능하다.

```html
<link rel="prefetch" href="/page" as="document" />
```

크로미움 기반 브라우저에서는 [Speculation Rules API](https://developer.chrome.com/blog/prerender-pages/#the-speculation-rules-api)를 사용하여 문서를 미리 가져오는 것이 가능하다.

Speculation Rules은 페이지의 HTML에 포함된 JSON 객체로 정의되거나 Javascript를 통해 동적으로 추가된다.

```html
<script type="speculationrules">
  {
    "prefetch": [
      {
        "source": "list",
        "urls": ["/page-a", "/page-b"]
      }
    ]
  }
</script>
```

위 코드에선 `/page-a`, `/page-b`를 prefetch 하도록 지시받는다. `<link rel="prefetch">`와 마찬가지로 Speculation Rules은 특정 상황에서 브라우저가 무시할 수 있는 힌트다.

### prerendering 페이지

리소스를 미리 가져오는 것 외에도 사용자가 해당 페이지로 이동하기 전에 페이지를 미리 렌더링하도록 힌트를 줄수 있다. 이렇게 하면 페이지와 해당 리소스가 백그라운드에서 페칭되어 거의 즉시 페이지 로드가 가능하다.

```html
<script type="speculationrules">
  {
    "prerender": [
      {
        "source": "list",
        "urls": ["/page-a", "page-b"]
      }
    ]
  }
</script>
```

> 중요: 전체 prerendering은 prerendering되는 페이지에서 JavaScript도 실행한다. JavaScript는 상당히 크고 계산 비용이 많이 드는 유형의 리소스가 될 수 있으므로 `prerender`를 아껴서 사용하는 것이 좋으며, 사용자가 prerendering된 페이지로 이동하려는 것이 확실할 때만 사용하는 것이 좋다.

### Service worker precaching

Service worker를 사용하여 리소스를 미리 페치하는 것도 가능하다. Service worker precaching은 API를 사용해서 리소스를 페칭하고 저장할 수 있으므로, 브라우저가 네트워크에 가지 않고도 API를 사용하여 `Cache` 요청을 할 수 있다.

![](https://web.dev/static/learn/performance/prefetching-prerendering-precaching/image/fig-1_856.png)

> 캐시 전용 전략은 Service Worker 설치 중에 네트워크에서 적격 리소스만 검색한다. 설치가 완료되면 캐시된 리소스는 Service Worker 캐시에서만 검색된다.

서비스 워커를 사용하여 리소스를 precaching 하려면 [Workbox](https://developer.chrome.com/docs/workbox/)를 사용할 수 있다. 하지만 원하는 경우 파일 집합을 캐시하는 코드를 직접 작성도 가능하다. 서비스 워커를 사용하려면 리소스를 precaching하기로 결정한 경우 어느 쪽이든 [서비스 워커가 설치](https://developer.chrome.com/docs/workbox/service-worker-lifecycle/#installation)될 때 precaching이 발생한다는 것이 중요하다. 설치 후 precaching된 리소스는 서비스 워커가 웹사이트에서 제어하는 모든 페이지에서 검색할 수 있다.

Workbox는 [precaching 매니페스트](https://developer.chrome.com/docs/workbox/modules/workbox-precaching?hl=ko#explanation-of-the-precache-list)를 사용하여 어떤 리소스를 precaching해야하는지 결정하고 이는 "Source of Truth"역할을 하는 파일 및 정보 목록이다.

```json
[
  {
    "url": "script.ffaa4455.js",
    "revision": null
  },
  {
    "url": "/index.html",
    "revision": "518747aa"
  }
]
```

위 코드는 파일을 포함하는 매니페스트다. `/index.html`와 같이 리소스 파일 자체에 버전 정보를 포함하는 경우 `revision`속성으로 두는 것이 가능하고 버전이 지정되지 않은 `script.ffaa4455.js`의 리소스의 경우 빌드 시 해당 리소스에 대한 리비전을 생성할 수 있다.

```js
workbox.precaching.precacheAndRoute([
  "/styles/product-page.ac29.css",
  "/styles/product-page.39a1.js",
]);
```

위와 같이 서비스 워커를 사용하여, CSS와 Javascript를 precaching하여 제품 페이지를 렌더링할때 탐색을 훨씬 더 빠르게 느낄 수 있다.

> 참고: 서비스 워커와 HTTP 캐시가 사용하는 인터페이스 Cache 는 동일하지 않다. Cache인터페이스는 JavaScript가 제어하는 ​​고수준 캐시인 반면 HTTP 캐시는 Cache-Control 헤더가 제어하는 ​​저수준 캐시이다.

#### 중요

리소스 힌트나 추측 규칙을 사용하여 리소스를 prefetching하거나 prerendering하는 것과 유사하게, Service Worker precaching은 네트워크 대역폭, 스토리지, CPU를 소모한다. 사용될 가능성이 있는 리소스만 precaching하고 precaching 매니페스트에 너무 많은 리소스를 지정하지 않는 것이 좋다. 확실하지 않는 경우 적게 precaching하는 것이 많이 precaching하는 것보다 낫고, [다양한 패턴](https://developer.chrome.com/docs/workbox/caching-strategies-overview) 중 하나를 사용하여 Service Worker 캐시를 채워 속도와 리소스 신선도의 균형을 맞추는 [런타임 캐싱](https://developer.chrome.com/docs/workbox/caching-resources-during-runtime)에 의존한다.

### 출처

- [prefetching-prerendering-precaching](https://web.dev/learn/performance/prefetching-prerendering-precaching)
