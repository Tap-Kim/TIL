# Next.js - 텍스트 컨텐츠와 서버 렌더된 HTML간의 불일치와 해결방법

## 오류 발생이유

애플리케이션 렌더링시, 서버에서 사전 렌더링된 리액트 트리와 브라우저에서 처음 렌더링(hydration) 중에 렌더링된 리액트 트리 사이에 차이가 있다.

[하이드레이션(Hydration)](https://react.dev/reference/react-dom/client/hydrateRoot)시 리액트가 이벤트 핸들러를 연결하여 서버에서 미리 렌더링된 HTML을 완전히 상호작용 가능한 애플리케이션으로 변환하는 과정을 말한다.

## 일반적인 원인

하이드레이션 오류는 다음과 같은 이유로 발생한다.

1. HTML 태그의 잘못된 중첩

   - 다른 `<p>` 태그에 중첩된 `<p>` 태그
   - `<div>` 태그 안에 중첩된 `<p>` 태그
   - `<p>` 태그 안에 중첩된 `<ul>` 또는 `<ol>`
   - [Interactive Contents](https://html.spec.whatwg.org/#interactive-content-2)는 중첩할 수 없다.(`<a>` 태그 안에 `<a>` 태그 중첩, `<button>` 태그에 안에 `<button>` 태그 중첩 등)

2. `typeof window !== 'undefined'` 렌더링 로직과 같은 체크 사용
3. 렌덜이 로직에서 `window` 브라우저 전용 API 사용(`localStorage`)
4. `Date()` 렌더링 로직의 생성자와 같은 시간 종속 API 사용
5. [브라우저 확장 프로그램](https://github.com/facebook/react/issues/24430) HTML 수정
6. 잘못 구성된 [CSS-in-JS 라이브러리](https://nextjs.org/docs/app/building-your-application/styling/css-in-js)
7. [Cloudflare 자동 축소](https://developers.cloudflare.com/speed/optimization/content/troubleshooting/disable-auto-minify/)와 같이 HTML 응답을 수정하려는 잘못 구성된 Edge/CDN

## 수정 방법

### 해결방법 1: 클라이언트에서만 실행하기 위해 `useEffect` 사용

컴포넌트가 초기 클라이언트 측 렌더링과 동일한 서버 콘텐츠와 렌더링하여 하이드레이션 불일치를 방지한다. `useEffect`를 사용하여 클라이언트에서 의도적으로 다른 콘텐츠를 렌더링할 수 있다.

```jsx
import { useState, useEffect } from "react";

export default function App() {
  const [isClient, setIsClient] = useState(false);

  useEffect(() => {
    setIsClient(true);
  }, []);

  return <h1>{isClient ? "This is never prerendered" : "Prerendered"}</h1>;
}
```

리액트에서 하이드레이션 중에 `useEffect`가 호출된다. 즉, `window`와 같은 브라우저 API를 하이드레이션 불일치 없이 사용 가능하다.

### 해결방법 2: 특정 컴포넌트에서 SSR 비활성화

Next.js에선 특정 컴포넌트의 [SSR 비활성화](https://nextjs.org/docs/app/building-your-application/optimizing/lazy-loading#skipping-ssr)하여 하이드레이션을 방지할 수 있다.

```jsx
import dynamic from "next/dynamic";

const NoSSR = dynamic(() => import("../components/no-ssr"), { ssr: false });

export default function Page() {
  return (
    <div>
      <NoSSR />
    </div>
  );
}
```

### 해결방법 3: `suppressHydrationWarning` 사용

때로는 서버와 클라이언트 간에 타임스탬프와 같은 콘텐츠가 불가피하게 다를 수 있따. 이때 `suppressHydrationWarning={true}` 요소를 추가하여 하이드레이션 불일치 경고를 끌 수 있다.

```jsx
<time datetime="2016-10-25" suppressHydrationWarning />
```

## iOS 문제

iOS는 텍스트 콘텐츠에서 전화번호, 이메일 주소 및 기타 데이터를 감지하고 이를 링크로 변환하려고 시도하는데, 이로 인해 하이드레이션 불일치가 발생한다.

`meta` 태그를 사용하여 비활성화할 수 있다.

```jsx
<meta
  name="format-detection"
  content="telephone=no, date=no, email=no, address=no"
/>
```

## 참고

- [Text content does not match server-rendered HTML](https://nextjs.org/docs/messages/react-hydration-error#solution-3-using-suppresshydrationwarning)
