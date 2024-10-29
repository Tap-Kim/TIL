# [디자인 패턴] Observer

## 개념

옵져버(Observer) 패턴은 특정 구독하는 주체를 Observer라고하고, 구독 가능한 객체를 Observable이라 한다. 이벤트가 발생할때 마다 Observable은 모든 Observer에 이벤트를 전파한다.

Observable 객체는 보통 3가지 주요 특징을 포함한다.

- `observers` : 이벤트가 발생할때마다 전파할 Observer들의 배열
- `subscribe()` : Observer를 Observer 배열에 추가한다
- `unsubscribe()` : Observer 배열에서 Observer를 제거한다
- `notify()` : 등록된 모든 Observer들에게 이벤트를 전파한다

```js
class Observable {
  constructor() {
    this.observers = [];
  }

  subscribe(func) {
    this.observers.push(func);
  }

  unsubscribe(func) {
    this.observers = this.observers.filter((observer) => observer !== func);
  }

  notify(data) {
    this.observers.forEach((observer) => observer(data));
  }
}
```

`subscribe` 메서드를 통해 `Observer`를 등록하고 반대로 `unsubscribe`를 통해 등록 해지할 수 있다. 그리고 `notify` 메서드를 통해 모든 `Observer`에게 이벤트를 전파할 수 있다.

[![image](https://res.cloudinary.com/ddxwdqwkr/video/upload/f_auto/v1609056518/patterns.dev/jspat-41_nxsnbd.jpg)](https://res.cloudinary.com/ddxwdqwkr/video/upload/f_auto/v1609056518/patterns.dev/jspat-41_nxsnbd.mp4)

```jsx
import React from "react";
import { Button, Switch, FormControlLabel } from "@material-ui/core";
import { ToastContainer, toast } from "react-toastify";
import observable from "./Observable";

function handleClick() {
  observable.notify("User clicked button!");
}

function handleToggle() {
  observable.notify("User toggled switch!");
}

function logger(data) {
  console.log(`${Date.now()} ${data}`);
}

function toastify(data) {
  toast(data, {
    position: toast.POSITION.BOTTOM_RIGHT,
    closeButton: false,
    autoClose: 2000,
  });
}

observable.subscribe(logger);
observable.subscribe(toastify);

export default function App() {
  return (
    <div className="App">
      <Button variant="contained" onClick={handleClick}>
        Click me!
      </Button>
      <FormControlLabel
        control={<Switch name="" onChange={handleToggle} />}
        label="Toggle me!"
      />
      <ToastContainer />
    </div>
  );
}
```

Observer 패턴은 다양하게 활용이 가능하지만 비동기 호출이나 이벤트 기반 데이터를 처리할 때 매우 유용하다. 어떤 컴포넌트가 특정 데이터의 다운로드 완료 알림을 받기 원하거나, 사용자가 메시지 보드에 새로운 메시지를 게시했을 대 모든 멤버가 하는 등의 상황이다.

## 사례 분석

RxJS는 Observer 패턴을 구현한 유명한 라이브러리다.

> "ReactiveX는 Observer 패턴, 이터레이터 패턴, 함수영 프로그래밍을 조합하여 이벤트 순서를 이상적으로 관리할 수 있다. "

RxJS를 사용하면 Observable과 Observer(Subscriber)를 만들어낼 수 있다.

```jsx
import React from "react";
import ReactDOM from "react-dom";
import { fromEvent, merge } from "rxjs";
import { sample, mapTo } from "rxjs/operators";

import "./styles.css";

merge(
  fromEvent(document, "mousedown").pipe(mapTo(false)),
  fromEvent(document, "mousemove").pipe(mapTo(true))
)
  .pipe(sample(fromEvent(document, "mouseup")))
  .subscribe((isDragging) => {
    console.log("Were you dragging?", isDragging);
  });

ReactDOM.render(
  <div className="App">Click or drag anywhere and check the console!</div>,
  document.getElementById("root")
);
```

## 장점

Observer 패턴을 사용하는 것도 관심사의 분리와 단일 책임의 원칙을 강제하기 위한 좋은 방법이다. Observer 객체는 Observable 객체와 강결합되어있지 않고 언제든 분리가 가능하다. Observable 객체는 이벤트 모니터링의 역할을 갖고, Observer는 받은 데이터를 처리하는 역할을 갖는다.

## 단점

Observer가 복잡해지면 모든 Observer들에 알림을 전파하는 데 성능 이슈가 발생할 수 있다.

## 출처

- [Observer 패턴](https://patterns-dev-kr.github.io/design-patterns/observer-pattern/)
