# Next.js의 캐싱(1)

## 들어가면서

| Mechanism           | What                    | Where  | Purpose                                  | Duration                   |
| ------------------- | ----------------------- | ------ | ---------------------------------------- | -------------------------- |
| Request Memoization | 함수의 반환 값          | Server | React Component 트리에서 데이터를 재사용 | 요청 라이프사이클 동안     |
| Data Cache          | 데이터                  | Server | 사용자 요청 및 배포 간 데이터 저장       | 지속적 (재검증 가능)       |
| Full Route Cache    | HTML 및 RSC 페이지 로드 | Server | 렌더링 비용 절감 및 성능 향상            | 지속적 (재검증 가능)       |
| Router Cache        | RSC 페이지 로드         | Client | 네비게이션 시 서버 요청 감소             | 사용자 세션 또는 시간 기반 |

> 알면 좋은점: Next.js의 캐싱 휴리스틱은 대부분 API 사용량에 결정되고 최소한의 구성으로 최상의 성능을 낼 수 있는 기본값으로 [Data Fetching and Caching](https://nextjs.org/docs/app/building-your-application/data-fetching/fetching)의 정보가 있다.

Next.js에선 기본적으로 성능 개선과 비용 절감을 위해 가능하면 캐시를 한다. 즉, 사용자가 opt out 하지 않는한 경로가 정적으로 렌더링 되고 데이터 요청이 캐시된다. 아래 다이어그램은 **빌드 시 경로가 정적으로 렌더링되는 경우**와 **정적 경로가 처음 방문 되는 경우** 기본 캐싱 동작을 보여준다.

![](https://nextjs.org/_next/image?url=%2Fdocs%2Fdark%2Fcaching-overview.png&w=1920&q=75)

캐싱 동작은 경로가 정적으로 렌더링 되는지 동적으로 렌더링되는지, 데이터가 캐시되는지 캐시되지 않는지, 요청이 초기 방분의 일부인지 후속 탐색의 일부인지에 따라 달라지게 된다. 각 사례별로 경로 및 데이터 요청에 대한 캐싱 동작을 구성할 수 있다.

Next.js는 URL과 옵션이 동일한 요청을 자동으로 메모화하도록 [fetch API](https://nextjs.org/docs/app/building-your-application/caching#fetch)를 확장한다. 즉, React 컴포넌트 트리의 여러 위치에서 동일한 데이터에 대한 fetch 함수를 한 번만 실행하면서 호출할 수 있다.

![](https://nextjs.org/_next/image?url=%2Fdocs%2Fdark%2Fdeduplicated-fetch-requests.png&w=1920&q=75)

예를 들면 레이아웃, 페이지, 여러 컴포넌트 등 경로 전체에서 동일한 데이터를 사용해야하는 경우 트리 최상단에서 데이터를 가져오거나 컴포넌트 간에 props를 전달할 필요가 없다. 대신 동일 데이터에 대한 네트워크를 통해 여러 번 요청시 성능에 미치는 여향에 댛 ㅐ걱정할 필요 없이 필요한 컴포넌트에서 데이터를 가져올 수 있다.

```tsx
// app/example.tsx
async function getItem() {
  // `fetch` 함수는 자동으로 메모화되고 결과는 캐시된다.
  const res = await fetch("https://.../item/1");
  return res.json();
}

// getItem은 두번 호출되나 처음 한 번만 실행된다.
const item = await getItem(); // cache MISS

// 두 번째 호출 경로의 어느 곳에서나 가능하다.
const item = await getItem(); // cache HIT
```

### 요청(Request) 메모화 작동 방식

![](https://nextjs.org/_next/image?url=%2Fdocs%2Fdark%2Frequest-memoization.png&w=1920&q=75)

- 라우트 렌더링 동안 특정 요청이 처음 호출되면 그 결과는 메모리에 없고 `cache MISS`가 된다.
- 따라서 함수가 실행되고 외부 소스에서 데이터를 가져와 그 결과가 `메모리`에 저장된다.
- 동일한 `render pass`에서 요청의 후속 함수 호출은 `cache HIT`가 되고 함수를 실행하지 않고 `메모리에서 데이터가 반환`된다.
- 경로가 렌더링되고 `rendering pass`가 완료되면 메모리는 `리셋`되고 모든 요청 메모화 항목이 지워진다.

> 알면 좋은점

    - 요청 메모화는 React 기능이다. 다른 캐싱 메커니즘과 상호작용하는 방법을 위해 보여주었다.
    - 메모화는 fetch 요청의 `GET` 메서드에서만 적용된다.
    - 메모화는 React 컴포넌트 트리에만 적용된다.
        - generateMetadata, generateStaticParams, Layouts, Pages, 그리고 다른 Server Components의 요청에 적용된다.
        - 일부 데이터베이스 클라이언트, CMS 클라이언트 또는 GraphQL 클라이언트와 같이 fetch가 적합하지 않는 경우 [React `cache` 함수](https://nextjs.org/docs/app/building-your-application/caching#react-cache-function)를 사용하여 함수를 메모화한다.

### 기간

캐시는 **React 컴포넌트 트리가 렌더링을 완료할 때**까지 서버 요청의 수명 동안 지속된다.

### 재검증

메모화는 **서버 요청간에 공유되지 않고 렌더링 중에만 적용**되므로 재검증에 필요가 없다.

### Opting Out

메모화는 fetch 요청의 `GET` 메소드만 적용되고 `POST, DELETE`와 같은 다른 메서드에서는 메모화되지 않는다. 이는 기본 동작으로 React 최적화를 위한 것이므로 opt out 하지 않는 것이 좋다.

개별로 요청을 관리하려면 `AbortController`의 `signal` 프로퍼티를 사용할 수 있다. 하지만 이러면 메모화에서 요청을 opt out하는 것이 아니라 in-flight 요청을 중단하게 된다.

```ts
// app/example.js
const { signal } = new AbortController();
fetch(url, { signal });
```

## 출처

- [Caching in Next.js](https://nextjs.org/docs/app/building-your-application/caching#overview)
