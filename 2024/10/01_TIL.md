# Next.js - 증분 정적 재생성(ISR - Incremental Static Regeneration)

- 전체 사이트를 다시 빌드하지 않고 정적 콘텐츠 업데이트가 가능하다.
- 대부분의 요청에 대해 미리 렌더링된 정적 페이지를 제공하여 서버 부하를 줄인다.
- 적절한 `cache-header`가 페이지에 자동으로 추가되는지 확인한다.
- 긴 `next build` 시간 없이 대량의 콘텐츠 페이지를 처리할 수 있다.

예제.

```tsx
// app/blog/[id]/page.tsx
interface Post {
  id: string;
  title: string;
  content: string;
}

// Next.js 는 최대 60초에 한 번씩 요청이 들어올때 캐시 무효화
export const revalidate = 60;

// 빌드 시점에 `generateStaticParams`의 파라미터만 미리 렌더링
// 생성되지 않은 경로에 대한 요청이 들어오는 경우
// Next.js는 온디맨드 방식으로 서버 렌더링
export const dynamicParams = true; // 또는 알 수 없는 경로의 경우 404 설정

export async function generateStaticParams() {
  const posts: Post[] = await fetch("https://api.vercel.app/blog").then((res) =>
    res.json()
  );
  return posts.map((post) => ({
    id: String(post.id),
  }));
}

// 빌드시 게시물 생성
export default async function Page({ params }: { params: { id: string } }) {
  const post = await fetch(`https://api.vercel.app/blog/${params.id}`).then(
    (res) => res.json()
  );
  return (
    <main>
      <h1>{post.title}</h1>
      <p>{post.content}</p>
    </main>
  );
}
```

1. `next build` 중에 알려진 모든 블로그 게시물이 생성된다.
2. 이런 페이지(예: `/blog/1`)에 대한 모든 요청은 캐시되어 즉시 처리된다.
3. 60초가 지난 후에도 다음 요청에는 캐시된(오래된) 페이지가 표시된다.
4. 캐시가 무효화되고 백그라운드에서 새 버전의 페이지가 생성되기 시작한다.
5. 성공적으로 생성되면 Next.js가 업데이트된 페이지를 표시하고 캐시한다.
6. `/blog/26`이 요청되면 Next.js는 이 페이지를 온디맨드 방식으로 생성하고 캐시한다.

## 참고

### Route segment config

- [`revalidate`](https://nextjs.org/docs/app/api-reference/file-conventions/route-segment-config#revalidate)
- [`dynamicParams`](https://nextjs.org/docs/app/api-reference/file-conventions/route-segment-config#dynamicparams)

### 함수

- [`revalidatePath`](https://nextjs.org/docs/app/api-reference/functions/revalidatePath)
- [`revalidateTag`](https://nextjs.org/docs/app/api-reference/functions/revalidateTag)

## 예제

### 시간 기반 재검증

`/blog`에 있는 블로그 글 목록을 가져와 표시한다. 1시간이 지나면 다음에 페이지를 방문할 때 이 페이지의 캐시가 무효화된다. 그러면 백그라운드에서 최신 블로그 글이 포함된 새 버전의 페이지가 생성된다.

```tsx
export const revalidate = 3600; // 1시간마다 재검증

export default async function Page() {
  const data = await fetch("https://api.vercel.app/blog");
  const posts = await data.json();
  return (
    <main>
      <h1>Blog Posts</h1>
      <ul>
        {posts.map((post) => (
          <li key={post.id}>{post.title}</li>
        ))}
      </ul>
    </main>
  );
}
```

재확인 시간을 높게 설정하는 것이 좋다. 예를 들어 1초가 아닌 1시간으로 설정하는 것이 좋다. 정밀도가 더 필요하다면 온디맨드 재검증을 사용하는 것이 좋다. 실시간 데이터가 필요한 경우 [동적 렌더링](https://nextjs.org/docs/app/building-your-application/rendering/server-components#dynamic-rendering)으로 전환하는 것을 고려하자.

### `revalidatePath`를 사용한 온디맨드 재검증

보다 정확한 재검증 방법을 원한다면 `revalidatePath` 함수를 사용하여 온디맨드 방식으로 페이지를 무효화하자.

예를 들어 서버 작업은 새 글을 추가한 후에 호출된다. `fetch` 또는 데이터베이스 연결 등 서버 컴포넌트에서 데이터를 검색하는 방법에 관계없이 전체 경로에 대한 캐시를 지우고 서버 컴포넌트가 새 데이터를 가져올 수 있도록 하자.

```tsx
"use server";

import { revalidatePath } from "next/cache";

