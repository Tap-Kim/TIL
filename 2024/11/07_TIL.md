# SVG 사용 이유와 viewBox 활용

SVG를 사용하는 이유는 아래와 같다.

1. 안깨지고
2. 용량이 적고
3. 출력이 빠르고
4. 수정과 애니메이션이 가능

## SVG 란?

SVG는 Scalable Vector Graphics의 약어로 각 위치 값을 표시하는 벡터와 같은 방식의 2차원 그래픽용 XML 기반 형식이다. 어떤 디바이스에서도 깨지지 않고 마크업 언어로 이루어져 css와 javascript로 수정이 가능하다. 우리가 보는 웹에서 보는 움직이는 텍스트와 아이콘은 모두 SVG이다.

![](https://img1.daumcdn.net/thumb/R1280x0/?fname=http://t1.daumcdn.net/brunch/service/user/7J2z/image/9UfhOlsrIZ7dAGtFz5V25i-quBc.jpg)

위 사진은 48x48 px의 PNG와 SVG 아이콘의 선명도 차이다. 왜 차이가 날까? PNG나 JPG는 비트맵 기반의 이미지로 각 항목에 하나 이상의 정보 비트를 가지고 있는 표현이다. 반면 SVG는 벡터 기반으로 각 좌표에 점을 이어 만들어짐 개념인데 벡터 기반의 아이콘은 확대나 축소를 해도 깨지지 않는 것과 같은 원리다.

## PNG의 출력 높이면 안되나?

선명하게 하려면 PNG에 더 많은 데이터를 추가하면 되나 웹 성능이 고려되지 않게 된다. 고해상도의 PNG는 일반적으로 HDPI 디스플레이에 렌더링이 될 때 크기가 매우 크다. 보통 배경이 없는 이미지를 사용하기 위해 PNG를 사용해왔고 때문에 상대적으로 용량이 큰 PNG를 로드하는데 시간이 오래 걸리기 때문에 브라우저 로딩 속도가 느려진다.

SVG는 코드로 이루어져 있어 바이트도 안 되는 크기로 용량 적인 측면에서 웹 사이트 로딩 속도를 빠르게 만들어 준다.

## 사이트 로딩의 출력 차이

브라우저 로딩시 서버에 HTTP 요청을 해야한다. 이미지가 많을술고 더 많은 HTTP 요청이 이루어져 사이트 속도가 느려지게 된다.

SVG는 크기가 작은 뿐 아니라 HTML의 인라인에 포함되어 HTTP 요청을 제거할 수 있어 유연성이 빨라진다. sprite 방식과 같이 PNG를 활용할때는 아이콘을 하나의 파일에 합쳐 위치 값으로 나머지를 숨기고 표시해서 아이콘을 안보이게 하는 것도 로딩 속도와 용량 때문이다.

아래는 아이콘을 합치게 보이는 간단한 코드다.

```html
<div class="icons-home"></div>
<div class="icons-shop"></div>
<div class="icons-cart"></div>
```

```css
.icons-home,
. icons-shop,
. icons-cart {
  background-image: url("@link");
  background-repeat: no-repeat;
  height: 64px;
}
.icons-home {
  width: 60px;
  background-position: 0 0;
}
.icons-shop {
  width: 60px;
  background-position: -60px 0;
}
.icons-cart {
  width: 60px;
  background-position: -120px 0;
}
```

위 에는 3개의 아이콘이 존재한다. 위 처럼 모든 아이콘을 하나로 모아 하나의 PNG 파일로 만들어 위치 값(`background-position`)으로 아이코을 보이게 한다. 이렇게 디자이너는 용량을 줄이고 로딩 속도는 빠르게하기 위해 노력했다. 때문에 더 용량이 적고 빠른 SVG를 사용하게 되었다.

## 애니메이션

SVG를 사용하게되면 XML언어로 이루어져있기 때문에 css와 javascript로 수정이 가능하다.
SVG를 활용한 애니메이션은 Jake Archibald가 2013년에 이 기술을 개척했다. 원리는 SVG에는 탐색이 가능한 DOM이 있어 기존의 웹 기반의 DOM과 동일한 개념이 적용된다. 그 다음 SVG의 다른 부분에 요소를 첨부해서 애니메이션을 적용하는 방식이다.

1. 일단 원하는 텍스트를 일러스트에서 작성해줍니다.
   ![](https://img1.daumcdn.net/thumb/R1280x0/?fname=http://t1.daumcdn.net/brunch/service/user/7J2z/image/PzX7ASl66TA9Lo0tkuSKJC-XcgA.png)

2) 그다음 해당 텍스트를 SVG소스로 Save as 합니다. 그러면 아래와 같은 텍스트의 SVG 코드를 얻을 수 있습니다.
   ![](https://img1.daumcdn.net/thumb/R1280x0/?fname=http://t1.daumcdn.net/brunch/service/user/7J2z/image/oB_3O1GvWVr2qoVkRtPssK5DfEw.png)

3) 이제 필요 없는 소스들은 날리고 HTML로 가져와줍니다.
   ![](https://img1.daumcdn.net/thumb/R1280x0/?fname=http://t1.daumcdn.net/brunch/service/user/7J2z/image/oFyWABS1kq90Ikt-njZyM1zWPLg.png)

4) 그다음 해당 클래스 값에 keyframes animate CSS효과를 적용해줍니다.
   ![](https://img1.daumcdn.net/thumb/R1280x0/?fname=http://t1.daumcdn.net/brunch/service/user/7J2z/image/po4dAHVKuz-wbO191DKo9Ih1YEw.png)

5) 그러면 하단의 화면과 같은 SVG효과를 얻을 수 있습니다.
[영상 링크 주소](https://play-tv.kakao.com/embed/player/cliplink/v8c5b5K88Qf8m6fssfWQzcV@my?service=daum_brunch&section=article&showcover=1&showinfo=0&extensions=0&rel=0)
<iframe width="700.000000" height="394.000000" src="//play-tv.kakao.com/embed/player/cliplink/v8c5b5K88Qf8m6fssfWQzcV@my?service=daum_brunch&amp;section=article&amp;showcover=1&amp;showinfo=0&amp;extensions=0&amp;rel=0" frameborder="0" allowfullscreen=""></iframe>

이렇게 하면 SVG를 활용해서 애니메이션을 활용할 수 있게 된다.

## 아이콘을 SVG로 활용하자

SVG는 아이콘, 로고, 일러스트레이션, 차트, 애니메이션에 적합하고 PNG나 JPEG는 사진에 적합하다.

SVG는 크로스 브라우저 호환성과 관련해서 IE8 이하 버전을 제외한 모든 브라우저에서 제공하나 옛날 브라우저 버전은 호환이 안되기도 한다. HTML에서 SVG 효과를 활용하는 경우 철저한 크로스 브라우저 테스트를 수행해야 한다.

또한 어느 정도 지식이 있어야한다. Sketch와 Figma의 SVG Export 값이 달라서 섞어서 사용하는 편이 좋다. 또 Sketch에서 아이콘 생성시 `evenodd`와 `nonzero`의 규칙도 알아야한다. 이말은 스케이의 채우기 규칙이 짝수로 시작하는 것과 0으로 시작하는 것의 차이이다.

## viewBox란

viewBox는 svg 요소가 그려지는 영역에서 svg 요소의 크기를 확대/축소, 위치 조정이 가능한 속성이다. 이는 화면 크기가 달라져도 svg 요소의 크기는 고정되어 있어 반응형에 좋은 옵션이다. 즉, viewBox 속성을 사용하면 화면 크기에 따라 svg 요소의 크기가 자동으로 조절된다.

## viewBox의 속성 값

`viewBox="<min-x> <min-y> <width> <height>"`

![](https://user-images.githubusercontent.com/40762111/138582965-1d57b799-4cee-42fa-83dc-fa9e38aa2564.png)

위 그림처럼 `min-x`와 `min-y`는 svg가 그려지는 영역의 `시작점`, `왼쪽 상단의 꼭짓점`으로, width와 height는 각각 영역의 가로, 세로 길이로 볼 수 있다.

여기서 `주의할 점은 width와 height가 우리가 흔히 생각하는 px 단위가 아니라는 것`이다. viewBox의 의미에서 말했던 것처럼 svg 요소들의 위치를 조정하거나, 요소들의 확대와 축소를 위한 일종의 좌표 평면이다. 그럼 예시를 통해서 좀 더 자세히 알아보자.

### 예시

svg에 `circle`요소를 추가하여 컴포넌트 형태로 만들었다.

```jsx
const App = () => (
  <div style={{ marginTop: "50px", marginLeft: "50px" }}>
    <div style={{ backgroundColor: "yellow", width: "300px", height: "300px" }}>
      <svg viewBox="0 0 200 200">
        <circle r="100" fill="blue" />
      </svg>
    </div>
  </div>
);
export default App;
```

![](https://user-images.githubusercontent.com/40762111/138591296-1cca74c3-136b-4f64-b6db-0cf36cedd3b0.png)

`viewport`에서 svg는 width와 height가 각각 300px인 영역 안에 위치하고 있다.

하지만 svg의 viewBox는 "0 0 200 200"이므로, viewport의 width, height 300px의 길이는 svg viewBox 좌표평면 기준으로 200의 길이가 된다.

svg의 요소 중에서 circle의 좌표 또한 svg의 viewBox의 좌표를 따르게 된다. circle 요소의 중심 좌표 속성 cx와 cy가 설정되지 않은 상태이므로 현재 cx="0", cy="0"이 적용된 상태이면서, 반지름 r=100의 circle 요소가 만들어져 있는 것을 확인할 수 있다.

여기서 특징은 원 중심을 기준으로 제1, 3, 4 사분면(보라색 부분)은 svg viewBox 영역을 벗어났기 때문에 viewport에서 나타나지 않는다는 점이다.

### 예시 - 위치조정

일단 `circle` 요소에 `cx="100", cy="100"`을 추가했다.

```jsx
const App = () => (
  <div style={{ marginTop: "50px", marginLeft: "50px" }}>
    <div style={{ backgroundColor: "yellow", width: "300px", height: "300px" }}>
      <svg viewBox="0 0 200 200">
        <circle cx="100" cy="100" r="100" fill="blue" />
      </svg>
    </div>
  </div>
);

export default App;
```

![](https://user-images.githubusercontent.com/40762111/138591304-625a271a-d0b2-4b5e-813b-750dee8f68c0.png)

현재 상태에서 viewBox를 이용해서 어떻게 기본 예시와 같은 모양을 만들 수 있을까. 바로 `min-x와 min-y`를 변경하면 된다.

![](https://user-images.githubusercontent.com/40762111/138591309-32277e0e-626e-45f8-8117-a22f2f36825c.png)

viewBox의 min-x, min-y를 변경하면 viewport에서 보이는 영역(현재 예시에서의 width, height 300px 영역)이 동일하지만, svg에서 보여주는 영역은 변경된다.

viewBox가 "0 0 200 200"부터 "1 1 200 200", "2 2 200 200" 이런 식으로 "100 100 200 200"까지 변경되면, 아래 영상과 같이 된다. 영상은 위의 왼쪽 사진의 빨간 네모가 오른쪽 사진의 빨간 네모 위치로 이동한다고 생각하면 된다. 그런데, 빨간 네모는 실제로 화면에서 보이는 부분이므로 정지되어 있다.

즉, 빨간 네모 화면이 동남쪽으로 이동하는 것은 파란 원이 북서쪽으로 이동하는 것처럼 보이므로 영상과 같이 나타난다.

### 예시 - 확대/축소

viewBox의 width와 height를 변경하면 svg 요소들을 확대하거나 축소할 수 있다.

```jsx
const App = () => (
  <div style={{ marginTop: "50px", marginLeft: "50px" }}>
    <div style={{ backgroundColor: "yellow", width: "300px", height: "300px" }}>
      // 이 부분만 변경 viewBox="0 0 100 100" / viewBox="0 0 200 200" /
      viewBox="0 0 300 300"
      <svg viewBox="0 0 100 100">
        <circle cx="100" cy="100" r="100" fill="blue" />
      </svg>
    </div>
  </div>
);

export default App;
```

![](https://user-images.githubusercontent.com/40762111/138591367-31e26466-f18d-4158-91b2-8033b77fa7bd.png)

viewBox의 속성값 중 width와 height를 각각 100, 200, 300으로 변경했다. 이 값을 변경하더라도 viewport에서 보이는 영역(width, height 300px)은 동일하지만, svg 영역 안에서의 요소를 확대하거나 축소할 수 있다.

# 출처

- ['SVG'를 사용하는 이유!](https://brunch.co.kr/@ggk234/11)
- [SVG viewBox를 알아보자](https://tecoble.techcourse.co.kr/post/2021-10-24-svg-viewBox/)
