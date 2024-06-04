# TIL 

## 어떤 공부를 했는지 or 문제가 있었는지
- [아티클 번역](https://github.com/Korean-FE-Article/korean-fe-article/pull/424)
- Input 영역의 초기 값 제어를 위해 비동기 정보를 useEffect 없이 제어하는 방법을 고민함

## 내가 시도해본 것들
- Conext API를 활용해서 Input 영역의 비동기 제어를 시도

## 어떻게 해결했는지
- useEffect 사용 X, Context API 구성하여 useInput 훅은 Context에 관리. 이렇게 되면 실제 비동기 컴포넌트를 제어하기 위한 Input 컴포넌트를 별도로 작성해서 Suspense를 활성화하여 비동기 제어가 가능해진다.

## 무엇을 새롭게 알았는지
-
