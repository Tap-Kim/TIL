# AsyncGenerator와 AsyncGeneratorFuction

## 정리

- `AsyncGenerator` 객체는 `async generator function`에 의해 반환되고, *비동기 순회 프로토콜 + 비동기 반복자 프로토콜*을 준수한다.
- `Async generator 메서드`는 항상 `Promise` 객체를 *yield* 한다.
- `AsyncGenerator`는 숨겨진 `AsyncIterator` 클래스의 *하위 클래스*이다.
- `AsyncGeneratorFunction.prototype`의 프로퍼티인 `prototype`은 `async generator function`에서 공유하며, 그 값은 `AsyncGenerator.prototype`이다. `async function*` 구문 또는 `AsyncGeneratorFunction()` 생성자를 사용하여 생성된 각 비동기 제너레이터 함수에는 자체 프로토타입 프로퍼티가 있고, 그 프로토타입은 `AsyncGeneratorFunction.prototype.prototype`이다. 비동기 제너레이터 함수가 호출되면 해당 프로토타입 프로퍼티는 반환된 비동기 제너레이터 객체의 프로토타입이 된다.

![](https://mdn.github.io/shared-assets/images/diagrams/javascript/asyncgeneratorfunction/prototype-chain.svg)

> 다이어그램은 비동기 생성기 함수의 프로토타입 체인과 그 인스턴스를 보여준다. 각 속이 빈 화살표는 상속 관계(즉, 프로토타입 링크)를 나타내고, 각 실선 화살표는 속성 관계를 나타냅니다. gen—에서 genFunc에 액세스할 수 있는 방법은 없으며 인스턴스 관계만 있다는 점에 유의한다.


### 참고 
- 주요 키워드: 비동기, generator, prototype, iterator, iterable, async-await, promise
- 관련 기술: AsyncGenerator, AsyncGeneratorFuction, AsyncIterator

## 무엇을 알았는지

### AsyncGenerator 생성자

- AsyncGenerator 생성자와 대응되는 JavaScript 객체는 없다.
- async generator function에서 AsyncGenerator의 인스턴스를 반환해야한다.

```js
async function* createAsyncGenerator() {
  yield await Promise.resolve(1);
  yield await Promise.resolve(2);
  yield await Promise.resolve(3);
}
const asyncGen = createAsyncGenerator();
asyncGen.next().then((res) => console.log(res.value)); // 1
asyncGen.next().then((res) => console.log(res.value)); // 2
asyncGen.next().then((res) => console.log(res.value)); // 3
```

- async generator function이 생성한 모든 객체가 공유하는 프로토타입 객체인 `숨겨진 객체`만 있다. 
- 이는 클래스처럼 보이게 하기 위해 `AsyncGenerator.prototype`으로 그려지지만, `AsyncGeneratorFunction`은 실제 Javascript 객체이기 때문에 `AsyncGeneratorFunction.prototype.prototype`이 더 적절하다.([AsyncGeneratorFunction.prototype.prototype](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/AsyncGeneratorFunction/prototype)의 프로토타입 체인)

### AsyncGenerator 인스턴스 속성

> 참고: `AsyncGenerator` 객체는 자신을 생성한 async generator function의 `참조를 저장하지 않는다.`

- **AsyncGenerator.prototype[@@toStringTag]**
    - `@@toStringTag` 속성의 초기 값은 문자열 `AsyncGenerator`. 
    - 이 속성은 `Object.prototype.toString()`에서 사용한다.

### AsyncGenerator 인스턴스 메서드

부모 `AsyncIterator`에서 인스턴스 메서드를 상속한다.

- **AsyncGenerator.prototype.next()**
    - `Promise`를 반환하며, `yield 표현식에 의해 산출`되어 주어진 값으로 이행한다.

- **AsyncGenerator.prototype.return()**
    - 현재 일시 중단된 위치에서 생성기 본문에 `return 문이 삽입되어 생성기를 종료`하고 `try...finally 블록과 결합`하여 정리 작업을 수행할 수 있는 것처럼 작동한다.

- **AsyncGenerator.prototype.throw()**
    - 현재 일시 중단된 위치의 생성기 본문에 `throw 문을 삽입하여 생성기에 오류 상태를 알리고 오류를 처리`하거나 정리를 수행하고 스스로 닫을 수 있도록 하는 것처럼 작동한다.

### AsyncGenerator 순회
```js
function delayedValue(time, value) {
  return new Promise((resolve /*, reject*/) => {
    setTimeout(() => resolve(value), time);
  });
}

async function* generate() {
  yield delayedValue(2000, 1);
  yield delayedValue(100, 2);
  yield delayedValue(500, 3);
  yield delayedValue(250, 4);
  console.log("All done!");
}

async function main() {
  for await (const value of generate()) {
    console.log("value", value);
  }
}

main().catch((e) => console.error(e));

```

## 자바스크립트의 AsyncGenerator를 사용되는 라이브러리나 사용처(with. GPT)

자바스크립트의 AsyncGenerator는 비동기 반복 작업을 처리할 때 매우 유용한 기능으로, 특히 비동기 데이터를 스트리밍 방식으로 처리할 때 자주 사용됩니다. AsyncGenerator를 사용하는 주요 라이브러리나 사용처를 몇 가지 소개하겠습니다.

1. 라이브러리
  -	Node.js 스트림: Node.js에서 비동기 스트림을 처리할 때 AsyncGenerator가 유용합니다. 예를 들어, 네트워크 요청, 파일 읽기, 데이터베이스 쿼리 등을 비동기적으로 처리하고, 이를 반복적으로 소비할 때 사용됩니다.
  -	RxJS: 비동기 데이터 스트림을 관리하는 RxJS 라이브러리에서 from 연산자를 사용하여 AsyncGenerator를 Observable로 변환할 수 있습니다. 이를 통해 복잡한 비동기 흐름을 보다 선언적으로 관리할 수 있습니다.

2. 사용처

	-	API 응답 스트리밍: 클라이언트가 서버로부터 데이터를 스트리밍 방식으로 받을 때, AsyncGenerator를 사용하여 데이터를 점진적으로 처리할 수 있습니다. 예를 들어, 페이징을 사용하지 않고 서버로부터 데이터를 스트리밍 방식으로 전송받아 하나씩 처리하는 경우에 사용됩니다.
	-	비동기 순회: 비동기 작업을 순차적으로 처리해야 할 때 for await...of 루프와 함께 사용됩니다. 예를 들어, 여러 비동기 작업을 순차적으로 실행하면서 각 작업의 결과를 처리하는 경우에 유용합니다.
	-	파일 처리: 파일 시스템에서 대용량 파일을 비동기적으로 읽을 때, 파일을 스트림으로 읽어들이면서 점진적으로 처리할 수 있습니다.
	-	데이터베이스 쿼리: 대용량 데이터베이스 쿼리 결과를 한 번에 로드하지 않고, AsyncGenerator를 사용하여 필요한 만큼만 비동기적으로 가져와 처리할 수 있습니다.

이 외에도 AsyncGenerator는 비동기 데이터 흐름을 관리하는 다양한 상황에서 유용하게 사용됩니다. 비동기 작업의 결과를 점진적으로 처리하거나, 대량의 데이터를 효율적으로 다루기 위한 방법을 제공하기 때문에 많은 현대적인 자바스크립트 애플리케이션에서 활용되고 있습니다.



### 출처
- [AsyncGenerator - MDN](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/AsyncGenerator)
- [AsyncGeneratorFunction.prototype.prototype](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/AsyncGeneratorFunction/prototype)
