# 타입스크립트의 타입 가드 사용법

타입 가드는 일반적으로 조건부 블록 내에서 변수의 타입에 대한 정보를 얻는 데 사용되는 타입스크립트 기법이다. 타입 가드는 Boolean을 반환하는 **일반 함수**로, _타입을 가져와서 타입스크립트에 더 구체적인 것으로 범위를 좁힐 수 있는지_ 알려준다.

타입 가드는 반환된 Boolean에 따라 테스트된 값이 정해진 타입인지 확인하는 고유한 속성을 가지고 있다.

타입스크립트는 `typeof`, `instanceof`와 `in` 같은 내장 자바스크립트 연산자를 사용하여 객체에 속성이 포함되어 있는지 확인한다. 타입 가드를 사용하면 타입스크립트 컴파일러에 **특정 컨텍스트에서 변수에 대한 특정 타입을 유추하도록 지시**할 수 있다. 이 프로세스는 **인수의 타입이 지정된 타입과 일치하는지 확인**하여 타입 정확성과 코드 안정성을 향상시킨다.

타입 가드는 일반적으로 타입을 좁히는 데 사용되며 기능 감지와 매우 유사하여 값의 올바른 메서드, 프로토타입 및 속성을 감지할 수 있다.

## `instanceof` 타입 가드

`instanceof`는 값이 주어진 **생성자 함수**나 **클래스의 인스턴스**인지 확인하는 데 사용할 수 있는 내장형 타입 가드다. 이 타입 가드를 사용하면 객체나 값이 클래스에서 파생되었는지 테스트할 수 있으므로 인스턴스 타입의 타입을 결정하는 데 유용하다. `instanceof`

타입 가드의 기본 구문은 다음과 같다.

```ts
objectVariable instanceof ClassName;
```

```ts
interface Accessory {
  brand: string;
}
class Necklace implements Accessory {
  kind: string;
  brand: string;
  constructor(brand: string, kind: string) {
    this.brand = brand;
    this.kind = kind;
  }
}
class bracelet implements Accessory {
  brand: string;
  year: number;
  constructor(brand: string, year: number) {
    this.brand = brand;
    this.year = year;
  }
}
const getRandomAccessory = () => {
  return Math.random() < 0.5
    ? new bracelet("cartier", 2021)
    : new Necklace("choker", "TASAKI");
};
let Accessory = getRandomAccessory();
if (Accessory instanceof bracelet) {
  console.log(Accessory.year);
}
if (Accessory instanceof Necklace) {
  console.log(Accessory.brand);
}
```

