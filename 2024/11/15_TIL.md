# 동시성, 병렬성과 자바스크립트의 이해

## 순차, 동시 및 병렬 처리

순차적인 실행은 기본적으로 작업이 겹치지 않고 차례대로 수행되는 것을 말한다. 이 접근 방식의 문제점은, 예를 들어 휴대폰으로 친구에게 무언가를 물어볼 때 친구가 대답이 돌아올 때까지 다른 작업으로 전환하지 않으면 시간을 낭비하게 된다는 것이다. 따라서 때때로 `다양한 형태의 멀티태스킹`이 **시간을 절약하는 데 도움**이 될 수 있다. `동시성과 병렬성`은 `멀티태스킹`을 달성하는 방법이지만, 이 둘 사이에는 미묘하지만 중요한 차이점이 있다.

`동시성`은 **하위 작업을 번갈아 가며 수많은 작업을 처리하는 것**(일명 인터리빙)이고, `병렬성`은 **여러 작업을 동시에 수행하는 것**과 같다. 예를 들어, 휴대전화를 보다가 국물을 한 숟가락 떠먹기 위해 내려놓았다가 숟가락을 내려놓은 후 다시 휴대전화를 보는 경우, 이는 동시 작업이다. 반대로 한 손으로 밥을 먹으면서 다른 한 손으로 문자를 보낸다면 병렬 작업이다. 두 경우 모두 멀티태스킹이지만, 멀티태스킹을 처리하는 방식에는 미묘한 차이가 있다.

