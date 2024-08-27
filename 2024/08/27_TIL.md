# 이미지 성능

## 정리

- 이미지는 리소스 중 가장 무겁기 때문에 이미지 최적화시 성능을 크게 개선이 가능하다. 이미지 최적화라함은 더 작은 바이트를 전송해서 네트워크 시간을 의미하지만 사용자 기기에 적절한 이미지를 제공해서 전송되는 바이트 양을 최적화화가 가능하다.
- 이미지 사이즈, 올바른 크기(치수)로 표시하는 것. DRP(device pixel ratio)를 기준으로 디바이스별로 최적의 이미지 크기를 선별하는 것이 좋다.
- `srcset, sizes, 파일 포멧, 압축, 레이지 로딩, 디코딩`을 통해 네트워크 대역폭을 최적화한다던지 디바이스 너비를 통해 이미지 크기를 선별하여 리소스를 최적화할 수 있다.

### 참고

- 주요 키워드: img, source, picture, srcset, sizes, 압축, 파일 포멧, lazy loading, decoding, LCP
- 관련 기술: 이미지 최적화, 손실/무손실 압축

## 무엇을 알았는지

이미지는 리소스 중 가장 무겁기 때문에 이미지 최적화시 성능을 크게 개선이 가능하다. 이미지 최적화라함은 더 작은 바이트를 전송해서 네트워크 시간을 의미하지만 사용자 기기에 적절한 이미지를 제공해서 전송되는 바이트 양을 최적화화가 가능하다.

이미지는 `<img>`, `<picture>` 요소, 또는 CSS `background-image`를 사용이 가능하다.

> 참고: SVG는 별개인데, 이는 HTML에 직접 삽입한 인라인으로 적용될 수 있다. 이를 통해서 Javascript를 통해서 SVG 자식 요소에 접근이 가능하고 이 모듈은 이미지 성능에 초점이 맞출 수 있다. 또한 SVG 사용시 벡터 이미지 형식이므로 내용이 line art, 다이어그램이나 차트와 같은 세부적인 이미지가 아닐때 유용하고, 텍스트 기반이기 때문에 최소화 및 압축이 적용된다. (이번 장에선 다루지 않는다.)

### `srcset`

`srcset`은 브라우저가 사용할 수 있는 이미지 소스 목록을 지정할 수 있는 속성을 지원하고 지정된 각 이미지 소스에는 이미지 URL과 너비 또는 픽셀 밀도가 포함된다.

```html
<img
  alt="An image"
  width="500"
  height="500"
  src="/image-500.jpg"
  srcset="/image-500.jpg 1x, /image-1000.jpg 2x, /image-1500.jpg 3x"
/>
```

위 HTML은 픽셀 밀도를 사용해서 `image-500.png`의 DPR이 1인 장치, `image-1000.jpg`의 DPR이 2인 장치, `image-1500.jpg`의 DPR이 3인 장치를 사용하도록 힌트를 제공한다.(페이지 레이아웃은 다른 고려사항)

### `sizes`

`sizes`는 모든 뷰포트에서 동일한 CSS 픽셀 크기로 이미지를 표시하는 경우에 작동한다. 많은 경우 레이아웃과 컨테이너 크기는 사용자 기기에 따라 달라진다.

이 요소를 사용하면 소스 크기의 집합을 지정할수 있고 각 크기는 미디어 조건과 값으로 구성된다. 이 속성은 CSS 픽셀로 의도된 이미지 크기를 표시하고, 너비 설명자와 결합하면 브라우저는 사용자 기기에 적합안 이미지 소스를 선택한다.

```html
<img
  alt="An image"
  width="500"
  height="500"
  src="/image-500.jpg"
  srcset="/image-500.jpg 500w, /image-1000.jpg 1000w, /image-1500.jpg 1500w"
  sizes="(min-width: 768px) 500px, 100vw"
/>
```

위 HTML은 이미지 후보군(`srcset`)을 `,`으로 구분했고, 목록의 후보군(`sizes`)는 이미지의 URL과 이미지 고유 너비를 나타내었다.
1000w 이미지의 고유 너비가 1000px 너비를 나타내고, 뷰포트 너비가 768px 초과시 이미지가 500px 너비로 표시하고 더 작은 뷰포트에는 이미지가 `100vw` 또는 전체 뷰포트로 표시된다.

그런다음 브라우저는 이 정보를 이미지 소스 목록과 결합하여 `srcset`의 최적의 이미지를 찾는다. 예를 들어 사용자 너비가 320px이고 DPR이 3인 모바일 기기 사용시 이미지는 `320 CSS pixel x 3 DPR = 960 device pixel`이다. 여기서 가장 가까운 이미지는 고유 너비가 1000px(1000vw)인 `image-1000.jpg`이 된다.

