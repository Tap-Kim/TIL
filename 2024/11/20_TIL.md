# 타입스크립트의 type과 interface

## type과 interface의 차이점

### 원시(Primitive) 타입

원시 타입은 타입스크립트에 내장된 타입으로 `number, string, boolean, null, undefined`이 포함된다.

다음과 같이 기본 타입에 대한 type의 별칭을 정의할 수 있다.

```ts
type Address = string;
```

코드 가독성을 높이기 위해 기본 타입과 유니온 타입을 결합하여 별칭을 정의하는 경우가 많다.

```ts
type NullOrUndefined = null | undefined;
```

그러나 기본 타입의 별칭에 인터페이스를 사용할 수는 없다. 인터페이스는 `객체` 타입에만 사용할 수 있다. 따라서 `원시 타입` 별칭을 정의해야 할 때는 `type`을 사용한다.

### 유니온(Union) 타입

유니온 타입을 사용하면 여러 타입 중 하나가 될 수 있는 값을 작성하고 다양한 원시, 리터럴 또는 복합 타입의 유니온을 만들 수 있다.

```ts
type Transport = "Bus" | "Car" | "Bike" | "Walk";
```

유니온 타입은 `type`을 통해서만 정의할 수 있다. 인터페이스에는 유니온 타입에 해당하는 것이 없다. 그러나 다음과 같이 두 인터페이스에서 새 유니온 타입을 만들 수 있다.

```ts
interface CarBattery {
  power: number;
}
interface Engine {
  type: string;
}
type HybridCar = Engine | CarBattery;
```

### 함수 타입

타입스크립트에서 함수 타입은 함수의 타입 서명을 나타낸다. 타입 별칭을 사용하여 매개변수와 반환 타입을 지정하여 함수 타입을 정의해야 한다.

```ts
// type
type AddFn = (num1: number, num2: number) => number;

// interface
interface IAdd {
  (num1: number, num2: number): number;
}
```

타입을 사용할 때 `:` 대 `=>`를 사용하는 인터페이스의 미묘한 구문 차이를 제외하고는 타입과 인터페이스 모두 유사하게 함수 타입을 정의한다. 이 경우 타입이 더 짧고 읽기 쉽기 때문에 타입이 선호된다.

함수 타입을 정의할 때 타입을 사용하는 또 다른 이유는 인터페이스에 없는 기능 때문이다. 함수가 더 복잡해지면 조건부 타입, 매핑된 타입 등과 같은 고급 타입 기능을 활용할 수 있다.

```ts
type Car = "ICE" | "EV";
type ChargeEV = (kws: number) => void;
type FillPetrol = (type: string, liters: number) => void;
type RefillHandler<A extends Car> = A extends "ICE"
  ? FillPetrol
  : A extends "EV"
  ? ChargeEV
  : never;
const chargeTesla: RefillHandler<"EV"> = (power) => {};
const refillToyota: RefillHandler<"ICE"> = (fuelType, amount) => {};
```

여기서는 조건부 타입과 공용체 타입으로 `RefillHandler` 타입을 정의한다. 이는 `EV` 및 `ICE` 핸들러에 type-safe 방식으로 통합된 함수 서명을 제공한다. 인터페이스에는 조건부 및 공용체 타입이 없기 때문에 동일한 기능을 구현할 수 없다.

## 선언 병합(Declaration merging)

선언 병합은 인터페이스 전용 기능이다. 선언 병합을 사용하면 인터페이스를 여러 번 정의할 수 있으며 타입스크립트 컴파일러는 이러한 정의를 단일 인터페이스 정의로 자동 병합한다. 다음 예제에서는 두 개의 클라이언트 인터페이스 정의가 타입스크립트 컴파일러에 의해 하나로 병합되어 클라이언트 인터페이스를 사용할 때 두 개의 프로퍼티를 갖게 된다.

```ts
interface Client {
  name: string;
}

interface Client {
  age: number;
}

const harry: Client = {
  name: "Harry",
  age: 41,
};
```

타입 별칭은 같은 방식으로 병합할 수 없다. 위의 예에서와 같이 `Client` 타입을 두 번 이상 정의하려고 하면 오류가 발생한다.