![](https://martin-thoma.com/images/2021/05/parallel-interleaved.png#75persize)

## 스레드

프로그래밍 맥락에서 프로세스의 더 큰 명령어 집합에서, 하위 작업은 **그 작업의 개별적인 부분**으로 생각할 수 있다. **서로 다른 하위 작업을 동시에 처리하는 일반적인 방법**은 `서로 다른 커널 스레드를 생성하는 것`이다. 이는 마치 각각 특정 작업을 처리하는 각각의 작업자가 동일한 명령어 집합과 리소스를 사용하면서 작업하는 것과 같다.

스레드가 `병렬 실행` 또는 `동시 실행`되는지는 **하드웨어의 상황에 따라 다르다**.

CPU의 코어 수가 동시에 실행되는 스레드 수보다 많다면, 각 스레드를 다른 코어에 할당해 병렬로 실행할 수 있다. 그러나 CPU의 코어 수가 스레드 수보다 적은 경우 운영 체제는 스레드 간에 인터리빙(동시성 처리)을 시작한다.

커널 스레드를 사용하는 경우, 개발자의 경험은 동일하게 유지되며 작업이 실제로 동시에 처리되든 병렬로 처리되든 큰 차이가 없다. 개발자는 성능을 개선하고 차단을 피하기 위해 스레드를 사용한다. 그러나 **사용 가능한 리소스에 따라 이러한 스레드를 처리하는 방법에 대한 최종 결정을 내리는 것**은 `운영 체제`다. 개발자가 스레드를 사용하는 한, 동시 실행이든 병렬 실행이든 상관없으며 두 경우 모두 **서로 다른 스레드의 명령이 실행되는 순서는 예측할 수 없다**.

따라서 개발자는 동일한 데이터에서 작동하는 두 개의 서로 다른 스레드에서 발생할 수 있는 `잠재적인 문제`(예: 경쟁 상태, 데드락, 라이브락 등)에 주의해야한다.

## 프로세스 생성, I/O 알림

스레드를 사용하는 것 외에도 동시성/병렬성을 달성할 수 있는 다른 방법이 있다. 예를 들어, 스레드만큼 효율적이지는 않지만, **여러 프로세스를 생성하는 것도 하나의 방법**이다. `CPU`는 여러 프로세스를 병렬 및 동시에 실행하기 때문에 여러 프로세스를 사용하여 **멀티태스킹**할 수 있다.

단점은 각 프로세스에 고유한 메모리 공간이 할당되어 있고 **스레드처럼 기본적으로 메모리 공간을 공유하지 않는다는 것**. 따라서 **서로 다른 프로세스가 동일한 상태에서 작동해야 하는 경우 공유 메모리 세그먼트, 파이프, 메시지 큐 또는 데이터베이스와 같은 일종의 IPC 메커니즘이 필요**하다.

## Node.js, 유저 스페이스에서의 동시성 예시

프로그래밍 언어는 운영 체제의 API(시스템 호출) 사용과 관련된 복잡성을 단순화하기 위해 `자체적인 동시성 메커니즘을 제공`하는 경우가 많다. 즉, **컴파일러나 인터프리터가 고수준 코드를 "운영 체제가 이해할 수 있는 저수준 시스템 호출로 변환"하여 사용자가 많은 생각을 할 필요가 없도록 해준다**.

`Node.js`는 자바스크립트 프로그램에서 **순차적 실행 흐름의 단일 스레드 환경에서 실행**되지만 `IO 작업과 같은 블로킹 작업`은 Node.js `워커 스레드`에 위임된다. 따라서 *Node.js는 개발자에게 이러한 차단 작업을 관리하는 복잡성을 드러내지 않고 "백그라운드에서 스레드를 사용"하여 이러한 차단 작업을 관리*한다.

작동 방식은 파일의 읽기 또는 쓰기, 네트워크 요청 전송과 같은 차단 작업은 일반적으로 Node.js에서 제공하는 내장 함수를 사용하여 처리된다. 일반적으로 이러한 함수를 호출할 때 콜백 함수를 매개변수로 전달하면 Node.js 워커 스레드가 작업을 완료할 때 제공한 콜백 함수를 실행할 수 있도록 한다.

![](https://miro.medium.com/v2/resize:fit:828/format:webp/1*U_zyHnKdlvjCdAQkoh0uuQ.png#75persize)

Node.js 동시성 내부 동작을 이해해보자

```js
setTimeout(() => {
  while (true) {
    console.log("a");
  }
}, 1000);

setTimeout(() => {
  while (true) {
    console.log("b");
  }
}, 1000);
```

이렇게 실행하면 "a"만 표시 된다. 이는 Node.js 인터프리터가 아직 사용 가능한 명령어가 있는 한 "현재 콜백"을 계속 실행하기 때문이다.

메인 코드의 모든 명령어가 실행되는 즉시 **Node.js 런타임 환경은 콜백 함수를 호출하기 시작**한다. 작성하는 메인 코드가 기본적으로 콜백으로 호출된다고 생각할 수도 있다. 위 예시에서 첫 번째 `setTimeout`은 제공된 콜백 함수와 함께 실행되고, 두 번째 `setTimeout`도 콜백 함수와 함께 실행된다. 1초가 지나면 "a"를 스팸으로 보내기 시작한다. **첫 번째 콜백이 호출되면 "그 콜백이 메인 스레드를 차지"하고, while 루프가 끝없이 실행되기 때문에 "b"는 절대 보이지 않게 된다**. 따라서 두 번째 콜백은 절대 호출되지 않는다.

프로그래밍 로직에 많은 비동기 콜백 기반 함수(예를 들어 `fs.readFile()`, `setTimeout()`, `setImmediate()` 같은 함수, 또한 `Promise.then()`)가 포함되어 있으면 **경쟁 상태가 쉽게 발생**할 수 있다.

`await`에서도 경쟁 상태가 발생할 수 있다. 왜냐하면 `await` 구문은 **현재 스코프를 기준으로 남은 코드를 `Promise`의 `then` 메서드 안의 콜백 함수로 랩핑한 것을 간단히 표현한 방식**으로 볼 수 있기 때문이다.

아래 테스트 코드를 보자.

```js
const { scheduler } = require("node:timers/promises"),
  test = async () => {
    let x = 0;

    const foo = async () => {
      let y = x;
      await scheduler.wait(100);
      x = y + 1;
    };

    await Promise.all([foo(), foo(), foo()]);
    console.log(x); // Returns 1, not 3
  },
  test2 = async () => {
    let x = 0;

    const foo = async () => {
      await scheduler.wait(100);
      let y = x;
      x = y + 1;
    };

    await Promise.all([foo(), foo(), foo()]);
    console.log(x); // Returns 3
  },
  main = () => {
    test();
    test2();
  };

main();
```

`test()`가 1을 기록하는 이유는 `foo` 함수들이 호출될 때 `await scheduler.wait(100)`를 만나자마자 기본적으로 완료되기 때문이다. 왜냐하면 내부적으로 `await scheduler.wait(100)`을 사용하면 다음과 같이 평가되기 때문이다.

```js
scheduler.wait(100).then(() => {
  x = y + 1;
});
```

따라서 첫 번째 `foo` 함수가 작업을 완료하면 _이제 콜백 함수가 후속 작업을 이어가야 하지만_, 그 콜백 함수는 100ms 후에 호출되므로, **Node.js 인터프리터는 유휴 상태로 있지 않고** 두 번째 및 세 번째 foo 함수를 *순서대로 계속 실행*한다. 또한 첫 번째 foo의 콜백이 트리거되기 전에 y 변수를 x 값으로 설정하고 콜백 함수로 `scheduler.wait`를 호출한.

결과적으로 콜백이 최종적으로 실행될 때 모두 이전 값인 x를 사용하여 x를 업데이트하므로 3이 아닌 1을 얻게 된다.

`test2()`를 실행할 때 3이 출력되는 이유는? `await`가 실행되는 위치가 다르고 다음과 같이 평가되기 때문.

```js
scheduler.wait(100).then(() => {
  let y = x; // y 변수 할당이 then 에서 이루어진다.
  x = y + 1;
});
```

따라서 경쟁 상태가 발생할 수 없다.

여기서 중요한 점은 "동시성"을 달성하는 방법은 한 가지가 아니며 달성 방법에 따라 프로그램의 성능이나 발생할 수 있는 문제, 주의해야 할 사항 등 여러 가지에 영향을 미칠 수 있다는 것이다.

## 출처

- [[번역] 동시성, 병렬성, 그리고 자바스크립트에 대한 이해](https://velog.io/@surim014/concurrency-and-parallelism)