### 파일 포멧

브라우저는 여러 이미지 파일 형식을 지원하고, `WebP`, `AVIF`와 같은 최신 이미지 형식은 `PNG`, `JPEG` 보다 나은 압축을 제공하여 파일 크기를 작고 네트워크 시간을 줄일 수 있다. 이는 리소스 시간을 줄이고 LCP를 낮출수 있다.

### 압축

1. 손실 압축: [양자화](https://cs.stanford.edu/people/eroberts/courses/soco/projects/data-compression/lossy/jpeg/coeff.htm)를 통해 이미지 정확도를 낮추고 [크로마 서브샘플링](https://en.wikipedia.org/wiki/Chroma_subsampling)을 사용해서 추가 색상 정보를 삭제할 수 있. 노이즈와 색상이 많은 고밀도 이미지에서 가장 효과 적이다. 손실 압축으로 생성된 [아티팩트가](https://en.wikipedia.org/wiki/Compression_artifact) 세부 이미지에선 눈에 띄지 않을 가능성이 낮다. (JPEG, WebP 및 AVIF에 적합)
2. 무손실 압축: 데이터 손실 없이 파일 크기를 줄이고 주변 픽셀과 차이에 따라 픽셀을 결정한다. (GIF, PNG, WebP 및 AVIF에 적합)

[squoosh](https://squoosh.app/), [ImageOptim](https://imageoptim.com/) 또는 다른 이미지 최적화 서비스를 이용하여 압축을 이용 가능하다.

### `<picture>` 요소

`<picture>`는 여러 이미지 후보를 지정하면서 유연성을 제공한다.

```html
<picture>
  <source type="image/avif" srcset="image.avif" />
  <source type="image/webp" srcset="image.webp" />
  <img alt="An image" width="500" height="500" src="/image.jpg" />
</picture>
```

브라우저 환경에 따라 위에서부터 순서대로 확장자를 제공하는 순서로 노출된다. 지원되지 않는다면 아래로 계속 내려가면서 `<source>`에 더이상 호환되지 안흔다면 `<img>`로 대체된다.

```html
<!-- DPR별로 뷰포트 대비 source에서 srcset, sizes을 사용하여 세밀하게 조정이 가능하다. -->
<picture>
  <source
    media="(min-resolution: 1.5x)"
    srcset="/image-1000.jpg 1000w, /image-1500.jpg 1500w"
    sizes="(min-width: 768px) 500px, 100vw"
  />
  <img alt="An image" width="500" height="500" src="/image-500.jpg" />
</picture>

<picture>
  <source
    media="(min-width: 560px) and (min-resolution: 1.5x)"
    srcset="/image-1000.jpg 1000w, /image-1500.jpg 1500w"
    sizes="(min-width: 768px) 500px, 100vw"
  />
  <source
    media="(max-width: 560px) and (min-resolution: 1.5x)"
    srcset="/image-1000-sm.jpg 1000w, /image-1500-sm.jpg 1500w"
    sizes="(min-width: 768px) 500px, 100vw"
  />
  <img alt="An image" width="500" height="500" src="/image-500.jpg" />
</picture>

<picture>
  <source
    media="(min-width: 560px)"
    srcset="/image-500.jpg, /image-1000.jpg 2x, /image-1500.jpg 3x"
  />
  <source
    media="(max-width: 560px)"
    srcset="/image-500.jpg 1x, /image-1000.jpg 2x"
  />
  <img alt="An image" width="500" height="500" src="/image-500.jpg" />
</picture>
```

`<source>`는 `media, srcset, 및 sizes`을 제공한다.

### Lazy loading

뷰포트에 이미지가 나타날때 `loading` 속성으로 브라우저에 지연 로드를 지시할수있다. `lazy` 속성 값으로 제어하고 근처에 있을때까지 이미지를 다운받지 않는다. 이렇게 되면 네트워크 대역폭을 절약하여 중요한 리소스를 우선시할 수 있다.

### decoding

`decoding`은 이미지를 어떻게 디코딩해야하는가 알려주는데 `async`로 이미지를 비동기적으로 디코딩할수 있음을 알려주고 다른 콘텐츠를 렌더링하는데 시간을 개선시킬 수 있다. `sync` 값은 브라우저에 이미지를 다른 콘텐츠와 동시에 표시해야한다. 기본값인 `auto`는 브라우저가 사용자에게 적합한 것을 자동으로 결정한다.

> 참고: 디코딩은 Javascript에서 DOM에 이미지를 삽입할때 인스턴스에서 decode 메서드 사용이 가능하다.

### 출처

- [Image performance](https://web.dev/learn/performance/image-performance)
