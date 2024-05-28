# TIL 

## 어떤 공부를 했는지 or 문제가 있었는지
  - 테스팅 컴포넌트 시도(with zustand, msw)
  - 알고리즘
  - FE 아티클 번역 리뷰

## 내가 시도해본 것들
  - 테스트
    - 업무 중에 작성중에 있는 테스트 중 zustand와 msw가 같이 맞물리는 테스트 작성
  - 알고리즘

## 어떻게 해결했는지
  - 테스트
    - [zustand with Jest](https://docs.pmnd.rs/zustand/guides/testing#jest)에서 기본 mocking set을 제공하여 셋팅해봄 
  - 알고리즘

## 무엇을 새롭게 알았는지
  - 테스트
    - 확실히 `Provider`의 영향 없이 zustand 쪽에서의 로직 테스트가 수월했고, 공식 문서에서 제공해주는 set을 그대로 활용하니 문제 없이 테스트 작성이 가능하다는 것을 알게됨
    - jest 상에 `document.cookie`의 셋팅이 유틸함수로 작성한 `document.cookie`을 활용할 필요없이 필요한 테스트 구문에서 `beforeEach`에서 `document.cookie = 'accessToken=testAccessToken';`을 선언해서 명확한 테스트가 가능하다는 것을 알게됨.
  - 알고리즘
