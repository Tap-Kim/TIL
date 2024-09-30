# Next.js의 캐싱(5) - APIs

## `<Link>`

기본적으로 [`<Link>`](https://nextjs.org/docs/app/api-reference/components/link) 컴포넌트는 전체 라우트 캐시에서 라우트를 자동으로 `prefetch`하고 라우터 캐시에 React 서버 컴포넌트 페이로드를 추가한다.

`prefetch`를 비활성화하려면 `prefetch` 프로퍼티를 `false`로 설정하면 된다. 그러나 이렇게 해도 캐시를 영구적으로 무시하지 않고, 사용자가 경로를 방문할 때 경로 세그먼트는 여전히 클라이언트 캐싱이 된다.

## `router.prefetch`

[`useRouter`](https://nextjs.org/docs/app/api-reference/functions/use-router) 훅의 `prefetch` 옵션은 라우트를 수동으로 prefetch하는데 사용할 수 있다. 이러면 React 서버 컴포넌트 페이로드가 라우터 캐시에 추가된다.

## `router.refresh`

`useRouter` 훅의 `refresh` 옵션을 사용해서 경로를 수동으로 `refresh` 할 수 있다. 이러면 라우터 캐시가 완전히 지워지고 **현재 경로에 대해 서버에 새 요청**을 한다. **이때 `refresh`는 데이터 또는 Full Route Cache에는 영향을 미치지 않는다.**

## `fetch`

[`fetch`](https://nextjs.org/docs/app/api-reference/functions/fetch)에서 반환된 데이터는 데이터 캐시에 자동으로 캐싱된다.

`fetch`의 response를 캐싱하지 않으려면 아래와 같이 수행하면 된다.

```ts
let data = await fetch('https://api.vercel.app/blog', { cache: 'no-store' })
```

## `fetch options.cache`

캐시 옵션을 강제로 설정하여, 개별 fetch 캐싱으로 선택할 수 있다.

```ts
// 캐싱 설정
fetch(`https://...`, { cache: 'force-cache' })
```

## `fetch options.next.revalidate`

`fetch`의 `next.revalidate` 옵션을 사용해서 개별 `fetch` 요청의 재검증 기간(초)을 설정할 수 있다. 이렇게 하면 데이터 캐시의 유효성을 재확인하고 Full Route Cache의 유효성을 재확인한다. 새로운 데이터가 `fetch`하고 컴포넌트가 서버에서 다시 렌더링 된다.

```ts
// 1시간 이후 재검증
fetch(`https://...`, { next: { revalidate: 3600 } })
```

## `fetch options.next.tags`와 `revalidateTag`

Next.js에는 세분화된 데이터 캐싱 및 재검증을 위한 캐시 태그 시스템이 있다.
1. `fetch` 또는 `unstable_cache`를 사용할 때 하나 이상의 태그로 캐시 항목에 태그를 지정하는 옵션이 있다.
2. 그런 다음 `revalidateTag`를 호출하여 해당 태그와 연결된 캐시 항목을 제거할 수 있다.

예를 들어 데이터를 `fetch` 할 때 태그를 설정할 수 있다.

```ts
// 태그와 함께 캐시 데이터 설정
fetch(`https://...`, { next: { tags: ['a', 'b', 'c'] } })
```

그런 다음 태그와 함께 `revalidateTag`를 호출하여 캐시 항목을 제거.

```ts
// 그런 다음 태그와 함께 revalidateTag를 호출하여 캐시 항목을 제거
revalidateTag('a')
```

목적에 따라 두 가치 위치에서 `revalidateTag`를 사용할 수 있다.

1. [Route Handler](https://nextjs.org/docs/app/building-your-application/routing/route-handlers): 다른 이벤트(예: webhook)에 대한 response로 데이터를 재검증하기 위해. 라우터 핸들러는 특정 라우터에 연결되지 않으므로 라우터 캐시가 `revalidateTag`되지 않는다.

2. [Server Actions](https://nextjs.org/docs/app/building-your-application/data-fetching/server-actions-and-mutations): 사용자 작업(예: form submission) 후 데이터 유효성을 재확인한다. 이렇게 하면 연결된 라우터에 대한 라우터 캐시가 무효화된다.

## `revalidatePath`

[`revalidatePath`](https://nextjs.org/docs/app/api-reference/functions/revalidatePath)는 한 번의 작업으로 데이터를 수동으로 재검증하고 특정 라우터 아래의 라우트 세그먼트를 다시 렌더링할 수 있다. `revalidatePath` 메서드를 호출하면 데이터 캐시의 유효성이 재검증되고 Full Route Cache가 무효화된다.

```ts
revalidatePath('/')
```

목적에 따라 `revalidatePath`를 사용할 수 있는 곳은 두 가지다.

1. [Route Handler](https://nextjs.org/docs/app/building-your-application/routing/route-handlers): 다른 이벤트(예: webhook)에 대한 response로 데이터를 재검증한다.
2. [Server Actions](https://nextjs.org/docs/app/building-your-application/data-fetching/server-actions-and-mutations): 사용자 상호작용(예: form submission, 버튼 클릭) 후 데이터 재검증에 사용한다.

> `revalidatePath` vs `router.refresh`
>
> `router.refresh`를 호출하려면 라우터 캐시가 지워지고 데이터 캐시 또는 Full Route Cache가 무효화되지 않고 서버에서 라우트 세그먼트가 다시 렌더링 된다.
>
> 차이점은 `revalidatePath`는 데이터 캐시 및 Full Route Cache를 지우는 반면, `router.refresh()`는 **클라이언트 측 API**이기 때문에 데이터 캐시 및 Full Route Cache를 반경하지 않는 다는점.

## Dynamic Functions

`쿠키`와 `헤더` 같은 dynamic function, 페이지의 `searchParams` 프로퍼티는 런타임 수신 요청 정보에 따라 달라진다. 이러한 함수를 사용하면 Full Route Cache에서 라우트를 선택하면 라우트가 동적으로 렌더링된다.

### 쿠키

[쿠키](https://nextjs.org/docs/app/api-reference/functions/cookies)를 사용하는 라우트가 stale해지는 것을 방지하기 위해(예: 인증 변경 사항을 반영하는 경우) 서버 액션에서 `cookie.set` 또는 `cookie.delete`를 사용하면 라우터 캐시가 무효화된다.

### Segment Config Options

[Segment Config Options](https://nextjs.org/docs/app/api-reference/file-conventions/route-segment-config)은 라우트 세그먼트 기본값을 재정의하거나 `fetch` API(예: DB 클라이언트 또는 서드파티 라이브러리)를 사용할 수 없는 경우에 사용할 수 있다.

다음 Segment Config Options은 Full Route Cache를 Opting out한다.

- `const dynamic = 'force-dynamic'`
이 구성 옵션은 데이터 캐시에서 모든 `fetch`를 선택 해제한다.(즉, 저장하지 않는다.)

- `const fetchCache = 'default-no-store'`

더 많은 [fetchCache](https://nextjs.org/docs/app/api-reference/file-conventions/route-segment-config#fetchcache)를 확인하자.

## `generateStaticParams`

동적 세그먼트(예: `app/blog/[slug]/page.js`)의 경우 `generateStaticParams`가 제공하는 경로는 빌드 시 Full Route Cache에 캐싱된다. 요청 시점에 Next.js는 처음 방문할 때 빌드 시점에 알려지지 않은 경로도 캐시한다.

빌드 시점에 모든 경로 정적으로 렌더링하려면 `generateStaticParams`에 경로의 전체 목록을 제공하자.

```js
// app/blog/[slug]/page.js
export async function generateStaticParams() {
  const posts = await fetch('https://.../posts').then((res) => res.json())
 
  return posts.map((post) => ({
    slug: post.slug,
  }))
}
```

빌드 시점에 경로의 하위 집합을 정적으로 렌더링하고 나머지는 런타임에 처음 방문할 때 렌더링하려면 경로의 일부 목록을 반환합니다.

```js
// app/blog/[slug]/page.js
export async function generateStaticParams() {
  const posts = await fetch('https://.../posts').then((res) => res.json())
 
  // 빌드 시 처음 10개의 포스트 렌더링
  return posts.slice(0, 10).map((post) => ({
    slug: post.slug,
  }))
}
```

모든 경로를 처음 방문할 때 정적으로 렌더링하려면 빈 배열을 반환하거나(빌드 시점에 경로가 렌더링 되지 않음) `export const dynamic = 'force-static'`을 활용하자.

```js
// app/blog/[slug]/page.js
export async function generateStaticParams() {
  return []
}
```

> 알면 좋은점: 비어 있더라도 `generateStaticParams`에서 배열을 반환해야 한다. 그렇지 않으면 경로가 동적으로 렌더링된다.

```js
// app/blog/[slug]/page.js
export const dynamic = 'force-static'
```

요청시 캐싱을 비활성화 하려면 경로 세그먼트에 `export const dynamicParams = false` 옵션을 추가하자. 이 옵션을 사용하면 `generateStaticParams`에서 제공한 경로만 제공되며 다른 경로는 404 또는 일치하는 경로([catch-all routes](https://nextjs.org/docs/app/building-your-application/routing/dynamic-routes#catch-all-segments))가 된다.

## React `cache` 함수

React `cache` 함수를 사용하면 함수의 반환 값을 메모리에 저장하여 동일한 함수를 한번만 실행하면서 여러 번 호출할 수 있다.

fetch 요청은 자동으로 메모화되므로 React `cache`로 래핑할 필요가 없다. 하지만 fetch API가 적합하지 않은 경우 `cache`를 사용하여 데이터 요청을 수동으로 메모화할 수 있다. 예를 들어 일부 DB 클라이언트, CMS 클라이언트 또는 GraphQL 클라이언트가 이에 해당한다.

```ts
// utils/get-items
import { cache } from 'react'
import db from '@/lib/db'
 
export const getItem = cache(async (id: string) => {
  const item = await db.item.findUnique({ id })
  return item
})
```

## 출처

- [Caching in Next.js](https://nextjs.org/docs/app/building-your-application/caching#apis)
