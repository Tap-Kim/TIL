# Next.js의 캐싱(3) - Full Route Cache

# 전체 라우트 캐시(Full Route Cache)

> 관련 용어: 자동 정적 최적화(Automatic Static Optimization), 정적 사이트 생성(Static Site Generation) 또는 정적 렌더링(Static Rendering)이라는 용어는 `빌드 시 애플리케이션의 경로를 렌더링하고 캐싱하는 프로세스를 지칭할 떄 혼용되어 사용이 가능하다.`

Next.js는 빌드 시점에서 자동으로 경로를 렌더링하고 캐싱한다. 모든 요청에 대해 서버에서 렌더링하는 대신 캐시된 경로를 제공하여 페이지 로딩 속도를 높이는 최적화이다.

전체 경로 캐시의 작동 방식을 이해하려면 React가 렌더링 처리하는 방식과 Next.js가 결과를 캐시하는 방식을 살펴보는 것이 도움이 된다.

## 1. 서버에서 React 렌더링

Next.js는 서버에서 React API를 사용하여 렌더링을 오케스트레이션한다. 렌더링 작업은 개별 경로 세그먼트와 서스펜스 바운더리로 나뉜다.

각 두 단계로 렌더링된다.

1. React는 서버 컴포넌트를 스트리밍에 최적화된 특수 데이터 형식으로 **React 서버 컴포넌트 페이로드(React Server Component Payload)**라는 이름으로 렌더링 된다.
2. Next.js는 React 서버 컴포넌트 페이로드와 클라이언트 컴포넌트 Javascript 명령어를 사용하여 서버에서 HTML을 렌더링 한다.

즉, 작업을 캐싱하거나 응답을 전송하기 전에 모든 렌더링이 완료될 때까지 기다릴 필요가 없다. 대신 작업이 완료되는 즉시 응답을 스트리밍할 수 있다.

> React 서버 컴포넌트 페이로드란>
>
> React 서버 컴포넌트 페이로드는 렌더링된 React **서버 컴포넌트 트리의 압축된 바이너리** 표현이다. 클라이언트에서 React가 브라우저의 DOM을 업데이트하는데 사용된다.

- 서버 컴포넌트의 렌더링 결과 클라이언트 컴포넌트가 렌더링될 위치에 대한 Placeholder와 해당 Javascript 파일에 대한 참조
- 서버 컴포넌트에서 클라이언트 컴포넌트로 전달되는 모든 props

