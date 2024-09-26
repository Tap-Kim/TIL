# Next.js의 캐싱(4) - Client-side Router Cache & Cache Interactions

## 클라이언트 사이드 라우트 캐시 (Client-side Router Cache)

Next.js에는 레이아웃, 로딩 상태 및 페이지별로 분할된 라우트 세그먼트의 RSC 페이로드를 저장하는 in-memory 클라이언트 측 라우터 캐시가 있다.

유저가 라우트를 네비게이트할 때, Next.js는 방문한 라우트 세그먼트를 캐시하고 사용자가 탐색할 가능성이 높은 경로를 [prefetch](https://nextjs.org/docs/app/building-your-application/routing/linking-and-navigating#2-prefetching)해서 가져온다. 그 결과 즉시 앞/뒤로 네비게이트하고 네비게이트간에 전체 페이지를 다시 로드하지 않고 React 상태와 브라우저 상태를 보존할 수 있다.

라우트 캐시 사용시.

- **레이아웃**는 캐시되어 네비게이트시 재사용된다(partial rendering).
- **로딩 상태**는 캐시되어 즉시 네비게이션시 재사용된다.
- **페이지**는 기본적으로 캐시되지 않지만 브라우저 뒤로 및 앞으로 탐색시 재사용된다. 실험적인 `staleTimes` 구성 옵션을 사용하여 페이지 세그먼트에 대한 캐싱을 활성화 할 수 있다.

> 알면 좋은점: 이 캐시는 특히 Next.js 및 서버 컴포넌트에 적용되고 브라우저의 [bfcache](https://web.dev/bfcache/)와는 다르지만 결과는 비슷하다.

## 기간

캐시는 임시 메모리에 저장된다. 라우터 캐시의 지속 시간은 두 가지 요인에 의해 결정된다.

- **세션**: 캐시는 네비게이션 전반에 걸쳐 유지 된다. 그러나 새로 고침 시에는 제거된다.
- **자동 무효화 기간**: 레이아웃 및 로딩 상태의 캐시는 특정 시간이 지나면 자동으로 무효화된다. 기간은 리소스가 프리페치된 방법과 리소스가 [정적 생성](https://nextjs.org/docs/app/building-your-application/rendering/server-components#static-rendering-default)되었는지 여부에 따라 달라진다.

  - **기본 프리패칭** (`prefetch={null}` 또는 지정되지 않은): 동적 페이지의 경우 캐시되지 않고 정적 페이지인 경우 5분.
  - **전체 프리페칭** (`prefetch={true}` 또는 `router.prefetch`): 정적 및 동적 페이지 모두 5분.

  새로 고침은 캐시된 모든 세그먼트를 지우지만 자동 무효화 기간은 prefetch된 시점 부터 개별 세그먼트에만 영향을 미친다.

> 알면 좋은점: 실험적인 [`staleTimes`](https://nextjs.org/docs/app/api-reference/next-config-js/staleTimes) 옵션을 사용해서 위에서 언급한 자동 무효호화 시간을 조정할 수 있다.

## 무효화

라우터 캐시를 무효화하는 방법에는 두가지가 있다.

- **서버 액션**
  - path 기준([revalidatePath](https://nextjs.org/docs/app/api-reference/functions/revalidatePath)) 또는 캐시 태그 기준([revalidateTag](https://nextjs.org/docs/app/api-reference/functions/revalidateTag))으로 on-demand 데이터 검증
  - 오래된 것들(예: 인증)로 부터 쿠키를 사용하는 라우트는 [`cookie.set`](https://nextjs.org/docs/app/api-reference/functions/cookies#methods) 또는 [`cookie.delete`](https://nextjs.org/docs/app/api-reference/functions/cookies#methods)를 사용하면 캐시가 무효화된다.
- [`router.refresh`](https://nextjs.org/docs/app/api-reference/functions/use-router)를 호출하면 라우터 캐시가 무효화 되고 현재 경로에 대한 새 요청이 서버에 전송된다.

## 캐시 상호 작용(Cache Interactions)

### 데이터 캐시와 전체 경로 캐시

- 렌더링 출력은 데이터에 따라 달라지므로 데이터 캐시의 유효성을 다시 검사하거나 데이터 캐시를 선택 해제하면 전체 경로 캐시가 무효화된다.
- 전체 경로 캐시를 무효화하거나 opting out해도 데이터 캐시에는 영향을 미치지 않는다. 캐시된 데이터와 캐시되지않은 데이터가 모두 포함된 경로를 동적으로 렌더링할 수 있다.
- 페이지 대부분이 캐시된 데이터를 사용하지만 요청 시점에 데이터를 가져와야하는 몇 가지 컴포넌트가 있는 경우에 유용하다. 모든 데이터를 다시 가져올 때 성능에 미치는 영향에 대해 걱정하지 않고 동적으로 렌더링할 수 있다.

### 데이터 캐시와 클라리언트 사이드 라우터 캐시

- 데이터 캐시 및 라우터 캐시를 즉시 무효화하려면 서버 작업에서 revalidatePath 또는 revalidateTag를 사용하면 된다.
- [라우트 핸들러](https://nextjs.org/docs/app/building-your-application/routing/route-handlers)는 특정 경로에 연결되어 있지 않으므로 라우터 핸들러에서 데이터 캐시의 유효성을 재확인해도 라우터 캐시가 즉시 무효화되지 않는다. 즉, 라우터 캐시는 강력 새로 고침 또는 자동 무효화 기간이 경과할 때까지 이전 페이로드를 계속 제공한다.

## 출처

- [Caching in Next.js](https://nextjs.org/docs/app/building-your-application/caching#client-side-router-cache)
