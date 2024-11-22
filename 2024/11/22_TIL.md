# SWC를 사용해야하는 이유(Babel을 쓰지 않고)

## Babel과 SWC는 어떻게 다를까?

[Babel](https://babeljs.io/docs/en/)과 [SWC](https://swc.rs/)는 최신 버전의 자바스크립트/타입스크립트 코드를 이전 버전과 호환되는 자바스크립트 코드로 변환하는 두 가지 [트랜스파일러](https://blog.logrocket.com/why-you-should-use-swc/%E2%80%9Dhttps://www.digitalocean.com/community/tutorials/javascript-transpilers-what-they-are-why-we-need-them%E2%80%9D)다.

몇 가지 차이점는 다음과 같다.

- 속도: SWC는 Rust 백엔드를 활용하기 때문에 Babel보다 빠르다.
- 커뮤니티: Babel은 웹 개발 분야에서 훨씬 더 오래 사용되어 왔기 때문에 SWC에 비해 더 큰 커뮤니티를 보유하고 있다. 하지만 깃허브에서 3만 1천 개가 넘는 별을 보유한 것을 보면 SWC가 나날이 인기를 얻고 있다고 해도 과언이 아니다. 따라서 개발자가 도움을 구하기에 충분한 리소스를 확보할 수 있다.
- 사용 편의성: 명확하고 간결한 문서 덕분에 SWC는 Babel보다 설정과 사용이 훨씬 쉽다.

이제 Babel이 내부에서 어떻게 작동하는지 살펴보자. Babel은 사용자가 정의한 설정을 기반으로 소스 코드를 읽고 [화살표 함수](https://developer.mozilla.org/ko/docs/Web/JavaScript/Reference/Functions/Arrow_functions)나 [옵셔널 체이닝](https://developer.mozilla.org/ko/docs/Web/JavaScript/Reference/Operators/Optional_chaining)과 같은 최신 JavaScript 기능을 컴파일한다. 이 과정은 Babel의 세 가지 주요 도구를 사용하여 수행된다.

- 먼저, Babel의 [파서](https://github.com/babel/babel/tree/master/packages/babel-parser)가 자바스크립트 코드를 가져와서 컴퓨터가 이해할 수 있는 소스 코드의 구조인 [AST(추상 구문 트리)](https://en.wikipedia.org/wiki/Abstract_syntax_tree)로 변환한다.
- 그런 다음, Babel의 [트래버서](https://github.com/babel/babel/tree/master/packages/babel-traverse)가 AST를 가져와서 탐색하고 Babel 구성에서 정의한 의도한 코드로 수정한다.
- 마지막으로, Babel의 [제너레이터](https://github.com/babel/babel/tree/master/packages/babel-generator)는 수정된 AST를 다시 일반 코드로 변환한다.

![](https://blog.logrocket.com/wp-content/uploads/2024/11/javascript-abstract-syntax-tree.png)

## Babel의 대체제로 SWC를 사용해야 하는 이유

인터넷 프로젝트에서 "JavaScript + SWC" 또는 "TypeScript + SWC"라는 문구를 본 적이 있을 건데, 이 글에서는 SWC가 정확히 무엇인지, 그리고 왜 SWC로 전환해야 하는지 알아보자. SWC는 JavaScript를 위한 대체 트랜스파일러다.

많은 기업에서 코드의 다양한 부분을 SWC로 재작성하고 있다. 예를들어

- [Firefox](https://www.mozilla.org/en-US/)는 퀀텀 CSS라는 CSS 렌더러를 재작성하기로 결정하고 [상당한 성능 향상](https://hacks.mozilla.org/2017/08/inside-a-super-fast-css-engine-quantum-css-aka-stylo/)을 얻었다.
- [Tilde](https://www.tilde.io/)는 자바 HTTP 엔드포인트의 특정 부분을 Rust로 재작성하고 [메모리 사용량을 5GB에서 50MB로 줄여 큰 성능 향상](https://www.rust-lang.org/static/pdfs/Rust-Tilde-Whitepaper.pdf)을 얻었다.

왜 이 회사들이 모두 SWC로 전환했을까?

SWC는 [Rust](https://www.rust-lang.org/)로 작성되어 [Babel보다 훨씬 빠르기](https://swc.rs/blog/perf-swc-vs-babel) 때문. Rust는 성능과 안정성으로 잘 알려져 있으며, 특히 효과적인 가비지 컬렉션으로 인해 성능이 뛰어나다.

[가비지 컬렉션](<https://en.wikipedia.org/wiki/Garbage_collection_(computer_science)>)은 더 이상 사용되지 않는 데이터 객체를 제거하여 리소스를 확보하는 메모리 관리 방식이다. Rust는 컴파일 시점에 더 이상 필요하지 않으면서도 지속적으로 실행할 필요가 없는 메모리 리소스를 결정하기 때문에 해당 리소스를 동적으로 확보하여 처리 시간을 단축하고 성능을 크게 향상시킨다.

SWC 팀에서는 전체 페이지를 할애하여 속도를 자랑하고 있다. 다음은 Babel과 속도를 비교하고 수년간의 성능 향상을 보여주는 차트다.

![](https://blog.logrocket.com/wp-content/uploads/2024/11/swc-vs-babel-performance-comparison.png)

코드 트랜스파일링은 비용이 많이 드는 프로세스일 수 있다. 그렇기 때문에 Rust로 작성된 트랜스파일러를 사용하면 부피가 커지는 것을 간소화하는 데 도움이 된다. 이에 대해 더 자세히 살펴보겠지만, 먼저 트랜스파일러가 필요한지 여부를 결정해야 한다.

## 트랜스파일러가 필요한 이유는 뭘까?

트랜스파일러가 필요하지 않은 경우도 있다. ES3와 같이 잘 지원되는 자바스크립트 버전에 주로 의존하는 간단한 프로젝트를 구축하는 경우 트랜스파일러가 없어도 괜찮을 것이다. 예를 들어 아래 코드는 거의 모든 브라우저에서 작동한다.

```js
function saySomething(something) {
  console.log(`${something}. But don't tell anyone.`);
}

saySomething("I don't like Javascript!");
```

화살표 함수 등 최신 버전의 JavaScript에 의존하는 간단한 프로젝트를 빌드하고 있지만 지원해야 하는 브라우저도 이러한 새로운 기능을 지원하는 경우. 예를 들어, 최신 버전의 Chrome(45 이상)에서 아래 코드를 실행하면 정상적으로 작동한다.

![](https://blog.logrocket.com/wp-content/uploads/2024/11/arrow-function-browser-support.png)

```js
// we print it, but we don't agree with it
const saySomething = (something) => {
  console.log(`${something}. But don't tell anyone.`);
};

saySomething("I don't like Javascript!");
```

이 코드를 SWC 트랜스파일러에 입력하면 다음과 같은 자바스크립트 출력이 표시된다.

```js
var saySomething = function (something) {
  console.log("".concat(something, ". But don't tell anyone."));
};
saySomething("I don't like Javascript!");
```

트랜스파일러는 여러 자바스크립트 인터프리터 버전에서 호환성을 보장하기 위해 `concat` 및 `var`와 같은 키워드와 함수를 사용했다. 실제로 Mozilla의 문서에 따르면 이 코드는 구글 크롬의 첫 번째 버전에서 실행할 수 있다!

이러한 경우를 제외하고도 애플리케이션에는 여전히 트랜스파일러가 필요다. 브라우저는 `V8`(Chrome), `SpiderMonkey`(Firefox), `Chakra`(IE) 등 다양한 유형의 자바스크립트 엔진을 사용한다. 즉, 표준 자바스크립트 사양이 있더라도 브라우저마다 표준이 적용되는 시기와 지원 수준은 매우 다양하다.

따라서 다양한 브라우저에서 자바스크립트 코드를 일관되게 처리해야 한다. 트랜스파일러를 사용하면 코드가 깨지거나 새로운 기능을 사용할 기회를 잃는 것에 대한 스트레스를 줄일 수 있다.

트랜스파일러에 대한 의존도는 [ES6](https://www.w3schools.com/js/js_es6.asp) 또는 TypeScript를 [ES5](https://www.w3schools.com/js/js_es5.asp)로 변환하는 경우에만 국한되지 않다. 트랜스파일러는 자바스크립트의 미래를 가져다주는 동시에 [ES2019](https://blog.logrocket.com/5-es2019-features-you-can-use-today/)와 같은 다양한 자바스크립트 변환 사례를 처리할 수 있게 해준다. 트랜스파일러는 오늘날의 개발자에게 매우 강력한 도구다.

이제 간단한 설정으로 SWC 사용을 테스트한 다음 상대적인 성능과 속도를 Babel과 비교해 보자.

## SWC 사용하기

SWC는 NPM 패키지 관리자에서 패키지로 설치할 수 있다. 먼저 디렉터리 루트에서 이 명령을 실행하여 시작하자.

```sh
# use `cd` to go to the right directory and then run
mkdir swc_project
cd swc_project
# initialize a package.json
npm init

#install swc core as well as its cli tool
npm i -D @swc/core @swc/cli
```

이 작업을 실행하면 이제 [SWC 코어](https://swc.rs/docs/usage/core)와 [CLI](https://swc.rs/docs/usage/cli)를 모두 갖추게 된다. 코어 패키지는 빌드 설정에 도움이 되며, CLI 패키지는 터미널에서 명령어로 실행할 수 있다.

첫 번째 단계로 CLI 도구를 사용하여 JavaScript 파일을 트랜스파일링하는 데 중점을 두자. 디렉터리 루트에 아래와 같은 JavaScript 파일이 있다고 가정해 보자.

```js
// async.js
const axios = require("axios");

async function getData() {
  try {
    const response = await axios.get(
      "https://jsonplaceholder.typicode.com/todos/1"
    );
    console.log(response.data);
  } catch (error) {
    console.error(error);
  }
}
getData();

// result:
// ▶Object {userId: 1, id: 1, title: "delectus aut autem", completed: false}
```

위의 코드 스니펫에서 SWC의 트랜스파일러를 사용해 보자.

```sh
# 이 명령을 실행하면 트랜스파일된 데이터가 stdout으로 출력되며 터미널에 출력된다.
npx swc ./async.js

# 이 명령을 실행하면 'output.js'라는 새 파일이 생성된다.
npx swc ./async.js -o output.js

# 이 명령을 실행하면 `transpiledDir`이라는 새 디렉터리가 생성되고 원본 디렉터리 내의 모든 파일을 트랜스파일한다.
npx swc ./swc_project -d transpiledDir
```

이 [SWC 플레이그라운드](https://swc.rs/playground)를 사용하여 트랜스파일된 파일이 어떻게 보이는지 확인할 수 있다. 트랜스파일된 출력의 일부는 다음과 같다.

![](https://blog.logrocket.com/wp-content/uploads/2024/11/axios-async-function-example.png)

두 번째 단계로, 빌드 시스템에 SWC를 도구로 포함시키고자 한다. 이 단계에서는 [Webpack](https://webpack.js.org/)을 보다 고급의 구성 가능한 빌더로 사용하려고 한다.

우선 Webpack과 SWC를 설정한 `package.json`이 어떻게 보이는지 살펴보자. 이 설정을 사용하면 `npm run-script build`를 실행하여 웹팩이 패키지를 빌드하고 `npm run-script start`를 실행하여 웹팩이 애플리케이션을 서비스를 실행할 수 있다.

```json
{
  "name": "swc_learn",
  "version": "1.0.0",
  "description": "Tutorial project",
  "main": "async.js",
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1"
  },
  "author": "",
  "license": "ISC",
  "dependencies": {
    "axios": "^1.7.7",
    "react": "^18.3.1",
    "react-dom": "^18.3.1",
    "regenerator-runtime": "^0.14.1"
  },
  "devDependencies": {
    "@swc/cli": "^0.4.1-nightly.20240914",
    "@swc/core": "^1.7.39",
    "css-loader": "^7.1.2",
    "html-loader": "^5.1.0",
    "html-webpack-plugin": "^5.6.3",
    "sass": "^1.80.4",
    "sass-loader": "^16.0.2",
    "style-loader": "^4.0.0",
    "swc-loader": "^0.2.6",
    "webpack": "^5.95.0",
    "webpack-cli": "^5.1.4"
  }
}
```

애플리케이션을 빌드하고 시작하기 위한 위의 구성은 [webpack.config.js](https://webpack.js.org/configuration/) 파일에 저장되며 Webpack에서 자동으로 가져온다. 이 파일에는 아래와 같은 작업을 진행한다.

- [**output**](https://webpack.js.org/configuration/output/): 모든 트랜스파일된 파일을 포함한 번들, 에셋, 파일을 출력할 Webpack의 이름과 위치를 설정한다.
- [**devServer**](https://webpack.js.org/configuration/dev-server/): 이 구성을 통해 웹팩 앱에 콘텐츠를 제공할 위치를 웹팩에 알려주고 요청을 수신할 포트를 정의하여 웹팩 앱을 서비스한다.
- [**HTMLWebpackPlugin**](https://webpack.js.org/plugins/html-webpack-plugin/): 이 플러그인을 정의하여 웹팩 번들이 포함된 HTML 파일을 더 쉽게 서비스할 수 있도록 한다.

하지만 이 구성에서 가장 중요한 부분은 `.js` 또는 `.jsx` 파일 확장자를 가진 JavaScript 파일을 트랜스파일할 수 있는 [`swc-loader`](https://swc.rs/docs/usage/swc-loader)다.

```js
// global dependencies
//filename: webpack.config.js
const path = require("path");
const HTMLWebpackPlugin = require("html-webpack-plugin");

module.exports = {
  mode: "development",
  output: {
    path: path.resolve(__dirname, "./dist"),
    filename: "index_bundle.js",
  },
  devServer: {
    contentBase: path.join(__dirname, "dist"),
    compress: true,
    port: 9000,
  },
  module: {
    rules: [
      {
        test: /\.jsx?$/,
        exclude: /(node_modules|bower_components)/,
        use: {
          // `.swcrc` in the root can be used to configure swc
          loader: "swc-loader",
        },
      },
      {
        test: /\.html$/,
        use: [
          {
            loader: "html-loader",
            options: { minimize: true },
          },
        ],
      },
      {
        test: /\.scss/i,
        use: ["style-loader", "css-loader", "sass-loader"],
      },
    ],
  },
  plugins: [
    new HTMLWebpackPlugin({
      filename: "./index.html",
      template: path.join(__dirname, "public/index.html"),
    }),
  ],
};
```

웹팩 구성에 `swc-loader`를 설정했으므로 자바스크립트 파일을 성공적으로 트랜스파일링하는 데 절반은 성공했다. 하지만 파일을 정확히 어떻게 트랜스파일링할지 SWC에 지시해야 한다. 알고 보니 SWC는 여기에서도 Babel과 비슷한 접근 방식을 사용한다. 이 프로세스를 수행하려면 루트 디렉터리에 [.swcrc](https://swc.rs/docs/configuration/swcrc)라는 구성 파일을 만들어야 한다. 타입스크립트를 트랜스파일하려는 프로젝트에서 이 구성이 어떻게 보이는지 살펴보자.

이 config에서는 파일 확장자가 `.ts`인 파일과만 일치하도록 `test` config을 [정규식](https://en.wikipedia.org/wiki/Regular_expression)으로 사용하고 있다. 또한 `jsx.parser` 구성을 사용하여 SWC에 트랜스파일에 사용할 파서(`typescript/ecmascript`일 수 있음)를 지시하고 있다.

그러나 사용 사례에 어떤 번역 옵션을 사용할지 정의하여 syntax 파싱을 더 잘 제어할 수 있다. 예를 들어, 이 예제에서는 타입스크립트 [decorators](https://www.typescriptlang.org/docs/handbook/decorators.html)와 [dynamic import](https://basarat.gitbook.io/typescript/project/dynamic-import-expressions) 트랜스파일링에 관심이 있지만 파일 확장자가 `.tsx`인 파일은 트랜스파일링을 무시한다.

```json
// .swcrc
{
  "$schema": "https://swc.rs/schema.json",
  "test": ".*.ts$",
  "jsc": {
    "parser": {
      "syntax": "typescript",
      "tsx": false,
      "decorators": true,
      "dynamicImport": true
    }
  }
}
```

이제 위의 웹팩 SWC 예제에서 React를 사용하자. React는 `.jsx`라는 특정 파일 확장자를 사용하여 React 컴포넌트를 작성할 수 있다.

```js
// App.jsx
import React from "react";
import ReactDOM from "react-dom";

const App = () => {
  return <h1>My SWC App</h1>;
};

ReactDOM.render(<App />, document.querySelector("#root"));
```

웹팩을 통해 이 파일을 서비스하려면 위에서 정의한 올바른 `webpack loader`가 필요하다. 또한 `.swcrc` 파일에 올바른 트랜스파일 설정이 필요하다. 이제 이 접근 방식을 통해 최신 자바스크립트([ES2024](https://selvaganesh93.medium.com/javascript-whats-new-in-ecmascript-2019-es2019-es10-35210c6e7f4b))의 최신 기능을 사용할 뿐만 아니라 트랜스파일링 시 `.jsx` 파일도 지원한다. 또한 React 프로젝트에 추가 트랜스파일링 설정이 필요한 경우 [다양한 설정](https://swc.rs/docs/configuration/compilation)을 사용할 수 있다.

```json
// .swcrc
{
  "jsc": {
    "parser": {
      "syntax": "ecmascript",
      "jsx": true
    }
  }
}
```

## Babel과 SWC의 속도 비교

트랜스파일러의 속도는 빌드 프로세스에 구워지기 때문에 매우 중요하다. 물론 많은 개발자는 이 영역에서 절약할 수 있는 시간을 소중하게 생각한다. 속도 측면에서 두 도구가 어떻게 비교되는지 살펴보자.

먼저 두 트랜스파일러를 인위적인 방식으로 비교하자. Babel과 SWC의 코드 변환을 동기식으로 실행해보자. 알다시피 자바스크립트는 [싱글 스레드](https://www.red-gate.com/simple-talk/dotnet/asp-net/javascript-single-threaded/)이기 때문에 무거운 연산을 비동기식으로 실행하는 것은 불가능하다. 모든 것을 고려하더라도 이 방법은 여전히 속도 비교의 지표를 제공한다. 단일 코어 CPU에서 실행된 벤치마크 비교를 테스트해 보자([SWC 프로젝트의 유지 관리자](https://swc.rs/blog/perf-swc-vs-babel)가 수행한 테스트)

| Transform   | Speed (operation/second) | Sample Runs |
| ----------- | ------------------------ | ----------- |
| SWC (ES3)   | 616 ops/sec              | 88          |
| Babel (ES5) | 34.05 ops/sec            | 58          |

이는 SWC를 위한 ES3 변환 프로세스가 더 비싸더라도 Babel에 비해 SWC 변환 속도가 빠르다는 것을 나타내고있다.

이제 보다 현실적인 시나리오를 벤치마킹하려면 자바스크립트에서 연산을 처리하는 더 비싸고 실제적인 시나리오인 [`await Promise.all()`](https://developer.mozilla.org/ko/docs/Web/JavaScript/Reference/Global_Objects/Promise/all)에 대해 샘플을 실행할 수 있다. 이 벤치마크에서는 CPU 코어 수와 병렬 연산이 중요한 역할을 한다. 실행된 다른 벤치마크에서는 두 가지 실험이 수행되었다. 두 실험 모두 **8개의 CPU 코어**와 **4의 병렬 처리 능력**을 갖춘 컴퓨터를 사용했다.

첫 번째 실험은 4개의 프로미스로 실행한다.

| Transform   | Speed (operation/second) | Sample Runs |
| ----------- | ------------------------ | ----------- |
| SWC (ES3)   | 1704 ops/sec             | 73          |
| Babel (ES5) | 27.28 ops/sec            | 40          |

두 번째 실험은 100개의 프로미스로 실행한다.

| Transform   | Speed (operation/second) | Sample Runs |
| ----------- | ------------------------ | ----------- |
| SWC (ES3)   | 2199 ops/sec             | 54          |
| Babel (ES5) | 32 ops/sec               | 6           |

[참고 repository](https://github.com/swc-project/node-swc)

```sh
# 복제하고 복제된 저장소에 CD로 복사
git clone https://github.com/swc-project/node-swc.git
cd node-swc

#Node.js benchmark runner, modeled after Mocha and bencha, based on Benchmark.js.
npm i benchr -g

// /benches 디렉토리에서 multicore.js 또는 기타 벤치마크를 실행
benchr ./benches/multicore.js
```

이 수치에서 확인할 수 있는 가장 중요한 점은 Babel이 [이벤트 루프](https://developer.mozilla.org/ko/docs/Web/JavaScript/Event_loop)에서 작동하기 때문에 비동기 작업에서 성능이 저하된다는 것. 이는 [worker thread](https://nodejs.org/api/worker_threads.html)에서 실행되고 CPU 코어 수에 따라 적절하게 확장할 수 있는 SWC와는 대조적이다.

일반적으로 SWC는 단일 스레드 및 CPU 코어 기준으로 Babel보다 약 20배 빠른 반면, 멀티코어 비동기 작업 프로세스에서는 약 60배 빠르기 때문에 두 도구 간에는 분명한 속도 차이가 있다.

## 결론

- 빌드 워크플로우를 위한 SWC와 Babel의 설정은 비슷히디.
- 그러나 SWC는 Babel에 비해 상당한 속도 이점이 있다.

Babel을 사용 중이고 빌드 시간을 단축하기 위해 SWC로 전환할 생각이라면 반드시 전환하자.

## 출처

- [Why you should use SWC (and not Babel)](https://blog.logrocket.com/why-you-should-use-swc/)
