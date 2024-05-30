# TIL 

## 어떤 공부를 했는지 or 문제가 있었는지
- javascript 우선순위 큐(힙)을 공부함 - [우선순위 큐](https://github.com/Tap-Kim/algorithm-history/blob/main/data-structure/PriorityQueue.js)
- 프로그래머스 - [더맵게](https://github.com/Tap-Kim/algorithm-history/tree/main/programmers)
- 아티클: fe-article 리뷰 겸 리액트 19 컴파일러 문서 정독

## 내가 시도해본 것들
- [JavaScript로 Heap | Priority Queue 구현하기
](https://jun-choi-4928.medium.com/javascript%EB%A1%9C-heap-priority-queue-%EA%B5%AC%ED%98%84%ED%95%98%EA%B8%B0-8bc13bf095d9)
- [[JS] 힙/우선 순위 큐 새로 짜봤습니다.](https://dev-russel.tistory.com/55) 
  > 힙 구현시 pop에서 최상단 노드부터 왼쪽 노드, 오른쪽 노드를 비교해줘야하는데, 이 과정에서 undefined와 관련해서 많은 오류가 있는 걸 확인했습니다. 새로 짠 코드를 공유함

## 어떻게 해결했는지

## 무엇을 새롭게 알았는지
- 최대 최소를 구하는 작업중 배열로만 비교하게 되면 O(n)의 선형 그래프가 형성되어 숫자가 커지거나 비교 대상이 많아 질수록 효율성 저해가 된다. 이때 우선순위 큐를 활용하면 O(logn)의 자료 구조를 갖게 되어 효율성을 가져갈 수 있다.
