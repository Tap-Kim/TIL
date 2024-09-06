# Web Worker의 사례 (feat. Exif 메타데이터 제거)

## 정리

- Web Worker는 `fetch` 호출을 격리하고 응답을 처리하거나 메인 스레드를 차단하지 않고 대량의 데이터를 처리하는 등 모든 종류의 작업에 사용할 수 있다. 웹 애플리케이션의 성능을 개선할 때는 Web Worker 환경에서 합리적으로 수행할 수 있는 모든 것을 생각하자.

### 참고

- 주요 키워드: Web Worker, INP, GPU Thread, Single Thread, Multi Thread
- 관련 기술: Multi Thread, Messaging Pipeline

## 무엇을 알았는지

[Web Worker의 개요](https://github.com/Tap-Kim/TIL/blob/main/2024/09/05_TIL.md)에 대해 설명했으니 사례에 대해 알아보자. 앞서 Web Worker는 Javascript를 메인 스레드에서 분리된 Web Worker 스레드로 옮겨서 입력 응답성을 개선하고, 메인 스레드에 직접 접근할 필요가 없는 작업이 있을 때 INP를 개선하는데 도움이 된다.

예를들어 하나로 이미제엇 [Exif 메타데이터](https://en.wikipedia.org/wiki/Exif)를 제거해야하는 웹 사이트를 들자. 이는 FLicker와 같은 웹사이트에서 사용자에게 Exif 메타데이터를 보고 호스팅하는 이미지에 기술적 세부 정보(ex: 색상 깊이, 카메라 제조사 및 모델, 기타 데이터)를 알아볼 수 있는 방법을 제공한다.

이런 작업에서 이미지를 가져오고 ArrayBuffer로 변환하고, Exif 메타데이터를 추출하는 로직은 메인 스레드의 비용이 많이 든다. 이를 Web Worker의 메시징 파이프라인을 사용해서 Exif 메타데이터가 HTML 문자열로 메인 스레드로 다시 전송되어 사용자에게 표시되는 흐름을 알아보자

### Web Worker가 없는 메인 스레드

![](https://web.dev/static/learn/performance/web-worker-demo/image/fig-1.opt_1440.png)

1. 이 양식은 입력을 받아 `fetch` Exif 메타데이터가 포함된 이미지의 초기 부분을 가져오기 위한 요청을 전송한다.
2. 이미지 데이터는 `ArrayBuffer`.
3. 해당 [exif-reader](https://www.npmjs.com/package/exif-reader) 스크립트는 이미지에서 Exif 메타데이터를 추출하는 데 사용된다.
4. 메타데이터를 스크래핑하여 HTML 문자열을 구성한 후, 이를 메타데이터 뷰어에 채운다.

### Web Worker를 사용한 메인 스레드

![](https://web.dev/static/learn/performance/web-worker-demo/image/fig-2.opt_1440.png)

메타데이터 뷰어를 HTML로 채우는 것을 제외한 모든 것이 메인 스레드에서만 했던 일을 Web Worker 스레드에서 수행된다. 이런 방식의 장점으로는 `exif-reader` 스크립트가 다운로드하고, 파싱하고, 컴파일하는 비용이 메인 스레드에서 발생하는게 아니라 Web Worker 스레드로 로드된다는 점이다.

### Web Worker 코드 살펴 보기

```js
// scripts.js
// Exif reader Web Worker 등록
const exifWorker = new Worker("/js/with-worker/exif-worker.js");

// CORS 제한으로 인해 이 프록시를 통해 이미지 요청야 한다.
const imageFetchPrefix = "https://res.cloudinary.com/demo/image/fetch/";

const imageFetchPanel = document.getElementById("image-fetch");
const imageExifDataPanel = document.getElementById("image-exif-data");
const exifDataPanel = document.getElementById("exif-data");
const imageInput = document.getElementById("image-url");

document.getElementById("image-form").addEventListener("submit", (event) => {
  event.preventDefault();

  // Web Worker에 이미지 URL 전송
  exifWorker.postMessage(`${imageFetchPrefix}${imageInput.value}`);
});

// Web Worker에 Exif 메타데이터가 돌아올때까지 기다린다.
exifWorker.addEventListener("message", ({ data }) => {
  // 이렇게하면 Exif 메타데이터 뷰어가 채워진다.
  exifDataPanel.innerHTML = data.message;
  imageFetchPanel.style.display = "none";
  imageExifDataPanel.style.display = "block";
});
```

이 코드는 메인 스레드에서 실행되고, 이미지 URL을 Web Worker에 전송하도록 폼을 설정한다. 거기서 Web Worker 코드는 외부 `exif-reader` 스크립트를 로드하는 [importScripts](https://developer.mozilla.org/docs/Web/API/WorkerGlobalScope/importScripts) 문으로 시작한 다음 메인 스레드로의 메시징 파이프라인을 설정한다.

```js
// exif-worker.js
// exif-reader 스크립트 가져오기
importScripts("/js/with-worker/exifreader.js");

// 메시징 파이프라인을 설정하여 Exif 데이터를 `window`로 전송한다.
self.addEventListener("message", ({ data }) => {
  getExifDataFromImage(data).then((status) => {
    self.postMessage(status);
  });
});
```

> 참고: 외부 스크립트를 Web Worker 범위로 가져오기 위한 `importScripts` 구문은 광범위하게 호환되지만, 대부분의 브라우저에서 `static import` 구문을 사용하여 모듈을 Web Worker로 가져오는 것도 가능하다. 자세한 내용은 [모듈 워커로 웹 스레딩하기](https://web.dev/articles/module-workers)를 참조.

이 Javascript는 메시징 파이프라인을 설정하여 사용자가 URL이 포함된 양식을 JPEG 파일로 제출하면 URL이 Web Worker에 도착하도록 한다. 다음 코드는 JPEG 파일에서 Exif 메타데이터로 추출하고 HTML 문자열을 작성한 다음 해당 HTML을 `window`로 다시 전송하여 사용자에게 표시한다.

```js
// 블롭을 가져와서 이미지 데이터를 'ArrayBuffer'로 변환
// 참고: Promise는 가독성을 위해 단순화되었으며,
// 실패 시 거부 처리는 포함하지 않는다. 전체 Web Worker 코드를 확인.
// https://glitch.com/edit/#!/exif-worker?path=js%2Fwith-worker%2Fexif-worker.js%3A10%3A5
const readBlobAsArrayBuffer = (blob) =>
  new Promise((resolve) => {
    const reader = new FileReader();

    reader.onload = () => {
      resolve(reader.result);
    };

    reader.readAsArrayBuffer(blob);
  });

// Exif 메타데이터를 가져와 마크업 문자열로 변환하여
// DOM의 Exif 메타데이터 뷰어에 표시한다.
const exifToMarkup = (exif) =>
  Object.entries(exif)
    .map(([exifNode, exifData]) => {
      return `
    <details>
      <summary>
        <h2>${exifNode}</h2>
      </summary>
      <p>${
        exifNode === "base64"
          ? `<img src="data:image/jpeg;base64,${exifData}">`
          : typeof exifData.value === "undefined"
          ? exifData
          : exifData.description || exifData.value
      }</p>
    </details>
  `;
    })
    .join("");

// 부분 이미지를 가져오고 해당 Exif 데이터를 가져온다.
const getExifDataFromImage = (imageUrl) =>
  new Promise((resolve) => {
    fetch(imageUrl, {
      headers: {
        // 범위 요청을 사용하여 이미지의 처음 64KB만 다운로드한다.
        // 이렇게 하면 메타데이터만 필요한 경우 대용량 JPEG 파일을 다운로드하여 대역폭이 낭비되는 것을 방지할 수 있다.
        Range: `bytes=0-${2 ** 10 * 64}`,
      },
    })
      .then((response) => {
        if (response.ok) {
          return response.clone().blob();
        }
      })
      .then((responseBlob) => {
        readBlobAsArrayBuffer(responseBlob).then((arrayBuffer) => {
          const tags = ExifReader.load(arrayBuffer, {
            expanded: true,
          });

          resolve({
            status: true,
            message: Object.values(tags)
              .map((tag) => exifToMarkup(tag))
              .join(""),
          });
        });
      });
  });
```

Web Worker는 `fetch` 호출을 격리하고 응답을 처리하거나 메인 스레드를 차단하지 않고 대량의 데이터를 처리하는 등 모든 종류의 작업에 사용할 수 있다. 웹 애플리케이션의 성능을 개선할 때는 Web Worker 환경에서 합리적으로 수행할 수 있는 모든 것을 생각해 봐

### 출처

- [web-worker-demo](https://web.dev/learn/performance/web-worker-demo)
