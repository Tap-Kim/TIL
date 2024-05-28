# TIL 

## 어떤 공부를 했는지 or 문제가 있었는지
  - 테스팅 컴포넌트 시도(with zustand, msw)
  - 알고리즘
  - FE 아티클 번역 리뷰
  - 아티클 정독
    - [[React] 모달 깊게 파헤치기 : 효율적인 모달 관리에 대하여 (useModal custom hooks)](https://cocoon1787.tistory.com/m/875)

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
  - `ReactDOM.createRoot`과 상태관리 라이브러리를 사용하여 `Alert`를 제어하는 방법을 알게됨. 보통 UI Provider에 정속되는 모달은 서버 사이드 에러에 관한 모달 처리가 애매하다.
    이때 `ReactDOM.createRoot`을 사용하면 modal 객체에대한 conatiner 정보를 document 객체를 동적으로 생성해서 관리 할 수 있게된다. 이렇게 modal 객체를 동적으로 관리하게 되면 Next.js와 같이 서버 사이드 렌더링시 모달 제어와 zustand의 set 또는 recoil의 Selector 함수 내부에서 모달을 띄울수 있게 가능해지는 것을 알게되었다.


