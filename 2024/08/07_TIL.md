# React - Suspense의 대수적 효과에 의한 동작 원리(with. 데이터 패칭)

## 정리
- React에서 선언적 UI를 따르는 대표적인 컴포넌트인 `Suspense`의 구동 원리를 `대수적 효과`라는 이론을 기반으로 설명한다.
- Sebastian Markbåge는 대수적 효과를 구현한 비동기 로직을 보여주며, Dan Abramov는 대수적 효과를 토대로 Suspense가 만들어졌다고 언급한다.
- Sebastian Markbåge는 자바스크립트에서 패칭과 관련된 로직을 `runPureTask`이라는 함수를 만들어 비동기 제어를 `선언적`으로 구현한다.
- Dan Abramov는 React에선 ES2025의 문법을 이용하여 알려주고 있으며, `try-handle` 구문을 이용하여 대수적 효과를 직관적으로 소개한다.
- React에서 Suspense와 대수적 효과는 다음과 같은 결론을 내린다.
    1. 대수적 효과는 감싸고 있는 함수의 로직이 감싸진(내포하는) 함수의 역할을 분리할 때 실현된다.
    2. React의 Suspense는 자식 컴포넌트를 감싸는 부모 컴포넌트에게 로딩 UI 표시라는 역할을 분리하고 있다.
    3. React의 Suspense의 창안 원리는 대수적 효과이다.

### 참고 
- 주요 키워드: Suspense, Promise, feteching, 대수적 효과, 선언적 UI, 비동기
- 관련 기술: React

## 무엇을 알았는지

### Suspense for Data Fetching 컨셉 이해(with. Sebastian Markbåge)

```js
export function fetchProfileData() {
  let userPromise = fetchUser(); // 프로미스를 리턴
  let postsPromise = fetchPosts();
  return {
    user: wrapPromise(userPromise),
    posts: wrapPromise(postsPromise),
  };
}
​
function wrapPromise(promise) {
  let status = 'pending'; // 최초의 상태
  let result;
​
  // 프로미스 객체 자체
  let suspender = promise.then(
    (r) => {
      status = 'success'; // 성공으로 완결시 success로
      result = r;
    },
    (e) => {
      status = 'error'; // 실패로 완결시 error로
      result = e;
    }
  );
​
  // 위의 Suspense For Data Fetching 예제에서의 read() 메소드입니다.
  // 위 함수의 로직을 클로저삼아, 함수 밖에서 프로미스의 진행 상황을 읽는 인터페이스가 된다
  return {
    read() {
      if (status === 'pending') {
        throw suspender; // 펜딩 프로미스를 throw 하면 Suspense의 Fallback UI를 보여준다
      } else if (status === 'error') {
        throw result; // Error을 throw하는 경우 ErrorBoundary의 Fallback UI를 보여준다
      } else if (status === 'success') {
        return result; // 결과값을 리턴하는 경우 성공 UI를 보여준다
      }
    },
  };
}
```

- API 호출이 존재하는 컴포넌트는 렌더링이 시도될때 마다 `read()`를 통해 결과값을 읽으려는 시도를하고, 결과값이 `throw된 Error`나 `pending 상태의 Promise`, `정상적인 결과값`이냐에 따라 어떤 UI를 보여줄지 달라진다.
- 이경우 상위 Suspense, Errorboundary의 fallback UI를 찾아서 보여주게 된다.
- 비동기 요청을 하는 컴포넌트는 `read()` 메소드가 `리턴`하거나 `throw`하는 값들을 통해 Supense, ErrorBoundary 컴포넌트와 상호작용하고 있음을 알 수 있다.

### 특정 컴포넌트 비동기 로직의 상태값을 계속해서 watching하여 매번 렌더링 시킬수 있는 이유

```js
// 실제로 React의 Suspense가 이것과 똑같이 동작하지는 않겠지만
// 구현 컨셉을 잘 드러내고 있는 코드 조각입니다.
​
let cache = new Map();
let pending = new Map();
​
function fetchTextSync(url) {
  if (cache.has(url)) {
    return cache.get(url); // 캐시 맵객체
  }
  if (pending.has(url)) {
    throw pending.get(url); // Pending Promise throw
  }
  // 비동기 로직
  let promise = fetch(url)
    .then((response) => response.text()) // 처리되는 경우
    .then((text) => {
      pending.delete(url);
      cache.set(url, text);
    });
  pending.set(url, promise); // 팬딩 객체에 팬딩인거 표시
  throw promise;
}
​
async function runPureTask(task) {
  for (;;) {
    // while true
    //!!! 태스크를 리턴할 수 있을 때까지 바쁜대기를 함(무한루프) !!!
    try {
      return task(); // 태스크 값을 리턴할 수 있게 되면 무한루프에서 벗어난다
    } catch (x) {
      // throw를 거른다
      if (x instanceof Promise) {
        await x; // pending promise가 throw된 경우 await으로 resolve 시도 => suspense
      } else {
        throw x; // Error가 throw된 경우 그대로 error throw => ErrorBoundary, 종료
      }
    }
  }
}

function getUserName(id) {
  var user = JSON.parse(fetchTextSync('/users/' + id)); // 비동기 로직
  return user.name;
}
​
function getGreeting(name) {
  if (name === 'Seb') {
    return 'Hey';
  }
  return fetchTextSync('/greeting'); // 비동기 로직
}
​
function getMessage() {
  let name = getUserName(123);
  return getGreeting(name) + ', ' + name + '!';
}
​
runPureTask(getMessage).then((message) => console.log(message));
```

