# 비동기 처리 방법(with. 리액티브 프로그래밍, RxJS)

## 정리
- 자바스크립트에서 전반적으로 패칭, 이벤트, 웹 API 관련된 기능 등 기본적으로 비동기 API를 다뤄야한다.
- 하지만 비동기를 다루는 문제가 어려운데, 여러가지 방법중 반응형과 함수형 프로그래밍에 결합된 Observable과 RxJS에 대해 공부해보았다.
- 데이터 변화에 민감하고 실시간 데이터 처리, 네트워크, 콜백 지옥 등 개선하며, 명령형 
    -> 선언형 프로그래밍이 가능해지면서, 가독성과 유지보수성이 뛰어난 처리
- 자바스크립트에선 RxJS 라이브러리와 Observable(Web API)이 유일핳게 해당 패러다임을 가져가는 기술이다.
- 하지만 타 언어에 비해 자바스크립트에선 Promise가 나오게 되면서 표준으로 자리 잡지 못하고 있는 상황이며 약간의 러닝 커브가 존재한다.

### 참고 
- 주요 키워드: Observer 패턴, Iterator 패턴, 함수형 프로그래밍, 리액티브 프로그래밍
- 관련 기술: RxJS, Observable, ReactiveX

## 무엇을 알았는지
### Observable
![](https://yozm.wishket.com/media/news/1753/image005.png)
![](https://yozm.wishket.com/media/news/1753/image007.png)
- pull / push 베이스 통신
    1. Pull 
        - 외부에 명령하여 응답받고 처리
        - 값을 가지고 오기 위해서는 계속 호출
        - iteration
        - 예) I/O, Storage, File 등
    2. Push
        - 외부에 응답이 오면 그때 반응하여 처리
        - 갑을 가지고 오기 위해서 구독
        - observation
        - 예) DOM 이벤트, 웹 소켓, 애니메이션, 워커, 시간(쓰로틀링, 디바운싱) 등
- 외부 통신을 위한 데이터를 pull 베이스가 아닌 push 베이스 방식의 비동기 처리에 적합한 모델
    - 웹의 비동기 대부분은 push 베이스 처리
    - 비동기 흐름을 선언적으로 작성이 가능 Observable composition
    - 취소가 가능하다.
    - 단건으로 밖에 처리하지 못하는 Promise에 반해 여러 데이터를 비동기로 처리가 가능하다.

### RxJS
- 관찰 가능한 시퀀스를 사용하여 비동기 및 이벤트 기반 프로그래밍
- RxJS는 Array와 Promise의 성질을 모두 가진 이벤트를 다루는 새로운 객체 타입인 Observable를 제공하는 라이브러리
- 핵심 유형인 Observable(`Array`), 위성 유형(`Observer`, `Scheduler`, `Subjects`) 및 메소드(`map, filter, reduce` 등)에서 영감을 받은 연산자 `every`를 제공하여 비동기 이벤트를 컬렉션 처리가 가능.
- ReactiveX는 Observer 패턴 + Iterator 패턴 + 함수형 프로그래밍을 컬렉션과 결합하여 이벤트 시퀀스를 관리하는 도구
    > 이벤트가 비동기, 시간을 마지 `Array` 처럼 다룰 수 있게 만들어 주는 라이브러리
- Value와 Array와 Promise, 그리고 Observable 모두 (비)동기 컬렉션, 즉 스트림으로 추상화하여 생각할 수 있습니다. Value는 그냥 값이고, Array는 동기 컬렉션, Promise는 비동기 값, Observable은 비동기 컬렉션으로 모든 것은 스트림으로 되어 있으며, 
프로그램은 결국 스트림을 다루는 것으로 귀결되는 심플한 반응형 프로그르매이 패러다임 
- 비동기 처리의 문제점
    - `~하면 ~이라면 ~하면` 이라는 구현방식이 많아 지는데 이때, Promise나 명령형으로 프로그래밍을 작성하게 되면 가독성과 유지보수성이 떨어지 게된다.
    - 이때 RxJS와 같은 도구를 사요하게 되면 `비동기 로직도 데이터로 취급해 함수형 코드`로 다뤄지게되어 코드가 간결해진다.
    - Array 형태이며 데이터를 메서드 체이닝이 아닌 내부에 `오퍼레이터`들이 존재하여 적절히 조합하여 `선언적`으로 작성이 가능해진다.
    - `pipe`라는 개념이 RxJS5.5버전 부터 도입되어 기존에 메서드 체이닝과 같은 문법의 단점이 Tree-shaking에 불리한 점을 개선할 수 있도록 적용되어졌다.(가독성이 떨어지며, 타입스크립트와의 호환성도 떨어지긴함)

- RxJS를 사용하면?
    - 비동기 로직을 `Data`로 다룰 수 있게 된다.
    - 모든 코딩을 스트림과 함수형으로 `쉽게 작성`할 수 있게 된다.
    - Pull 패러다임에서 `Push 패러다임으로 코딩`할 수 있게 된다.

- Oberservavle과 Pipeline을 아직도 논쟁 중
    - 자바스크립트가 싱글 스레드 이벤트 루프 방식으 택하면서 필연적으로 비동기 처리를 콜백으로 해야하기 때문에 콜백 지옥이 되었고, 예외 처리까지 더해지면서 복잡간 코드가 작성이 되었따.
    - Promise A+라는 라이브러리 만들어지면서 비동기를 다루는 방식이 쉬워져 Promise가 표준 API로 자리 잡게 되며 , `async/await`문법이 생겼다.
    - 함수형 프로그래밍을 기반으로 둔 Observable과 pipeline operator도 TC39 표쥰 라이브러리에 제안되어있지만 9년이 지났음에도 아직도 논의 중인 점이 아쉬운 부분.

### ReactiveX
![](https://reactivex.io/assets/operators/legend.png)

### 출처
- [리액티브 프로그래밍의 이해와 적용](https://f-lab.kr/insight/understanding-reactive-programming)
- [ECMAScript Observable](https://github.com/tc39/proposal-observable)
- [ReactiveX - Observable](https://reactivex.io/documentation/ko/observable.html)
- [RxJs 한번 배워보실래요?](https://yozm.wishket.com/magazine/detail/1753/)
- [[Track 2-2] 나석주 - 비동기를 우아하게 처리하기 위한 Observable](https://youtu.be/oHF8PEkteq0?si=S25kKohDjbhfyVFY)