export async function createPost() {
  // 캐시에서 /posts 경로 무효화
  revalidatePath("/posts");
}
```

### `revalidateTag`를 사용한 온디맨드 재검증

대부분은 전체 경로를 검증하는 것을 선호한다. 보다 세분화된 제어가 필요하다면 `revalidatePath` 함수를 사용하자. 예를 들어 개별 `fetch` 호출에 태그를 지정할 수 있다.

```tsx
export default async function Page() {
  const data = await fetch("https://api.vercel.app/blog", {
    next: { tags: ["posts"] },
  });
  const posts = await data.json();
  // ...
}
```

ORM을 사용하거나 데이터베이스에 연결하는 경우 `unstable_cache`를 사용할 수 있다.

```tsx
import { unstable_cache } from "next/cache";
import { db, posts } from "@/lib/db";

const getCachedPosts = unstable_cache(
  async () => {
    return await db.select().from(posts);
  },
  ["posts"],
  { revalidate: 3600, tags: ["posts"] }
);

export default async function Page() {
  const posts = getCachedPosts();
  // ...
}
```

다음 서버 작업 또는 라우트 핸들러에서 `revalidateTag`를 사용할 수 있다.

```tsx
"use server";

import { revalidateTag } from "next/cache";

export async function createPost() {
  // 캐시에서 "게시물"로 태그된 모든 데이터를 무효화
  revalidateTag("posts");
}
```

### 알수 없는 예외발생 처리

데이터 재검증을 시도하는 동안 오류가 발생하면 마지막으로 성공적으로 생성된 데이터가 캐시에서 계속 제공된다. 다음 후속 요청 시 Next.js는 데이터 재검증을 다시 시도한다. 오류 처리에 대해 자세히 알아보자.

### 커스터마이징 캐시 location

페이지 캐싱 및 재검증(ISR)은 동일한 공유 캐시는 사용한다. [Vercel에 배포](https://vercel.com/docs/incremental-static-regeneration?utm_source=next-site&utm_medium=docs&utm_campaign=next-website)할 때 ISR 캐시는 자동으로 내구성 있는 스토리지에 유지된다.

셀프 호스팅 시 ISR 캐시는 Next.js 서버의 파일 시스템(디스크에 있음)에 저장된다. 이는 페이지 및 앱 라우터를 모두 사용하여 셀프 호스팅시 자동으로 작동된다.

캐시된 페이지 및 데이터를 내구성 있는 스토리지에 유지하거나 여러 컨테이너 또는 Next.js 애플리케이션 인스턴스에서 캐시를 공유하려는 경우 Next.js 캐시 위치를 구성할 수 있다. [자세히](https://nextjs.org/docs/app/building-your-application/deploying#caching-and-isr)

## 트러블 슈팅

### 로컬 개발에서 캐시된 데이터 디버깅

`fetch` API를 사용하는 경우 로깅을 추가하여 어떤 요청이 캐시되거나 캐시 해제되는지 파악할 수 있다. [`logging` 옵션에 대해 자세보자](https://nextjs.org/docs/app/api-reference/next-config-js/logging).

```tsx
// next.config.js
module.exports = {
  logging: {
    fetches: {
      fullUrl: true,
    },
  },
};
```

### 올바른 프로덕션 동작 확인

프로덕션 환경에서 올바르게 캐시되고 재검증되는지 확인하려면 `next build` 실행후 로컬로 테스트한 다음 프로덕션 Next.js 서버를 실행하면 된다.

이러면 ISR 동작을 확인할 수 있고 추가 디버깅을 위해 `.env` 파일에 `NEXT_PRIVATE_DEBUG_CACHE=1`를 추가하자

이러면 Next.js 서버 콘솔이 ISR 캐시 히트 및 미스를 기록한다. 출력 검사시 `next build` 중에 어떤 페이지가 생성되는지, 경로가 온디맨드 방식으로 액세스될 때 페이지가 어떻게 업데이트되는지 확인할 수 있다.

### Caveats

- ISR은 Node.js 런타임시만 지원(기본)
- [정적 내보내기](https://nextjs.org/docs/app/building-your-application/deploying/static-exports)를 생성할 때는 ISR이 지원되지 않는다.
- 정적으로 렌더링되는 경로에 여러 개의 `fetch` 요청이 있고 각각 재검증 빈도가 다른 경우 가장 낮은 시간이 ISR에 사용된다. 그러나 이러한 재검증 빈도는 여전히 데이터 캐시에 의해 존중된다.
- 경로에 사용된 `fetch` 요청 중 재검증 시간이 `0`이거나 명시적으로 `no-store` 요청이 있는 경우 경로가 [동적으로 렌더링](https://nextjs.org/docs/app/building-your-application/rendering/server-components#dynamic-rendering)된다.
- 미들웨어는 온디맨드 ISR 요청에 대해 실행되지 않으므로 미들웨어의 경로 재작성 또는 로직이 적용되지 않는다. 정확한 경로의 유효성을 다시 확인해야 한다. 예를들어 `/post-1` 대신 `/post/1`을 다시 작성한다.

## 출처

- [Incremental Static Regeneration(ISR)](https://nextjs.org/docs/app/building-your-application/data-fetching/incremental-static-regeneration)
