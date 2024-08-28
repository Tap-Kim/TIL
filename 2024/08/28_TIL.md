# 비디오 퍼포먼스

## 정리

- 비디오는 파일 작업시 운영체제(`.mp4, .webpm` 등)의 파일을 컨테이너라고 부른다. 이는 하나 이상의 `Stream`이 포함되어있고 대부분 비디오나 오디오 스트림을 가진다.
- 코덱을 사용하여 각 스트림을 압축할 수 있다. `video.webm`은 압축된 비디오 스트림은 VP9. 오디오 스트림은 Vorbis를 사룔하여 WebM이 컨테이너가 될 수 있다.
- 컨테이너나 코덱의 차이점은 적은 대역폭을 사용하여 파일을 압축한다는 개념이고 전체 페이지 로드 시간을 단축하고 LCP를 개선할 수 있다.
- GIF와 같이 자동재생 또는 애니메이션 구현을 위해 대역폭이 큰 영상 다운로드 리소스를 `poster`와 `IntersectionObserver API`와 같이 첫 로드를 이미지로 대체하여 UX 개선과 리소스 개선할 수 있다.
- Youtube나 Videom과 같은 영상 플랫폼을 임베디드시 관련된 리소스를 다운받는데 있어서 UX에 영향이 갈 수 있다. 경우에 따라선 youtube는 `lite-youtube-embed`이라는 경량화된 임베디드 방식을 사용 가능하다.

### 참고

- 주요 키워드: video, source, poster, 코덱, autoplayer, preload, embeds, LCP
- 관련 기술: 비디오 최적화, IntersectionObserver

## 무엇을 알았는지

### 다양한 형식

```html
<video>
  <source src="video.webm" type="video/webm" />
  <source src="video.mp4" type="video/mp4" />
</video>
```

모든 최신 브라우저가 H.264 코덱을 지원하므로 MP4 를 레거시 브라우저의 대체 수단으로 사용할 수 있다. WebM 버전은 아직 널리 지원되지 않는 최신 AV1 코덱 이나 AV1보다 더 잘 지원 되지만 일반적으로 AV1만큼 잘 압축하지 않는 이전 VP9 코덱을 사용할 수 있다.

> 참고: `<picture>`와 마찬가지로 나열하는 순으로 브라우저의 우선순위를 결정할수 있다.

### `poster` 속성

`poster` 속성은 재생이 시작전 이미지로 힌트를 제공한다.

```html
<video poster="poster.jpg">
  <source src="video.webm" type="video/webm" />
  <source src="video.mp4" type="video/mp4" />
</video>
```

### `autoplay`

HTTP 아카이브에 따르면 웹상의 비디오 중 20%에 `autoplay` 속성이 포함되어있다.(비디오 배경이나 GIF를 대체 가능하다)

애니메이션 GIF는 매우 클 수 있고, 그에 따라 리소스 대역폭을 상당히 소모가 가능하다. 따라서 일반적으로 애니메이션 이미지 형식은 피해야한다.

```html
<video autoplay muted loop playsinline>
  <source src="video.webm" type="video/webm" />
  <source src="video.mp4" type="video/mp4" />
</video>
```

`poster`와 [Intersection Observer API](https://developer.mozilla.org/docs/Web/API/Intersection_Observer_API)를 결합하면 [비디오가 뷰포트 내에 있을때만 비디오를 다운로드](https://web.dev/articles/lazy-loading-video#video-gif-replacement)하도록 구성할수 있다. 이미지는 첫 번째 프레임과 같이 품질이 낮은 이미지 플레이스 홀더일 수 있다. 이렇게하면 초기 뷰포트의 콘텐츠 로드 시간을 개선이 가능하지만 `poster` 이미지와 `autoplay` 사용시 비디오가 로드되고 재싱되기 전까지 순간적으로 사용자에게 노출이 된다.(UX를 고려하여 적용 여부를 판단하자)

- poster와 Intersection Observer API 결합시
  ![](https://cdn.glitch.global/97616b87-f930-4eb0-a8a0-84c6a73d97e7/gif-1.webm?v=1671616580215)

  ```html
  // index.html
  <video
    class="lazy"
    preload="none"
    muted
    controls
    loop
    playsinline
    width="640"
    height="1138"
    poster="https://cdn.glitch.global/97616b87-f930-4eb0-a8a0-84c6a73d97e7/gif-1.jpg?v=1671616580215"
  >
    <source
      src="https://cdn.glitch.global/97616b87-f930-4eb0-a8a0-84c6a73d97e7/gif-1.webm?v=1671616580215"
      type="video/webm"
    />
    <source
      src="https://cdn.glitch.global/97616b87-f930-4eb0-a8a0-84c6a73d97e7/gif-1.mp4?v=1671616580215"
      type="video/mp4"
    />
  </video>
  ```

  ```js
  // script.js
  document.addEventListener("DOMContentLoaded", function () {
    var lazyVideos = [].slice.call(document.querySelectorAll("video.lazy"));
    if ("IntersectionObserver" in window) {
      var lazyVideoObserver = new IntersectionObserver(function (
        entries,
        observer
      ) {
        entries.forEach(function (video) {
          if (video.isIntersecting) {
            video.target.autoplay = true;
            video.target.controls = false;
            video.target.load();
            lazyVideoObserver.unobserve(video.target);
          }
        });
      });

      lazyVideos.forEach(function (lazyVideo) {
        lazyVideoObserver.observe(lazyVideo);
      });
    }
  });
  ```

### `preload`

일반적으로 브라우저는 `video` 요소 발견 즉시 다운로드한다. 이때 `preload` 속성을 사용하여 `none`을 사용하여 미리 로드를 막거나, `metadata` 설정으로 비디오 길이와 대략정이 정보를 통해 메타데이터를 가져올 수 있다.(특정 브라우저나 모바일 기기에 호환이 안될 가능성이 있으니 유의하자)

### `fetchpriority`

이미지와 마찬가지로 LCP 요소를 예상하여 비디오의 순위를 매길수 있다.

```html
<link rel="preload" as="image" href="poster.jpg" fetchpriority="high" />
```

### Embeds

효율적으로 최적화 하고 복잡성을 고려시 Youtube나 Vimeo와 같은 영상 플랫폼을 사용할 수있다. 각 플랫폼에서 비디오 최적화를 진행하지만 임베드도ㅚㄴ 플레이어는 Javascript와 같이 많은 추가 리소스를 다운받아야해서 성능에 영향을 미치게 된다. Youtube 같은 경우는 메인 스레드를 1.7초 이상 차단하고 INP에 영향을 미치기 때문에 UX에 영향이 크게 간다.

이를 해결하기 위해 [lite-youtube-embed](https://github.com/paulirish/lite-youtube-embed)와 같은 라이브러리를 사용하게 되면 대역폭을 크게 줄일 수 있다.

- 라이브러리 사용시

  ![](https://cdn.glitch.global/97616b87-f930-4eb0-a8a0-84c6a73d97e7/lite-youtube-embed.png?v=1671622156330)

- 미사용시

  ![](https://cdn.glitch.global/97616b87-f930-4eb0-a8a0-84c6a73d97e7/youtube.png?v=1671622156330)

### 출처

- [Video performance](https://web.dev/learn/performance/video-performance)