[Server Components](https://nextjs.org/docs/app/building-your-application/rendering/server-components) 문서 참조.

## 2. Next.js 서버 캐싱(전체 라우트 캐시)

![](https://nextjs.org/_next/image?url=%2Fdocs%2Fdark%2Ffull-route-cache.png&w=3840&q=75)

Next.js의 기본 동작은 **서버에서 라우트의 렌더링된 결과**(React 서버 컴포넌트 페이로드란 및 HTML)를 캐시한 것이다. 이는 빌드 시 또는 재검증 중에 정적으로 렌더링된 경로에 적용된다.

## 3. 클라이언트에 대한 React Hydration 및 Reconciliation

클라이언트에서 요청시

1. HTML은 클라이언트 및 서버 컴포넌트의 빠른 비상호작용 초기 프리뷰를 즉시 표시하는데 사용된다.
2. React 서버 컴포넌트 페이로드는 클라이언트 및 렌더링된 서버 컴포넌트 트리를 조정하고 DOM을 업데이트하는데 사용된다.
3. Javascript 지침은 클라리언트 컴포넌트에 [hydrate](https://react.dev/reference/react-dom/client/hydrateRoot)하여 애플리케이션을 대화형으로 만드는데 사용된다.

## 4. 클라이언트에서 Next.js 캐싱(Router Cache)

React 서버 컴포넌트 페이로드는 개별 라우트 세그먼트별로 분할된 별도의 `in-memory 캐시`인 클라리언트 측 [Router Cache](https://nextjs.org/docs/app/building-your-application/caching#client-side-router-cache)에 저장된다.

## 5. 후속 네비게이션

후속 네비게이션 또는 프리페칭 중에 Next.js는 React 서버 컴포넌트 페이로드가 라우터 캐시에 저장되어 있는지 확인한다. 그렇다면 서버에 새 요청을 보내는 것을 건너 뛴다.

라우트 세그먼트가 캐시에 없는 경우 Next.js는 서버에서 React 서버 컴포넌트 페이로드를 가져와서 클라리언트의 라우터 캐시를 채운다.

## 정적(Static) 및 동적(Dynamic) 렌더링

빌드 시점에 라우트가 캐시되었는지에 대한 여부는 라우트가 정적으로 렌더링되는지 동적으로 렌더링되는지에 따라 달라진다. `정적 라우트는 기본적으로 캐시`되는 반면, `동적 라우트는 요청 시 렌더링`되며 캐시되지 않는다.

아래 다이어그램은 캐시된 데이터와 캐시되지 않은 데이터를 사용하여 정적 경로와 동적 경로의 차이점을 보여준다.

![](https://nextjs.org/_next/image?url=%2Fdocs%2Fdark%2Fstatic-and-dynamic-routes.png&w=3840&q=75)

여기서 [정적(Static) 및 동적(Dynamic) 렌더링](https://nextjs.org/docs/app/building-your-application/rendering/server-components#server-rendering-strategies)에 대해 자세하게 살펴볼 수 있다.

## 기간

기본적으로 전체 라우트 캐시는 `영구적`이다. 즉, 렌더링 아웃풋은 사용자 요청에 걸쳐 캐시된다.

## 무효화

전체 라우트 캐시를 무효화 하는 두가지 방법.

- [데이터 재검증](https://nextjs.org/docs/app/building-your-application/caching#revalidating): [데이터 캐시](https://nextjs.org/docs/app/building-your-application/caching#data-cache)의 유효성을 검사하면 서버에서 컴포넌트를 다시 렌더링하고 새 렌더링 결과를 캐싱하여 라우터 캐시가 무효화된다.
- [재배포]: 배포 간에 유지되는 데이터 캐시와 달리 전체 라우트 캐시는 새 배포 시 제거된다.

## Opting out

_전체 라우트 캐시를 사용하지 않도록 설정하거나, 들어오는 모든 요청에 대해 컴포넌트를 동적으로 렌더링하는 방법을 선택할 수 있다._

- [Dynamic Function](https://nextjs.org/docs/app/building-your-application/caching#dynamic-functions) 사용: 이렇게하면 전체 라우트 캐시에서 경로를 선택 해제하고 요청 시점에 동적으로 렌더링한다. 데이터 캐시는 계속 사용이 가능하다.
- `dynamic = 'force-dynamic'` 또는 `revalidate = 0` 라우트 세그먼트 구성 옵션 사용: 전체 라우트 캐시와 데이터 캐시를 건너뛴다. **즉, 서버로 들어오는 모든 요청에 대해 컴포넌트가 렌더링되고 데이터를 가져온다. 라우터 캐시는 클라이언트 측 캐시이므로 계속 적용** 된다.
- [데이터 캐시](https://nextjs.org/docs/app/building-your-application/caching#data-cache)에서 Opting out: 라우트에 캐시되지 않은 `fetch` 요청시 전체 라우트 캐시에서 해당 경로를 Opting out한다. 특정 `fetch` 요청에 대한 데이터는 들어오는 모든 요청에 대해 가져오게된다. 캐싱을 Opting out하지 않은 다른 `fetch` 요청은 여전히 데이터 캐시에 캐싱된다. 이렇게 하면 캐시된 데이터와 캐시되지 않은 데이터를 혼합하여 사용할 수 있다.

## 출처

- [Caching in Next.js](https://nextjs.org/docs/app/building-your-application/caching#full-route-cache)