- `wrapPromise`와 같이 비동기 상태를 Promise를 이용하여 success, fail, pending 상태를 직접 정의하지 않고, 클로저로 사용되는 `cache, pending` 객체에 상태를 부여하여 비동기 처리시 성공 여부에 따라, `set`과 `delete`를 하여 현재 비동기의 `상태`를 부여하고, 어떤 비동기 요청인지 `캐시`를 설정한다.
- 이때 비동기 상태의 성공 유무에 따라 기다려주는 함수가 `runPureTask` 함수이며, React에서는 컴포넌트 데이터 요청 상태를 계속해서 확인하고 렌더링을 시도하는 로직이라고 이해하면 된다.
- React에선 **"데이터가 계속 흘러들어옴에 따라 React는 렌더링을 다시 시도하며, 그때마다 React가 더 깊은 곳까지 데이터를 처리할 수 있게 된다"** 라고 설명한다.
- 이렇게 응답을 계속 흘러들어오게 처리한다면?
    - 컨텐츠를 더 일찍 표현할 수 있다. 
    - 응답을 기대하면서 명령적인 예외나 후처리가 불필요하다. 
    - 응답이 올때 명령적으로 상태관리를하여 렌더링 할 필요도 없다.(state or redux store 등)
- 명령형 구조와 로직이 빠지면서 비동기 데이터 표시는 더 빨라지고 로직도 줄어든다.


### `무엇(what - try문)`과 `어떻게(how - handle문)`를 우아하게 분리하는 방법. (추상화와 응집도 & with. Dan Abramov)

```js
// ES2025문법인 try-handle문을 기반으로 작성함
function getName(user) {
  let name = user.name;
  if (name === null) {
    // 2. 여기서 효과를 수행
    name = perform 'ask_name';
    // 5. 그리고 여기로 돌아옴, name값은 핸들 블락에서 넣은 Arya Stark
  }
  // 6. 마지막으로 값을 리턴함
  return name;
}
​
function makeFriends(user1, user2) {
  user1.friendNames.add(getName(user2));
  user2.friendNames.add(getName(user1));
}
​
const arya = { name: null };
const gendry = { name: 'Gendry' };
  try {
    // 1. 함수 실행(try-handle 문에서 먼저 실행)
    makeFriends(arya, gendry);
  } handle (effect) {
    // 3. 효과를 수행하면 Handle 구문으로
    if (effect === 'ask_name') {
      // 4. try, catch와는 다르게 값을 전달하면서 기존 try문 내부의 코드를 이어서 실행
      resume with 'Arya Stark';
    }
  }
```

- 예제의 대수적 효과 문법인 `try-handle` 블럭은 try-catch와 다르게 Exception을 던지고 블럭을 나가는 대신에 _handle문에 명시된 특정 효과를 수행하고 로직을 계속 이어나가게 된다_. `resume` 키워드는 효과가 수행된 곳으로 다시 돌아갈 수 있고, 핸들러를 통해 무언가 전달을 할 수도 있다.
- Dan은 대수적 효과가 **"무엇(what - try문)과 어떻게(how - handle문)를 분리할 수 있게 해주는 강력한 도구가 될 수 있다"**


### 출처
- [Suspense for Data Fetching의 작동 원리와 컨셉 (feat.대수적 효과)](https://maxkim-j.github.io/posts/suspense-argibraic-effect/)
- [토스ㅣSLASH 21 - 프론트엔드 웹 서비스에서 우아하게 비동기 처리하기](https://www.youtube.com/watch?v=FvRtoViujGg)
- [Sebastian Markbåge - SynchronousAsync.js](https://gist.github.com/sebmarkbage/2c7acb6210266045050632ea611aebee)
- [Algebraic Effects for the Rest of Us](https://overreacted.io/algebraic-effects-for-the-rest-of-us/#how-is-all-of-this-relevant-to-react)