위의 `getRandomAccessory` 함수는 `Necklace` 또는 `bracelet` 객체 중 하나를 반환하는데, 둘 다 액세서리 `Accessory`를 구현하기 때문이다. `Necklace` 와 `bracelet` 생성자 서명은 모두 다르며, [`instanceof` 타입 가드](https://www.typescriptlang.org/docs/handbook/2/narrowing.html#instanceof-narrowing)는 두 생성자 서명을 비교하여 효과적으로 타입을 결정한다.

## `typeof` 타입 가드

[`typeof` 타입 가드](https://www.typescriptlang.org/docs/handbook/2/typeof-types.html)는 변수의 타입을 결정하는 데 사용된다. 타입 가드는 매우 제한적이고 얕은 조건인데, 자바스크립트에서 인식하는 다음 타입만 결정할 수 있다.

- `boolean`
- `string`
- `bigint`
- `symbol`
- `undefined`
- `function`
- `number`

이 목록에 속하지 않는 모든 항목에 대해 `typeof` 타입 가드는 단순히 `object`를 반환한다. `typeof `타입 가드는 다음 두 가지 방법으로 작성할 수 있다.

```ts
typeof v !== "typename";
// 또는
typeof v === "typename";
```

`typename`은 `string`, `number`, `symbol`, 또는 `boolean`이 될 수 있다.

아래 예제에서 `StudentId`에는 `string` | `number` 타입의 유니온 매개 변수 항목이 있다. 변수가 `string`이면 `Student`가 출력되고, `number`인 경우 `Id`가 출력되는 것을 볼 수 있다. [`typeof` 타입 가드](https://www.typescriptlang.org/docs/handbook/2/typeof-types.html)는 `x`에서 타입을 추출하는 데 도움이 된다.

```ts
function StudentId(x: string | number) {
  if (typeof x == "string") {
    console.log("Student");
  }
  if (typeof x === "number") {
    console.log("Id");
  }
}
StudentId(`446`); //prints Student
StudentId(446); //prints Id
```

## `in` 타입 가드

[`in` 타입 가드](https://www.typescriptlang.org/docs/handbook/2/narrowing.html#the-in-operator-narrowing)는 객체에 특정 프로퍼티가 있는지 확인하여 이를 통해 다른 타입을 구분한다. 일반적으로 해당 프로퍼티가 해당 객체에 존재하는지 여부를 나타내는 boolean을 반환한다. 이 가드는 브라우저 지원 여부를 확인할 뿐만 아니라 범위를 좁히는 기능에 사용된다. `in` 타입 가드의 기본 구문은 다음과 같다.

```ts
propertyName in objectName;
```

아래 예제에서 `in` 타입 가드는 속성 집이 존재하는지 확인한다. 존재하는 경우 `true`를 반환하고, 존재하지 않는 경우 `false`를 반환한다.

```ts
"house" in { name: "test", house: { parts: "door" } }; // => true
"house" in { name: "test", house: { parts: "windows" } }; // => true
"house" in { name: "test", house: { parts: "roof" } }; // => true
"house" in { name: "test" }; // => false
"house" in { name: "test", house: undefined }; // => true
```

```ts
interface Pupil {
  ID: string;
}
interface Adult {
  SSN: number;
}
interface Person {
  name: string;
  age: number;
}
let person: Pupil | Adult | Person = {
  name: "Britney",
  age: 6,
};
const getIdentifier = (person: Pupil | Adult | Person) => {
  if ("name" in person) {
    return person.name;
  } else if ("ID" in person) {
    return person.ID;
  }
  return person.SSN;
};
```

## `is` 연산자

`is` 연산자는 값이나 변수가 특정 타입인지 확인한다. 변수나 표현식의 타입을 좁히는 데 사용할 수 있는 타입 가드다. 특정 코드 블록 내에서 변수의 타입을 좁히기 위해 사용자 정의 타입 가드 함수와 함께 자주 사용된다.

이 연산자를 사용하면 런타임에 타입 테스트를 통해 값이 특정 타입인지 여부를 확인할 수 있습니다. **변수에 특정 연산을 수행하기 전에 변수가 특정 타입인지 확인하려는 경우에 특히 유용하다**. `is` 연산자의 기본 구문은 다음과 같다.

```ts
variablename is typename
```

```ts
interface Cat {
  meow(): void;
}
interface Dog {
  bark(): void;
}
function isCat(pet: Dog | Cat): pet is Cat {
  return (pet as Cat).meow !== undefined;
}
let pet: Dog | Cat;
// 'is' 키워드 사용
if (isCat(pet)) {
  pet.meow();
} else {
  pet.bark();
}
```

위의 코드는 변수 `pet`의 타입을 확인하고 타입에 따라 다르게 작동한다. `pet`의 타입이 `Cat`이면 `meow()` 메서드를 호출한다. 반려동물의 타입이 `Dog`이면 `bark()` 메서드를 호출한다.

`isCat()` 함수는 반려동물에 `meow()` 메서드가 존재하는지 확인하여 반려동물의 타입이 `Cat`인지 확인하는 타입 가드다. 메서드가 있으면 이 함수는 `true`를 반환하여 반려동물의 타입이 `Cat`임을 나타낸다. 그렇지 않으면 `false`를 반환. `is` 키워드는 `if` 문에서 타입 검사를 수행하는 데 사용된다.

## 동일성 좁히기 타입 가드

동일성 범위 좁히기는 **표현식의 값을 확인**한다. 두 변수가 같으려면 두 변수의 타입이 모두 같아야 한다. 변수의 타입을 알 수 없지만 정확한 타입을 가진 다른 변수와 동일한 경우 타입스크립트는 잘 알려진 변수가 제공하는 정보로 첫 번째 변수의 타입을 좁힌다.

```ts
function getValues(a: number | string, b: string) {
  if (a === b) {
    // 여기서 문자열로 좁혀짐
    console.log(typeof a); // string
  } else {
    // 좁혀지지 않으면 타입을 알 수 없음
    console.log(typeof a); // number or string
  }
}
```

변수 `a`가 변수 `b`와 같으면 둘 다 같은 타입을 가져야 합니다. 이 경우 타입스크립트는 문자열로 범위를 좁힌다. 범위를 좁히지 않으면 숫자나 문자열일 수 있으므로 `a`의 타입이 여전히 불분명해진다.

## 사용자 정의 타입 가드

사용자 정의 타입 가드를 만드는 것은 일반적으로 타입 가드를 사용하는 가장 강력한 옵션이다. [사용자 정의 타입 가드를 직접 작성](https://www.typescriptlang.org/docs/handbook/2/narrowing.html#using-type-predicates)하여 만들면 확인할 수 있는 항목에 제한이 없다. 그러나 사용자 정의 타입 가드를 잘못 작성하면 많은 오류가 발생하니 정확성이 중요하다.

```ts
interface Necklace {
  kind: string;
  brand: string;
}
interface bracelet {
  brand: string;
  year: number;
}
type Accessory = Necklace | bracelet;

const isNecklace = (b: Accessory): b is Necklace => {
  return (b as Necklace).kind !== undefined;
};
const Necklace: Accessory = { kind: "Choker", brand: "TASAKI" };
const bracelet: Accessory = { brand: "Cartier", year: 2021 };
console.log(isNecklace(bracelet)); //Logs false
console.log(isNecklace(Necklace)); //Logs true
```

위 코드에서 사용자 정의 타입가드 `b`가 `Necklace`인 경우 타입스크립트는 boolean 값만 반환하는 대신 타입을 `Necklace`로 축소한다.

## 커스텀 타입 가드

타입스크립트 커스텀 타입 가드는 런타임에 값이나 표현식의 타입을 확인할 수 있는 함수다. 앞서 살펴본 것과 같이 기본 제공 타입 가드로는 쉽게 확인할 수 없는 복잡한 타입이나 조건을 확인하는 데 유용하다.

커스텀 타입 가드는 조건식 및 기타 코드에서 값이나 표현식의 타입을 확인해야 하는 부분에 사용할 수 있다.

커스텀 타입 가드를 사용하면 더 복잡한 타입과 조건을 확인하고 일반적으로 코드의 가독성을 향상시킬 수 있다.

```ts
function isBaby(obj: any): obj is Baby {
  return typeof obj === "object" && obj !== null && "sound" in obj;
}

interface Baby {
  sound: string;
}

function makeSound(baby: any) {
  if (isBaby(baby)) {
    console.log("Making a sound:", baby.sound);
  } else {
    console.log("Not a valid baby.");
  }
}
```

위에서 `makeSound` 함수는 any 타입의 인자 `baby`를 받는다. 이 함수는 `baby` 객체가 `sound`라는 프로퍼티를 가지고 있는지 확인하는 `isBaby` 타입 가드를 만족하는지 확인한다. `baby` 개체가 `isBaby` 타입 가드를 만족하면 콘솔에 `baby` 소리를 로깅힌다. 그렇지 않으면 콘솔에 "유효한 `baby`가 아닙니다."를 기록한다.

`isBaby`는 사용자 정의 타입 가드 함수다. 이 함수는 객체에 `baby`의 특징인 `sound` 속성이 있는지 확인한다.

## 객체 타입 확인을 위한 타입스크립트 타입 가드

타입 가드는 객체의 타입을 검사하거나 확인하는 데 사용할 수 있다. 타입 가드를 사용하면 객체가 가진 특정 속성을 확인하여 해당 속성과 관련된 작업을 수행할 수 있다.

또한 타입 가드를 사용하여 객체가 숫자, 문자열 또는 특정 클래스와 같은 특정 타입인지 확인할 수 있다.

```ts
function `isCar`(vehicle: any): vehicle is Car {
  return (
    typeof vehicle === "object" &&
    vehicle !== null &&
    "brand" in vehicle &&
    "model" in vehicle &&
    "year" in vehicle
  );
}

interface Car {
  brand: string;
  model: string;
  year: number;
}

function inspectVehicle(vehicle: any) {
  if (isCar(vehicle)) {
    // 이 블록 내에서 타입스크립트는 'vehicle'이 'Car' 타입인지 알고 있다.
    console.log("Brand:", vehicle.brand);
    console.log("Model:", vehicle.model);
    console.log("Year:", vehicle.year);
  } else {
    console.log("Not a valid car object.");
  }
}
```

위의 코드는 주어진 객체가 `Car` 타입인지 확인한. 여기에는 속성 `brand`, `model` 및 `year`를 정의하는 `Car` 인터페이스가 포함되어 있다. 또한 객체를 매개변수로 받고 `isCar` 타입 가드 함수를 사용하여 `Car`인지 확인하는 `inspectVehicle` 함수도 포함되어 있다. `Car`인 경우 차량의 `brand`, `model`, `year`를 기록한다. 그렇지 않으면 유효한 자동차 객체가 아님을 기록한다. 차량 매개변수가 실제로 `Car`인지 확인하기 위해 `inspectVehicle` 함수를 사용한다. 차량이 `Car`인 경우 차량의 `brand`, `model`, `year`를 기록한다. 차량이 자동차가 아닌 경우 유효한 자동차 개체가 아님을 기록한다.

아래 코드 블록에서 볼 수 있듯이 자동차 객체를 사용하여 테스트할 수 있다.

```ts
const myCar = {
  brand: "Toyota",
  model: "Camry",
  year: 2020,
};

inspectVehicle(myCar);
```

또는 여기에 표시된 것처럼 자동차가 아닌 물체로 테스트할 수도 있다.

```ts
const myBike = { type: "Mountain Bike", wheels: 2 };
inspectVehicle(myBike);
```

## 출처

- [How to use type guards in TypeScript](https://blog.logrocket.com/how-to-use-type-guards-typescript/)
