# [디자인 패턴] Singleton

## 개념

싱글톤은 1회 한에 인스턴스화가 가능하고 전역에서 접근 가능한 클래스를 지칭한다. 만들어진 싱글톤 인스턴스는 전역에 공유되기 때문에 앱의 전역 상태 관리에 적합하다.

아래 예제를 확인해보자.

```js
let counter = 0;

class Counter {
  getInstance() {
    return this;
  }

  getCount() {
    return counter;
  }

  increment() {
    return ++counter;
  }

  decrement() {
    return --counter;
  }
}

const counter1 = new Counter();
const counter2 = new Counter();

console.log(counter1.getInstance() === counter2.getInstance()); // false
```

위 코드는 패턴을 만족하지 않는데, 인스턴스를 단 한번만 만들 수 있어야하기 때문이다. 인스턴스를 두번 호출하여 서로 별개의 인스턴스임을 확인할 수 있다. 같은 참조를 바라보고 있지 않아 동일한 인스턴스가 아니다.

<video src="https://patterns-dev-kr.github.io/design-singleton01.mp4" width="100%"  autoplay controls playsinline loop></video>

인스턴스를 export하기 전에 인스턴스를 `Object.freeze` 해서 객체를 사용하는 쪽에서 직접 수정을 할 수 없도록한다. 이렇게 처리된 인스턴스는 프로퍼티의 추가나 수정이 불가능하여 싱글톤 인스턴스의 프로퍼티를 덮어쓰는 실수를 예방할 수 있다.

```js
const singletonCounter = Object.freeze(new Counter());
```

## 장단점

인스턴스를 하나만 만들도록하면 꽤 많은 메모리를 절약 가능하다. 때문에 전역으로 사용가능한 하나의 인스턴스를 저장하기 위한 메모를 사용했으나 싱글톤은 안티패턴 혹은 자바스크립트에서 하면 안되는 것으로 언급된다.

객체지향 프로그래밍 언어에는 객체를 만들기 위한 클래스를 작성해야하고 이렇게 만든 객체는 `instance` 변수와 같이 클래스의 인스턴스가 된다.
때문에 자바스크립트에선 클래스를 작성하지 않아도 객체를 만들 수 있어 클래스를 생성해서 만드는 행위는 약간의 오버 엔지니어링에 가깝다. 객체 리터럴(`{}`)을 이용해도 문제 없다.

## 객체 리터럴 사용

```js
let count = 0;

const counter = {
  increment() {
    return ++count;
  },
  decrement() {
    return --count;
  },
};

Object.freeze(counter);
export { counter };
```

객체 레퍼런스가 넘어갔기 때문에 동일한 `counter`를 참조한다.

## 테스팅

싱글톤 패턴의 코드는 테스팅이 어렵다고 한다. 인스턴스를 매번 생성할 수 없기 때문. 모든 테스트는 이전 테스트에서 만들어진 전역 인스턴스를 수정할 수 밖에 없다. 테스트 실행에 순서가 생기면 작은 수정사항 하나로 전체 테스트의 실패로 이어진다. 때문에 하나의 테스트가 끝나면 인스턴스의 변경사항들을 초기화해주어야한다.

```js
import Counter from "../src/counterTest";

test("incrementing 1 time should be 1", () => {
  Counter.increment();
  expect(Counter.getCount()).toBe(1);
});

test("incrementing 3 extra times should be 4", () => {
  Counter.increment();
  Counter.increment();
  Counter.increment();
  expect(Counter.getCount()).toBe(4);
});

test("decrementing 1  times should be 3", () => {
  Counter.decrement();
  expect(Counter.getCount()).toBe(3);
});
```

## 전역 동작

싱글톤 인스턴스는 앱 전체에 참조가 되어야하고 전역 스코프에서 전역 변수를 접근할 수 있는 한 해당 변수는 앱 전체에 접근이 가능하기 때문에 전역 변수는 반드시 같은 동작을 구현하는데 사용해야 한다.

ES2015에서는 전역변수를 생성하는게 일반적이지 않고 `let`, `const`를 통해 변수를 `블록 스코프 내에 선언`하게 하여 실수로 전역에 변수를 선언하는 것을 예방해 준다. 또 `module` 시스템은 `export`와 `import`를 이용하여 전역 객체를 수정하지 않고도 모듈 내에서 전역으로 사용 가능한 변수를 만들게 해준다.

하지만 싱글톤은 일반적으로 전역 상태를 위해 사용되고, 여러 부분에서 수정 가능한 하나의 객체를 직접 접근하도록 설계하면 예외가 발생하기 쉬워진다.

어떤 코드는 데이터를 읽기 위해 전역 상태를 수정하기도 한다. 이 경우 실행 순서가 중요한데 데이터가 만들어지지 않았는데 사용할 수 없기 때문이다.

## React 상태 관리

React에선 전역 상태 관리를 위해 싱글톤 객체를 생성하는 것 대신, Redux 또는 Context API를 사용한다. 싱글톤과 유사해보이지만 싱글톤은 인스턴의 값을 직접 수정할 수 있는 반면에 이 도구들은 `읽기 전용 상태`를 제공한다. Redux 사용시 오직 dispatcher를 통해서 넘긴 액션에 대해 실행된 `순수함수 리듀서`를 통해서만 상태 업데이트가 가능하다.

## 출처

- [singleton-pattern](https://patterns-dev-kr.github.io/design-patterns/singleton-pattern/)
