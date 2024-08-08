# Redux의 구조와 배경

## 정리

- React에서 상태관리란, 애플리케이션 전체적으로 하나의 `저장소(store)`에서 상태를 관리하여, 전체 컴포넌트에 동일한 결과값 및 제어를 하기 위하여 나타났다. 하지만 상태 변화가 일어남에 따라 즉각적으로 모든 요소들이 변경되어 애플리케이션이 찢어지는 현상(`tearing`_-하나의 상태에 따라 서로 다른 결과물을 사용자에게 보여주는 현상_)을 방지하기위해 오늘날 `상태관리 라이브러리 또는 상태를 주입하여 전달하는 방식`을 `Flux` 패턴을 근간으로하여 제공한다.
- 이번에는 Redux와 Flux 패턴 그리고, 리액트에서 직접 구현해보는 방식을 정리했다.

### 참고

- 주요 키워드: 상태관리, Flux 패턴, Context API, useSyncExternalStore, useReducer, useState
- 관련 기술: React, Redux, Elm

## 무엇을 알았는지

### Flux 패턴의 등장 배경

1. MVC 패턴
   ![](https://miro.medium.com/v2/resize:fit:640/format:webp/1*sz1U9AUxtw5WnVNmS1jgww.png)
   MVC 패턴은 앱의 크기가 커짐에 따라 상태(데이터)도 많은 곳에서 일어나 변화와 추적의 이해가 어려운 양방향 데이터 바인딩으로써 MVC은 React와 맞지 않는 방식이었다.

2. Flux 패턴
   ![](https://www.clariontech.com/hs-fs/hubfs/Image2-61.png?width=513&name=Image2-61.png)
   페이스북 팀에서는 코드 양은 많아지지만 복잡한 시나리오에서도 단반향으로 데이터 흐름을 설계하여 변화와 추적이 용이한 Flux 패턴을 채택하였고, Redux와 같은 상태관리 라이브러리에서도 이 패턴을 채택했다.

### Redux 구조와 흐름

Flux를 구현하기위해, [Elm](https://kyunooh.gitbooks.io/elm/content/architecture/) 아키텍처를 도입했고, 이는 웹페이지를 선언적으로 작성하기위한 언어다.

Redux는 하나의 상태 객체를 `store`에 저장하고, 이 객체를 업데이트하는 작업을 `dispatch`해서 업데이트를 수행한다. 이 작업은 `reducer` 함수로 발생시키는데 이 함수의 실행은 웹 앱 상태에 대한 완전히 `새로운 복사본을 반환`하고 앱이 `새롭게 만들어진 상태를 전파`한다.

![](https://www.clariontech.com/hs-fs/hubfs/Image3-43.png?width=417&name=Image3-43.png)

- `action`: 작업을 처리할 액션과 발생시 함께 포함시킬 데이터를 정의하여 dispatcher로 보낸다.
- `dispatcher`: action을 store에 보내는 역할을 하며, 콜백 함수 형태로 보낸다.
- `store`: 실제 상태에 따른 값과 변경할 수 있는 메서드를 가진다.
- `view`: 컴포넌트에 해당하는 부분으로 렌더링의 역할을하며, view에서도 입력과 같은 행위에 따라 상태를 업데이트가 가능하다.

### 출처

- [모던 리액트 Deep Dive](https://m.yes24.com/Goods/Detail/123161563)
