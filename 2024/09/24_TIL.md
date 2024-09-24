# Next.js의 캐싱(2) - 데이터 캐시

## 데이터 캐시

Next.js에서는 들어오는 서버 요청 및 배포에서 데이터 fetch 결과를 유지하는데 데이터 캐시가 `내장`되어 있따. 이는 네이티브 `fetch` API를 확장해서 서버의 각 요청이 자체적인 영구 캐싱 의 기능을 하도록 했기 때문이다.

> 알면 좋은점: 브라우저 fetch의 캐시 옵션은 요청이 브라우저의 HTTP 캐시와 상호 작용하는 방식을 나타내고, Next.js에서 캐시 옵션은 서버 측 요청이 서버의 데이터 캐시와 상호 작용하는 방식을 나타낸다.

캐시 동작을 구성하려면 `fetch`의 [`cache`](https://nextjs.org/docs/app/building-your-application/caching#fetch-optionscache) 및 [`next.revalidate`](https://nextjs.org/docs/app/building-your-application/caching#fetch-optionsnextrevalidate) 옵션을 사용하자.

### 데이터 캐시 작동 방식

![](https://nextjs.org/_next/image?url=%2Fdocs%2Fdark%2Fdata-cache.png&w=1920&q=75)

- 렌더링 중 `force-cache` 옵션이 포함된 fetch 요청이 처음 호출되면 Next.js는 데이터 캐시된 응답이 있는지 확인한다.
- 캐시된 응답이 발견되면 즉시 반환되어 메모에 저장된다.
- 캐시된 응답을 찾을 수 없는 경우 데이터 소스에 요청하고, 그 결과를 데이터 캐시에 저장한 후 메모화한다.
- 캐시되지 않은 데이터(예: `cache` 옵션이 정의되지 않았거나 `{ cache: 'no-store' }`사용)의 경우, 결과는 **항상 데이터 소스에서 가져와 메모화**된다.
- 데이터가 캐시되든 캐시되지 않든, 요청은 항상 메모화되어 React 렌더링 패스 중 동일한 데이터에 대한 중복 요청을 피할 수 있다.

> 참고: `데이터 캐시`와 `요청 메모화`의 차이점
>
> 두 캐싱 메커니즘 모두 캐시된 데이터를 재사용하여 성능 개선시 도움을 주지만, `데이터 캐시`는 **수신 요청과 배포에 걸쳐 지속**되는 반면, `메모화`는 **요청의 라이프사이클 동안만 지속**된다.

## 기간(duration)

데이터 캐시는 revalidate하거나 opt-out하지 않는 한 들어오는 요청과 배포에서 영구적으로 유지한다,

## 재검증(revalidate)

캐시된 데이터는 두 가지 방법으로 재검증한다.

- **Time-based 재검증**: 일정 시간 지난 후 새로운 요청이 있을 때 데이터의 유효성을 재검증한다. 이 기능은 자주 변경되지 않고 최신성이 그다지 중요하지 않은 데이터에 유용하다.
- **On-demand 재검증**: 이벤트(예: 양식 제출)를 기반으로 데이터를 재검증한다. On-demand 재검증은 태그 기반 또는 경로 기반 접근 방식을 사용하여 데이터 그룹을 한 번에 재검증 할 수 있다. 이 방법은 최신 데이터가 최대한 빨리 표시되도록 하려는 경우(예: 헤드리스 CMS의 콘텐츠가 업데이트되는 경우)에 유용하다.

### Time-based 재검증

시간 간경을 두고 데이터를 재검증하려면 `fetch`의 `next.revalidate` 옵션을 사용하여 리소스의 캐시 수명(초)를 설정할 수 있다.

```ts
// 매 시간 재검증
fetch("https://...", { next: { revalidate: 3600 } });
```

[Rout Segment Config options](https://nextjs.org/docs/app/building-your-application/caching#segment-config-options)을 사용하여 세그먼트의 모든 `fetch` 요청을 구성하거나 `fetch`를 사용할 수 없는 경우에 사용할 수 있다.

### Time-based 재검증 작동방식

![](https://nextjs.org/_next/image?url=%2Fdocs%2Fdark%2Ftime-based-revalidation.png&w=3840&q=75)

- 재검증 기능이 있는 **fetch 요청이 처음 호출**되면 외부 데이터 소스에서 데이터를 가져와 데이터 캐시에 저장한다.
- 지정된 시간(예: 60초) 내에 호출되는 모든 요청은 캐시데 데이터를 반환한다.
- 이 기간이 지나도 다음 요청은 여전히 캐시된(현재는 stale) 데이터를 반환한다.
  - Next.js는 백그라운드에서 데이터 재검증을 트리거한다.
  - 데이터가 성공적으로 가져오면, Next.js는 데이터 캐시를 새 데이트로 업데이트한다.
  - 백그라운드 재검증이 실패하면, 이전 데이터는 변경되지 않은 상태로 유지된다.

이는 [stale-while-revalidate](https://web.dev/stale-while-revalidate/)와 유사하다.

### On-demand 재검증

데이터는 경로([revalidatePath](https://nextjs.org/docs/app/building-your-application/caching#revalidatepath)) 또는 캐시 태그([revalidateTag](https://nextjs.org/docs/app/building-your-application/caching#fetch-optionsnexttags-and-revalidatetag))를 기준으로 온디맨드 방식으로 재검증이 가능하다.

### On-demand 재검증 작동방식

![](https://nextjs.org/_next/image?url=%2Fdocs%2Fdark%2Fon-demand-revalidation.png&w=3840&q=75)

- `fetch` 요청이 처음 호출되면 외부 데이터 소스에서 데이터를 가져와 데이터 캐시에 저장한다.
- on-demand 재검증이 트리거되면 해당 캐시 항목이 캐시에서 삭제된다.
  - **이는 새로운 데이터를 가져올 때까지 오래된 데이터를 캐시에 보관하는 시간 기반 재검증과는 다르다.**
- 다음에 요청이 이루어지면 다시 캐시 미스가 되고 외부 데이터 소스에서 데이터를 가져와 데이터 캐시에 저장한다.

## Opting out

`fetch` 응답을 캐시하지 않으려면 아래와 같이 수행하자.

```ts
let data = await fetch("https://api.vercel.app/blog", { cache: "no-store" });
```

## 출처

- [Caching in Next.js](https://nextjs.org/docs/app/building-your-application/caching#data-cache)
