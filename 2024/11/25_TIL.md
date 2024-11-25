# Shadow DOM 알아보기

Shadow DOM은 웹 컴포넌트 구성하기위해 마크업, 스타일, 동작을 HTML 태그로 숨겨서 캡슐화가 가능한 영역이다. Shadow DOM으로 선언한 영역은 다른 코드에서 부터 분리되어 충돌하지 않고 코드를 간결하게 유지할 수 있는 영역이다.

## 구조

Shadow DOM은 숨겨진 DOM 트리가 통상적인 DOM 트리에 속한 요소에 attach가 가능하다. Shadow DOM 트리는 shadow root로 부터 시작되어서 원하는 요소 안에 attach 될 수 있다.

![](https://developer.mozilla.org/ko/docs/Web/API/Web_components/Using_shadow_DOM/shadowdom.svg)

> Flattened Tree (for rendering): (렌더링을 위해) 평평해진 트리

알아야 할 조금의 shadow DOM 용어가 있다.

- **Shadow host**: shadow DOM이 attach되는 통상적인 DOM 노드.
- **Shadow tree**: shadow DOM 내부의 DOM 트리.
- **Shadow boundary**: shadow DOM이 끝나고, 통상적인 DOM이 시작되는 장소.
- **Shadow root**: shadow 트리의 root 노드.

Shadow DOM은 새로운 방식이 아니라, 기존 DOM 노드를 생성하고 children을 append하거나 요소 설정, 스타일을 설정하는 방식과 동일하게 구성된다. 다만 Shadow DOM 내부 코드 중 어느 것도 Shadow DOM 외부의 모든 영역에 영향을 주지 않다는 점이고, 이것이 캡슐화가 가능하다는 것을 증며한다.

## 내장된 Shadow DOM

브라우저 컨트롤의 복잡한 방식은 어떻게 만들어지고 스타일이 지정될까?

```html
<input type="range" />;
```

<input type="range" />

---

브라우저는 DOM/CSS를 내부적으로 그린다. 구조 자체는 숨겨져 있지만 개발자 도구에서 Dev Tools > "Show user agent shadow DOM" 옵션을 활성화하면 숨겨졌던 Shadow DOM의 구조가 보인다.

![](https://ko.javascript.info/article/shadow-dom/shadow-dom-range.png)

`#shadow-root`아래에 보이는 것이 Shadow DOM이다.

이건 캡슙활가 강력하게 설정되어 있어 외부에 영향을 주지 않는다고했다. 그렇다면 영향을 주게 하려면 어떻게 해야할까? `pseudo` 속성을 보게 되는데 이 속성을 기준으로 CSS 하위 요소의 스타일링이 가능하다.

```html
<style>
  /* make the slider track red */
  input::-webkit-slider-runnable-track {
    background: red;
  }
</style>
<input type="range" />
```

## shadow Tree

DOM 요소는 두 가지 유형읠 DOM 하위 트리를 가진다

1. Light Tree - HTML 자식으로 구성된 일반 DOM 서브트리. 이전 장에서 본 모든 서브트리는 "light"
2. shadow Tree - HTML에 반영되지 않고 엿보는 눈으로부터 숨겨진 DOM 서브트리

요소에 둘 다 있는 경우에는 브라우저는 shadow Tree만 렌더링 한다. 하지만 shadow와 Light 트리 사이에 구성을 설정할 수 있따.

예를 들어 `<shadow-hello>` 요소는 shadow Tree에서 내부 DOM을 숨긴다.

```html
<script>
  customElements.define(
    "show-hello",
    class extends HTMLElement {
      connectedCallback() {
        const shadow = this.attachShadow({ mode: "open" });
        shadow.innerHTML = `<p>
      Hello, ${this.getAttribute("name")}
    </p>`;
      }
    }
  );
</script>

<show-hello name="John"></show-hello>
```

![](https://ko.javascript.info/article/shadow-dom/shadow-dom-say-hello.png)

두가지 제한 사항이 있다.

1. 요소당 하나의 shadow root만 만들 수 있다.
2. 사용자 정의 요소이거나 "article", "aside", "blockquote", "body", "div", "footer", "h1…h6", "header", "main", "nav", "p", "section" 또는 "span", `<img>` 중 하나여야 한다. `elem`와 같은 요소는 shadow tree를 호스팅할 수 없다.

이 `mode`옵션은 캡슐화 수준을 설정한다. 다음 두 값 중 하나를 가져야한다.

- `"open"`: shadow root는 `elem.shadowRoot`로 사용 가능하다.
  모든 코드는 `elem` shadow tree에 접근할 수 있다.
- `"closed"`: `elem.shadowRoot`은 항상 `null`.

Shadow DOM은 `attachShadow`가 반환한 참조로만 접근할 수 있다(아마도 클래스 내부에 숨겨져 있을 것). `<input type="range">`와 같은 브라우저 네이티브 shadow tree는 닫혀 있다. 액세스할 수 있는 방법이 없다.

## 캡슐화

1. Shadow DOM 요소는 Light DOM에서 `querySelector`에 표시되지 않는다. 특히 Shadow DOM 요소에는 Light DOM의 요소와 충돌하는 ID가 있을 수 있다. shadow 트리 내에서만 고유해야한다.
2. Shadow DOM에는 자체 스타일시트가 있다. 외부 DOM의 스타일 규칙은 적용되지 않다.

```html
<style>
  /* document style won't apply to the shadow tree inside #elem (1) */
  p {
    color: red;
  }
</style>

<div id="elem"></div>

<script>
  elem.attachShadow({ mode: "open" });
  // shadow tree has its own style (2)
  elem.shadowRoot.innerHTML = `
    <style> p { font-weight: bold; } </style>
    <p>Hello, John!</p>
  `;

  // <p> is only visible from queries inside the shadow tree (3)
  alert(document.querySelectorAll("p").length); // 0
  alert(elem.shadowRoot.querySelectorAll("p").length); // 1
</script>
```

1. 문서의 스타일은 shadow tree에 영향을 주지 않다.
2. 하지만 내부의 스타일은 작동한다.
3. shadow tree의 요소를 가져오려면 트리 내부에서 쿼리해야한다.

그렇지 않으면 위에 정의했던, `pseudo` 속성을 직접 접근하여 브라우저마다 제공되는 속성상태 별로 스타일을 지정해주면 된다.

- `-webkit-`: 크롬, 오페라, 사파리, Edge
- `-moz-`: 파이어폭스는
- `-ms-`: 익스플로러는

## 출처

- [shadow DOM 사용하기](https://developer.mozilla.org/ko/docs/Web/API/Web_components/Using_shadow_DOM)
- [Shadow DOM](https://ko.javascript.info/shadow-dom)
- [딥다크한 어둠의 공간 Shadow DOM](https://codingapple.com/unit/html-31-shadow-dom/)
