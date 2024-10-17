# CJS와 ESM의 Tree-shaking, 왜 ESM일까?

## Tree-shaking의 역사

- 코드 최적화시 적용되는 "쓸모없는 코드 제거"의 기술이다.
- **dead-code 제거**: 실행되지 않는 코드 제거.

  > ex) 조건문에서 항상 false 분기, 할당 후 사용되지 않는 코드 등

  - 컴파일러 수준에서 코드 최적화가 이루어지는 경우가 많다(상대적으로 정적 언어[C, C++, Java]에서 사용된다.)

- Tree-shaking: **모듈 수준**에서 **모듈 전체를 대상**으로 사용하지 않는 코드 제거
  - 전체 번들 크기 감소, 더큰 영향, 동적 언어[Javascript, Python]에서 사용된다.
- 동적 언어에서의 dead-code 제거는 정적 언어보다 어렵다.
  > 프로그램의 모든 가능한 실행 흐름은 함수 호출 트리로 표현할 수 있고, 호출되지 않는 함수를 제거할 수 있다.
  > 그 후 Javascript 모듈 번들러에서 널리 사용된다. Rich Harris(Rollup)의 설명에 따르면 Dead code 제거와 Tree-shaking은 같은 목표를 갖고 있어도 둘은 서로 다르다고 한다.

![](https://so-so.dev/static/ea01bcb9302b5df1f7e9c356a84797f6/6af66/dead-code-elimination-vs-tree-shaking.png)

## 정적 언어와 동적 언어

- 정적 언어

  - 컴파일러가 프로그램 코드를 실행하기 전, 코드 전체를 분석하고 미리 오류를 찾거나 최적화를 진행
  - 대부분의 정보가 미리 확정
  - **정적 분석을 통해 코드가 실행되지 않거나 사용되지 않는지를 시행 전에 분석하여 실행할 수 있도록 dead code를 식별하고 제거하는 것이 가능함**

- 동적 언어
  - 런타임 시점에서 변수의 타입이나 코드의 흐름이 결정된다.
  - 변수 타입이 실행 중에 바뀔 수 있고, 코드의 일부를 실행 중에 새로 생성해서 사용할 수도 있다.
  - **JS와 같은 모듈 기반 언어는 의존성 그래프를 통해 사용되지 않는 모듈을 제거하고 실제 필요한 모듈만 사용할 수 있다.**

## ESM 시스템 파헤치기

ESM 시스템은 **구성, 인스턴스화, 평가** 세 단계로 이루어진다.

![](https://hacks.mozilla.org/wp-content/uploads/2018/03/07_3_phases.png)

### 1. 구성

가장 첫 단계, 로드해야하는 모듈을 파악하기 위해 종속성 트리를 구성
_그래프의 시작점(entry)이 될 파일을 명시하고, 시작점에서 import문을 따라가면서 종속성 트리를 생성._

import로 연결된 파일 자체는 브라우저에서 사용할 수 없다. 따라서 **Module Record**(export, import 정보가 담긴 데이터) 구조로 변환한다. 이 과정에서, 모든 파일을 찾아 로드하고, Module Record로 변환하기 위해 구문분석(Parsing)을 수행한다.

![](https://hacks.mozilla.org/wp-content/uploads/2018/03/04_import_graph.png)
[import로 연결된 파일들을 종속성 트리 구성]
![](https://hacks.mozilla.org/wp-content/uploads/2018/03/05_module_record.png)
[Module Record 구문 분석]

### 2. 인스턴스화

Module Record를 **모듈 인스턴스(Module Instance)**로 변환
모듈 인스턴스는 code와 state를 결합한 구조다.

- code: 그 자체로 아무것도 할 수 없는 사용할 값이 필요한 것
- state: 특정 시점의 변수 실제 값. 메모리이다.

import 할 모든 값을 메모리에 할당할 공간을 찾는 과정이며, export/import 모듀 해당 메모리를 가리킨다.

![](https://hacks.mozilla.org/wp-content/uploads/2018/03/06_module_instance.png)
[Module Instance(code + state)]

### 3. 평가

코드를 실행하여 변수의 **실제 값으로 메모리를 채우는 과정**. 각 단계는 개별적, 비동기적으로 수행된다.

ESM이 import, export를 통해 파싱하고 메모리를 할당하여 사용한다. 이는 CJS에서는 수행하지 않을까?

## CJS와 ESM의 동작 방식 차이

- CJS는 동적 구조이며 동기적으로 동작한다.
- require 함수를 사용해서 모듈을 가져오고, module.export를 사용하여 모듈을 내보낸다. require 함수는 어느 위치든 호출한다.
- 그 순간에 모듈을 가져오는 방식, 그리고 해당 모듈이 로드될 때까지 코드 실행은 중단된다.

### 특징 1. 정적 구조

```js
var foo = "foo";
var lib = require(`lib/${foo}`);
lib.someFunc(); // property lookup
```

위 코드를 보면 lib.someFunc를 통해 lib에 접근시, lib는 동적 값이기 때문에 property lookup을 수행해야 한다.

```js
import * as lib from "lib";
lib.someFunc(); // 정적분석 가능
```

ESM은 이와 반대로 lib를 가져올 때, lib 정보를 정적으로 파악할 수 있기 때문에 접근을 최적화할 수 있다.

```js
function foo() {
  export default "bar"; // SyntaxError
}
foo();
```

```js
// SyntaxError
import foo from `lib/${foo}`;
```

CJS와 달리 export 문은 반드시 ESM 최상위 레벨에 위치해야하고, JS 표준으로 브라우저 환경에 비동기 로드를

지원하기 때문에 브라우저에서 모듈을 로드할 때 페이지 로딩 속도를 저하시키지 않는다. 이 부분만 봐도 ESM이 브라우저에 최적화되어 있다.

### 특징2. Bindings, Not Values

CJS는 require로 로드한 모듈의 값을 그 순간에 불러와 사용한다. 하지만 이는 같은 메모리를 바라보지 않아서 export 된 쪽에서 값을 변경해도 require 쪽에서는 변경된 값으로 상요할 수 없다.(**Value 초점**)

![](https://so-so.dev/static/afbb2014388f6a164b8de720acdeed0b/6af66/cjs_binding.png)
![](https://so-so.dev/static/e1179d71ed49c62181ccc2807877f6e5/6af66/circular_reference.png)

반면 ESM은 import / export 모두 같은 메모리 주소를 바라보고있어 변경한 값을 바로 반영할 수 있다.(**Binding 초점**) -> Tree-Shaking이 수월한 이유.

![](https://so-so.dev/static/7cb97d0fb704db067b77ab735c3e8097/6af66/esm_binding.png)
![](https://so-so.dev/static/51a3104db32dbfa8921fd3dde98b3771/6af66/esm_binding_update.png)

## 결론

Rollup 번들러를 기준으로 ESM을 선택하게된 이유를 알게되었다. ESM이 Tree-Shaking에 수월한 점. 브라우저 최적화가 되었다는 점.

## 출처

- [CommonJS와 ES Modules의 Tree-shaking, 왜 ES Modules인가?](https://nami-socket.tistory.com/m/50)
- [Tree Shaking과 Module System](https://so-so.dev/web/tree-shaking-module-system/)
- [ES modules: A cartoon deep-dive](https://hacks.mozilla.org/2018/03/es-modules-a-cartoon-deep-dive/)