![](https://blog.logrocket.com/wp-content/uploads/2023/03/define-type-more-than-once-error.png)

선언 병합은 적절한 위치에서 사용하면 매우 유용할 수 있다. 선언 병합의 일반적인 사용 사례 중 하나는 특정 프로젝트의 필요에 맞게 서드파티 라이브러리의 타입 정의를 확장하는 것이다.

선언을 병합해야 하는 경우 인터페이스를 사용하는 것이 좋다.

## 확장(Extends) vs. intersection

인터페이스는 하나 또는 여러 개의 인터페이스를 확장할 수 있다. 확장 키워드를 사용하면 새 인터페이스는 기존 인터페이스의 모든 속성과 메서드를 상속하는 동시에 새 속성을 추가할 수 있다.

예를 들어 `Client` interface를 확장하여 `VIPClient` 인터페이스를 만들 수 있다.

```ts
interface VIPClient extends Client {
  benefits: string[];
}
```

타입에 대해 유사한 결과를 얻으려면 유니온 연산자를 사용해야 한다.

```ts
type VIPClient = Client & { benefits: string[] };
```

정적으로 알려진 멤버를 사용하여 타입 별칭에서 인터페이스를 확장할 수도 있다.

```ts
type Client = {
  name: string;
};

interface VIPClient extends Client {
  benefits: string[];
}
```

단, 유니온 타입은 예외다. 유니온 타입에서 인터페이스를 확장하려고 하면 다음 오류가 발생한다.

```ts
type Jobs = "salary worker" | "retired";

interface MoreJobs extends Jobs {
  description: string;
}
```

![](https://blog.logrocket.com/wp-content/uploads/2023/03/union-type-not-statically-known-error.png)

이 오류는 유니온 타입이 정적으로 알려지지 않았기 때문에 발생한다. 인터페이스 정의는 컴파일 시점에 정적으로 알려져 있어야 한다.

타입 별칭은 아래와 같이 유니온을 사용하여 인터페이스를 확장할 수 있다.

```ts
interface Client {
  name: string;
}
type VIPClient = Client & { benefits: string[] };
```

간단히 말해 인터페이스와 타입 별칭 모두 확장할 수 있다. 인터페이스는 정적으로 알려진 타입 별칭을 확장할 수 있고, 타입 별칭은 유니온 연산자를 사용하여 인터페이스를 확장할 수 있습니다.

## extends시 충돌 처리하기

타입과 인터페이스의 또 다른 차이점은 동일한 속성 이름을 가진 것에서 확장하려고 할 때 충돌이 처리되는 방식이다.

인터페이스를 확장할 때 아래 예제에서와 같이 동일한 속성 키는 허용되지 않는다.

```ts
interface Person {
  getPermission: () => string;
}

interface Staff extends Person {
  getPermission: () => string[];
}
```

![](https://blog.logrocket.com/wp-content/uploads/2023/03/conflict-detected-error-thrown.png)

타입 별칭은 충돌을 다르게 처리한다. 타입 별칭이 동일한 속성 키를 가진 다른 타입을 확장하는 경우 오류를 발생시키는 대신 모든 속성을 자동으로 병합한다.

다음 예제에서는 유니온 연산자가 두 `getPermission` 선언의 메서드 서명을 병합하고 타입 안전 방식으로 반환 값을 얻을 수 있도록 유니온 타입 매개변수의 범위를 좁히는 데 `typeof` 연산자를 사용한다.

```ts
type Person = {
  getPermission: (id: string) => string;
};

type Staff = Person & {
  getPermission: (id: string[]) => string[];
};

const AdminStaff: Staff = {
  getPermission: (id: string | string[]) => {
    return (typeof id === "string" ? "admin" : ["admin"]) as string[] & string;
  },
};
```

두 속성의 타입 유니온로 인해 예기치 않은 결과가 발생할 수 있다는 점에 유의해야 한다. 아래 예제에서 확장된 타입 `Staff`의 `name` 속성은 `string`과 `number`가 동시에 될 수 없으므로 `never`가 된다.

```ts
type Person = {
  name: string;
};

type Staff = person & {
  name: number;
};
// error: Type 'string' is not assignable to type 'never'.(2322)
const Harry: Staff = { name: "Harry" };
```

요약하면, 인터페이스는 컴파일 시 속성 또는 메서드 이름 충돌을 감지하여 오류를 생성하는 반면, 타입 유니온는 오류를 발생시키지 않고 속성 또는 메서드를 병합한다. 따라서 함수를 오버로드해야 하는 경우 타입 별칭을 사용해야한다.

## intersection 보다 extends 선호하는 이유

인터페이스를 사용할 때 타입스크립트는 일반적으로 오류 메시지, 도구 설명 및 IDE에서 인터페이스의 모양을 더 잘 표시한다. 또한 얼마나 많은 타입을 결합하거나 확장하든 훨씬 더 읽기 쉽다.

`타입 A = B & C;` 처럼 두 개 이상의 타입의 유니온을 사용하는 타입 별칭과 비교해보면, `타입 X = A & D;` 처럼 다른 유니온에 해당 별칭을 사용하도록 입력하면 타입스크립트는 결합된 타입의 구조를 표시하기 어려워 오류 메시지에서 타입의 모양을 이해하기 어렵게 된다.

타입스크립트는 한 인터페이스가 다른 인터페이스를 확장하는지 또는 두 인터페이스가 호환되는지 등 인터페이스 간의 평가된 관계의 결과를 캐시한다. 이 접근 방식은 향후 동일한 관계를 참조할 때 전반적인 성능을 향상시킨다.

반대로 타입스크립트는 유니온으로 작업할 때 이러한 관계를 캐시하지 않는다. 타입 유니온을 사용할 때마다 타입스크립트는 전체 유니온을 다시 평가해야 하므로 효율성 문제가 발생할 수 있다.

이러한 이유로 타입 유니온에 의존하는 대신 인터페이스 확장을 사용하는 것이 좋다.

## 인터페이스 또는 타입 별칭을 사용하여 클래스 구현하기

타입스크립트에서는 인터페이스나 타입 별칭을 사용하여 클래스를 구현할 수 있다.

```ts
interface Person {
  name: string;
  greet(): void;
}

class Student implements Person {
  name: string;
  greet() {
    console.log("hello");
  }
}

type Pet = {
  name: string;
  run(): void;
};

class Cat implements Pet {
  name: string;
  run() {
    console.log("run");
  }
}
```

위에서 살펴본 것처럼 인터페이스와 타입 별칭 모두 비슷하게 클래스를 구현하는 데 사용할 수 있으며, 유일한 차이점은 유니온 타입을 구현할 수 없다는 점이다.

```ts
type primaryKey = { key: number } | { key: string };

// 유니온 타입 불가
class RealKey implements primaryKey {
  key = 1;
}
```

![](https://blog.logrocket.com/wp-content/uploads/2023/03/class-represents-specific-data-shape-error.png)

위의 예제에서는 클래스가 특정 데이터 형상을 나타내지만 유니온 타입은 여러 데이터 타입 중 하나일 수 있기 때문에 타입스크립트 컴파일러가 오류를 발생시킨다.

## 튜플 타입으로 작업하기

타입스크립트에서 튜플 타입을 사용하면 각 요소에 *데이터 타입이 있는 고정된 수의 요소*로 배열을 표현할 수 있다. 고정된 구조의 데이터 배열로 작업해야 할 때 유용할 수 있다.

튜플은 길이가 고정되어 있고 각 위치에 타입이 할당되어 있으므로 이 구조를 위반하는 방식으로 요소를 추가, 제거 또는 수정하려고 하면 타입스크립트에서 오류를 발생시킨다.

```ts
type TeamMember = [name: string, role: string, age: number];
```

```ts
// member[3]; // Error: Tuple type '[string, string, number]' of length '3' has no element at index '3'.
const member: TeamMember = ['Alice', ‘Dev’, 28];

```

인터페이스는 튜플 타입을 직접 지원하지 않는다. 아래 예제처럼 몇 가지 해결 방법을 만들 수는 있지만, 튜플 타입을 사용하는 것만큼 간결하거나 가독성이 높지는 않다.

```ts
interface ITeamMember extends Array<string | number> {
  0: string;
  1: string;
  2: number;
}

const peter: ITeamMember = ["Harry", "Dev", 24];
const Tom: ITeamMember = ["Tom", 30, "Manager"]; //Error: Type 'number' is not assignable to type 'string'.
```

튜플과 달리 이 인터페이스는 일반 배열 타입을 확장하여 처음 세 개를 초과하는 요소를 얼마든지 가질 수 있다. 이는 타입스크립트의 배열이 동적이며 인터페이스에 명시적으로 정의된 것 이외의 인덱스에 액세스하거나 값을 할당할 수 있기 때문이다.

```ts
const peter: ITeamMember = [’Peter’, 'Dev', 24];
console.log(peter[3]); // No error, even though this element is undefined.
```

## 고급 타입 기능

타입스크립트는 인터페이스에서는 찾아볼 수 없는 다양한 고급 타입 기능을 제공한다. 타입스크립트의 고유한 기능 중 일부는 다음과 같다.

- **타입 추론**: 변수와 함수의 용도에 따라 타입을 유추할 수 있다. 이를 통해 코드의 양을 줄이고 가독성을 향상시킨다.
- **조건부 타입**: 다른 타입에 의존하는 조건부 동작이 있는 복잡한 타입 표현식을 만들 수 있다.
- [**타입 가드**](https://blog.logrocket.com/how-to-use-type-guards-타입스크립트/): 변수 타입에 따라 정교한 제어 흐름을 작성하는 데 사용된다.
- **매핑된 타입**: 기존 객체 타입을 새로운 타입으로 변환한다.
- **유틸리티 타입**: 타입을 조작하는 데 도움이 되는 기본 제공 유틸리티 세트

## 타입과 인터페이스의 사용 시기

타입 별칭과 인터페이스는 비슷하지만 이전 섹션에서 살펴본 것처럼 미묘한 차이가 있다.

거의 모든 인터페이스 기능은 타입으로 제공되거나 이와 동등한 기능을 가지고 있지만 선언 병합은 한 가지 예외다. 인터페이스는 일반적으로 기존 라이브러리를 확장하거나 새 라이브러리를 작성하는 등 선언 병합이 필요한 경우에 사용해야 한다. 또한 객체 지향 상속 스타일을 선호하는 경우 인터페이스와 함께 `extends` 키워드를 사용하는 것이 타입 별칭과 유니온을 사용하는 것보다 더 가독성이 좋은 경우가 많다.

`extends` 키워드가 포함된 인터페이스는 유니온 키워드가 포함된 타입 별칭에 비해 *컴파일러의 성능을 향상*시킨다.

그러나 타입의 많은 기능은 인터페이스로는 구현하기 어렵거나 불가능하다. 예를 들어 타입스크립트는 조건부 타입, 제네릭 타입, 타입 가드, 고급 타입 등과 같은 풍부한 기능을 제공하는데 이러한 기능을 사용하여 잘 제약된 타입 시스템을 구축하여 앱을 강력하게 타입화할 수 있다. 인터페이스는 이를 달성할 수 없다.

대부분의 경우 개인 취향에 따라 서로 바꿔서 사용할 수 있다. 하지만 다음과 같은 사용 사례에서는 타입 별칭을 사용해야 한다.

- 원시 타입의 새 이름을 만들기
- 유니온 타입, 튜플 타입, 함수 타입 또는 기타 더 복잡한 타입 정의하기
- 함수 오버로드하기
- 매핑된 타입, 조건부 타입, 타입 가드 또는 기타 고급 타입 기능을 사용하기

인터페이스에 비해 타입은 표현력이 더 풍부하다. 인터페이스에서는 사용할 수 없는 고급 타입 기능이 많으며, 타입스크립트가 발전함에 따라 이러한 기능은 계속 늘어나고 있다.

아래는 인터페이스에서 구현할 수 없는 고급 타입 기능의 예다.

```ts
type Client = {
  name: string;
  address: string;
};
type Getters<T> = {
  [K in keyof T as `get${Capitalize<string & K>}`]: () => T[K];
};
type clientType = Getters<Client>;
// type clientType = {
//     getName: () => string;
//     getAddress: () => string;
// }
```

매핑된 타입, 템플릿 리터럴 타입, `keyof` 연산자를 사용하여 모든 객체 타입에 대한 getter 메서드를 자동으로 생성하는 타입을 만들었다.

또한 함수형 프로그래밍 패러다임에 잘 부합하기 때문에 많은 개발자가 타입을 선호한다. 풍부한 타입 표현을 사용하면 함수형 구성, 불변성 및 기타 함수형 프로그래밍 기능을 type-safe 방식으로 더 쉽게 달성할 수 있다.
