# Next.js 서버 컴포넌트 - Static Rendering, Dynamic Rendering, Streaming

React 서버 컴포넌트 사용시 서버 렌더링후 선택적 캐싱이 가능한 UI 작성이 가능하다. Next.js에서는 렌더링 작업을 `라우트 세그먼트` 별로 더 분할해서 스트리밍 및 부분 렌더링을 가능하게하고, 세 가지 다른 서버 렌더링 전략을 가진다.

- [Static Rendering](https://nextjs.org/docs/app/building-your-application/rendering/server-components#static-rendering-default)
- [Dynamic Rendering](https://nextjs.org/docs/app/building-your-application/rendering/server-components#dynamic-rendering)
- [Streaming](https://nextjs.org/docs/app/building-your-application/rendering/server-components#streaming)

이 페이지에서는 서버 컴포넌트의 작동 방식과 사용 시기, 다양한 서버 렌더링 전략에 대해 설명한다.

## 서버 렌더링의 이점

서버에서 렌더링 작업을 수행하면 다음과 같은 몇 가지 이점이 있다.

- **데이터 패칭**: 서버 컴포넌트를 사용하면 데이터 패칭을 데이터 소스에 더 가까운 서버로 옮길 수 있다. 이렇게 하면 렌더링에 필요한 데이터를 패칭하는데 걸리는 시간과 클라이언트의 요청 횟수를 줄여 성능을 향상 시킬 수 있다.
- **보안**: 서버 컴포넌트를 사용하면 토큰 및 API 키와 같은 민감한 데이터와 로직을 클라이언트에 노출할 위험 없이 서버에 보관이 가능하다.
- **캐싱**: 서버에서 렌더링하면 결과를 캐시하여 후속 요청 및 사용자 전체에서 재사용할 수 있다. 이렇게 하면 각 요청에서 수행되는 렌더링 및 데이터 패칭 양을 줄여 성능을 개선하고 비용을 절감할 수 있다
- **성능**: 서버 컴포넌트는 기본부터 성능을 최적화할 수 있는 추가 도구를 제공한다. 예를 들어, 클라이언트 컴포넌트로만 구성된 앱으로 시작하는 경우 UI의 상호작용하지 않는 부분을 서버 컴포넌트로 이동하면 필요한 클라이언트 측 자바스크립트의 양을 줄이 수 있다. 이렇게 하면 브라우저에서 다운로드, 파싱 및 실행할 클라이언트 측 자바스크립트 양이 줄어들기 때문에 인터넷 속도가 느리거나 성능이 낮은 기기를 사용하는 사용자에게 유용하다.
- **초기 페이지 로드 및 [First Contentful Paint(FCP)](https://web.dev/fcp/)**: 서버에서는 클라이언트가 페이지를 렌더링하는 데 필요한 자바스크립트를 다운로드, 파싱 및 실행할 때까지 기다릴 필요없이 사용자가 즉시 페이지를 볼 수 있도록 HTML을 생성할 수 있다.
- **SEO 및 소셜 네트워크 공유 가능성**: 렌더링된 HTML은 검색 엔진 봇이 페이지 색인을 생성하는 데 사용할 수 있고 소셜 네트워크 봇이 페이지의 소셜 카드 미리보기를 생성하는데 사용할 수 있다.
- **스트리밍**: 서버 컴포넌트를 사용하면 렌더링 작업을 청크로 분할하여 준비되는 대로 클라이언트로 스트리밍할 수 있따. 이를 통해 사용자는 서버에서 전체 페이지가 렌더링될때까지 기다릴 필요 없이 페이지의 일부를 먼저 볼 수 있다.

## Next.js에서의 서버 컴포넌트 사용

기본적으로 Next.js는 서버 컴포넌트를 사용한다. 이를 통해 추가 구성 없이 서버 렌더링을 자동으로 구현할 수 있고, 필요한 경우 [클라이언트 컴포넌트](https://nextjs.org/docs/app/building-your-application/rendering/client-components)를 사용하도록 선택할 수 있다.

## 서버 컴포넌트는 어떻게 렌더링 될까?

서버에서 Next.js는 React의 API를 사용하여 렌더링을 오케스트레이션한다. 렌더링 작업은 개별로 라우트 세그먼트와 서스펜스 바운더리별로 청크 분할된다.

각 청크는 두 단계로 렌더링 된다.

1. React는 서버 컴포넌트를 **서버 컴포넌트 페이로드(RSC 페이로드)**라는 특수 데이터 형식으로 렌더링한다. Next.js는 RSC 페이로드와 클라이언트 컴포넌트 자바스크립트 명령어를 사용하여 서버에서 HTML을 렌더링한다.
2. Next.js는 RSC 페이로드 및 클라이언트 컴포넌트 자바스크립트 명령어를 사용하여 서버에서 HTML을 렌더링한다.

그리고 클라이언트에서

1. HTML은 라우트의 빠른 상호작용하지 않는 미리보기를 즉시 표시하는데 사용되고, 이는 초기 페이지 로드에만 사용된다.
2. React 서버 컴포넌트 페이로드는 클라이언트 및 서버 컴포넌트 트리를 조정하고 DOM을 업데이트하는 데 사용된다.
3. 자바스크립트 지침은 클라이언트 컴포넌트에 [하이드레이트](https://react.dev/reference/react-dom/client/hydrateRoot)하여 애플리케이션을 대화형으로 만드는 데 사용된다.

### 리액트 서버 컴포넌트 페이로드(RSC)는 무엇인가?

_RSC 페이로드는 렌더링된 리액트 서버 컴포넌트 트리의 압축된 바이너리 표현이다._ 클라이언트에서 React가 브라우저의 DOM을 업데이트하는데 사용된다.RSC 페이로드에는 그 다음에 표시된다.

- 서버 컴포넌트의 렌더링 결과
- 클라이언트 컴포넌트가 렌더링되어야 하는 위치와 자바스크립트 파일에 대한 참조를 위한 플레이스홀더
- 서버 컴포넌트에서 클라이언트 컴포넌트로 전달된 어떤 props들

## 서버 렌더링 전략

서버 렌더링에는 세 가지 전략이 있다. 정적, 동적, 스트리밍이다.

### 정적 렌더링(기본) - Static Rendering

정적 렌더링을 사용하면 **빌드 시** 또는 데이터 재검증 후 백그라운드에서 라우트가 렌더링 된다. 그 결과는 캐시되어 CDN으로 푸시될 수 있다. 이 최적화를 통해 사용자와 서버 요청 간에 렌더링 작업 결과를 공유할 수 있다.

정적 렌더링은 정적 블로그 게시물이나 제품 페이지와 같이 라우트에 사용자를 맞춤화하지 않고 빌드 시점에알 수 있는 데이터가 있는 경우에 유용하다.

### 동적 렌더링 - Dynamic Rendering

동적 렌더링은 사용하면 **요청 시점**에 각 사용자에 대해 라우트가 렌더링 된다. 동적 렌더링은 라우트에 사용자에게 맞춤화된 데이터가 있거나 쿠키 또는 URL의 search params와 같이 요청 시점에만 알 수 있는 정보가 있는 경우에 유용하다.

#### 캐시된 데이터가 있는 동적 경로

대부분의 웹사이트에서 라우트는 완전히 정적이거나 완전이 동적인 것이 아니라 다양한 스펙트럼을 가지고 있다. 예를 들어, 주기적으로 재검증되는 캐시된 제품 데이터를 사용하지만 캐시되지 않은 개인화된 고객 데이터도 있는 이커머스 페이지를 만들 수 있다.

Next.js에서는 캐시된 데이터와 캐시되지 않은 데이터가 모두 포함된 동적으로 렌더링이되는 라우트를 만들 수 있다. 이는 RSC 페이로드와 데이터가 별도로 캐시되기 때문이다. 따라서 요청시 모든 데이터를 가져올 때 성능에 미치는 영향에 대해 걱정하지 않고 동적 렌더링을 선택할 수 있따.

[Full Route Cache](https://nextjs.org/docs/app/building-your-application/caching#full-route-cache)와 [Data Cache](https://nextjs.org/docs/app/building-your-application/caching#data-cache)에 대해 자세히 알아보자.

#### 동적 렌더링으로 전환

렌더링 중에 [동적 함수](https://nextjs.org/docs/app/building-your-application/rendering/server-components#dynamic-functions) 또는 캐시되지 않은 데이터 요청이 발견되면 Next.js는 전체 라우트를 동적으로 렌더링하도록 전환한다. 이 표에는 동적 함수와 데이터 캐싱 라우트가 정적으로 렌더링되는지 동적으로 렌더링되는지에 미치는 영향이 요약되어 있다.

| Dynamic Functions | Data       | Route                |
| ----------------- | ---------- | -------------------- |
| No                | Cached     | Statically Rendered  |
| Yes               | Cached     | Dynamically Rendered |
| No                | Not Cached | Dynamically Rendered |
| Yes               | Not Cached | Dynamically Rendered |

위 표에서 라우트가 완전히 정적으로 되려면 모든 데이터가 캐시되어야한다. 그러나 캐시된 데이터 패칭과 캐시되지 않은 데이터 패칭을 모두 사용하는 동적으로 렌더링된 라우트를 가질 수 있다. Next.js는 사용되는 기능과 API에 따라 각 경로에 가장 적합한 렌더링 전략을 자동으로 선택하므로 개발자는 정적 렌더링과 동적 렌더링 중 하나를 선택할 필요가 없다. 대신 특정 데이터를 [캐시](https://nextjs.org/docs/app/building-your-application/data-fetching/fetching)하거나 [재검증할 시기](https://nextjs.org/docs/app/building-your-application/data-fetching/incremental-static-regeneration)를 선택하고 UI의 일부를 [스트리밍](https://nextjs.org/docs/app/building-your-application/rendering/server-components#streaming)하도록 선택할 수 있다.

#### 동적 함수

동적 함수는 사용자의 쿠키, 현재 요청 헤더 또는 URL의 search params 등 요청 시점에만 알 수 있는 정보에만 의존한다. Next.js에서는 이러한 동적 API가 있다.

- [`cookies()`](https://nextjs.org/docs/app/api-reference/functions/cookies)
- [`headers()`](https://nextjs.org/docs/app/api-reference/functions/headers)
- [`unstable_noStore()`](https://nextjs.org/docs/app/api-reference/functions/unstable_noStore)
- [`unstable_after():`](https://nextjs.org/docs/app/api-reference/functions/unstable_after)
- [`searchParams prop`](https://nextjs.org/docs/app/api-reference/file-conventions/page#searchparams-optional)

이러한 함수를 사용하면 요청 시 전체라우트가 동적 렌더링으로 전환된다.

## 스트리밍

![](https://nextjs.org/_next/image?url=%2Fdocs%2Fdark%2Fsequential-parallel-data-fetching.png&w=3840&q=75)

스트리밍을 사용하면 서버에서 UI를 점진적으로 렌더링할 수 있다. 작업이 준비되는대로 청크로 분할되어 **클라이언트로 스트리밍** 된다. 이를 통해 사용자는 전체 콘텐츠의 렌더링이 완료되기 전에 페이지의 일부를 즉시 볼 수 있다.

![](https://nextjs.org/_next/image?url=%2Fdocs%2Fdark%2Fserver-rendering-with-streaming.png&w=3840&q=75)

스트리밍은 기본적으로 Next.js 앱 라우터에 내장되어 있다. 이를 통해 초기 페이지 로딩 성능과 느린 데이터 패칭에 의존하여 전체 라우트 렌더링이 차단되는 UI를 모두 개선할 수 있다. 예를 들어 페이지의 리뷰가 있다.

`loading.js`와 React Suspense가 포함된 UI 컴포넌트를 사용하여 라우트 세그먼트 스트리밍을 시작할 수 있다. [로딩 UI 및 스트리밍 섹션](https://nextjs.org/docs/app/building-your-application/routing/loading-ui-and-streaming)

## 출처

- [Server Components](https://nextjs.org/docs/app/building-your-application/rendering/server-components)
