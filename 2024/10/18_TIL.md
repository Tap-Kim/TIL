# Vite의 태생과 구조 (feat. ESBuild, Rollup)

## Vite 태생

기존에 브라우저는 ESM 지원이 안되었다. 즉 스크립트 간 모듈을 보낼수 없었는데. 공통으로 사용되는 모듈 관리를위한 코드 관리가 어려워지게되었따.

html 파일에 script 파일을 불러와 사용시 전역 공간에 변수가 노출될 수 있고, 여러 script를 불러올시 변수 이름이 겹치는 문제도 발생하였다. 이때 다양한 전략으로 파일을 분리하고, window로 전역 객체를 활용하여 스크립트 파일간 모듈을 공유했다. 소스 모듈을 브라우저에서 실행할 수 있는 파일로 크롤링, 처리 및 연결하는 번들링을 해결했다.

- 크롤링 Crawling

  - 웹사이트에서 데이터를 수집하는 과정
  - HTML, CSS, JavaScript 등의 내용을 분석하여 원한느 정보를 추출하는 작업

- 번들링 Bundling
  - 모듈화된 소스 코드를 브라우저에서 실행할 수 있는 파일로 한데 묶어주는 작업

## ESBuild

ESBuild의 주목표는 현재 웹을 위한 빌드 도구의 100배빠른 번들러이며, 사용하기 쉬운 번들러를 만드는 것이다.

![](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FcoO9B0%2FbtsIZxO1v8U%2FCR0dq5zQ9WwuBVV3FTklJk%2Fimg.png)

- Go로 작성되어 네이티브 코드로 컴파일 된다.(parcel, rollup, webpack은 js 기반으로 번들링한다)
- 병렬성이 사용된다.
- TS 컴파일러조차 파서로 사용하지 않고 모든 것을 직접 작성했다.
- 메모리가 효율적으로 사용된다.

## ESBuild로 파일 통합 + Rollup으로 번들링 = Vite

Vite는 번들링시 Esbuild를 사용하지 않은 이유는, 빠르지만 유연한 Rollup의 유연한 플러그인 API와 인프라를 적극 채택했고, 더 나은 선택이었따. 또한 Rollup은 [v4에서 파서를 SWC 전환](https://github.com/rollup/rollup/pull/5073) 했기 때문에 성능 개선에 농력을 기여했다. 또한 Rollup은 Rust 포팅인 Rolldown이 진행 중이며, 이게 진행되면 Rollup과 ESBuild를 모두 대체하는 큰 성능의 번들링 대체제와 빌드 사이의 불일치를 제거를 기대하기 때문이다.

서버 구동시 번들러 기반 도구의 경우 앱 내 모든 코드에 대해 크롤링, 빌드 작업을 전부 마쳐야만 실제 페이지를 제공하는데, Vite는 앱의 모듈을 dependencies와 source code로 나뉘어 개발 서버 시작 시간을 개선했다.

1. **dependencies**: 개발시 내용이 바뀌지 않을 Plain JS Code, 사전 번들링으로 ESBuild 사용
2. **source code**: JSX,CSS 또는 Vue/Svelte 컴포넌트 처럼 수정이 찾은 Non-plain Code, Native ES Build 사용과 현재 화면에서 실제로 사용되는 경우에만 처리되도록 함

![](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FciphmD%2FbtsI1upjhzp%2FUdVzWRXadgyNkvpfBw0lLK%2Fimg.png)
[번들링된 파일 기반으로 작동하는 도구들의 작동방식 - Webpack]

![](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FbUBCvZ%2FbtsIZ55C8SB%2F9m2DkZZPaUzi3mjkA6lHo1%2Fimg.png)
[ES Module 기반으로 작동하는 작동방식 - Vite]

## 정리

최종적으로 Vite는 사전 번들링으로 빠른 ESBuild를 사용하고, ES Module 기반으로 작동하여 모듈 수정시 수정된 부분만 교체한다. 또한 배포시 Rollup을 사용하여 유연성에 최적화된 번들링을 진행한다.

## 출처

- [Why Vite? JS 모듈화의 역사, CJS, ESM, Webpack](https://nami-socket.tistory.com/48)
