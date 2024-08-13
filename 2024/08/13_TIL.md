# AHA 테스팅(Avoid Hasty Abstraction) - 성급한 추상화 피하기

## 정리

- 테스트의 데이터나 반환 받아야하는 인터페이스가 많이 있을 경우 추상화하는 것을 추천하지만, 적은 갯수의 테스트와 에러와 같은 상태 테스트를 하지 않은 이상 ANA 방향으로 가는 것을 추천한다.
- DRY를 너무 신경쓰게 되면 불필요한 추상화로인해 가독성과 유지 보수성이 떨어지게된다.
- 테스팅 단계시 추상화를 너무 하는 것과 하지 않음의 경계를 잘 구분지어서 작성해야 좋은 추상화이다. 적절히 AHA를 적용하는 것이 좋은 테스트 코딩이다.

### 참고

- 주요 키워드: test, ANA, AHA, DRY
- 관련 기술: test, node.js, react

## 무엇을 알았는지

### 테스트 스펙트럼

![](https://miro.medium.com/v2/resize:fit:640/format:webp/1*AitEFu2ZkwMuv1JTVBWZiQ.png)

### ANA(Absolutely No Abstraction, 추상화하지 않음) 테스팅

[AHA 프로그래밍 원칙](https://kentcdodds.com/blog/aha-programming), 성급한 추상화 피하라

대표적 예시로 ExpressJS(라우트 핸들러 테스트)

```js
import * as blogPostController from "../blog-post";

// 애플리케이션 전체에서 사용할 데이터베이스 모의 객체를 불러옵니다.
// 이렇다면 AANA (Almost Absolutely No Abstraction, 거의 절대적으로 추상화하지 않음)라는 의미겠죠?
// 하지만 이 글을 위해 전체 DB 모의를 코드를 작성하고 싶진 않았어요 😅
jest.mock("../../lib/db");

test("로그인한 사용자의 블로그 게시글 목록을 가져옵니다", async () => {
  const req = {
    locale: {
      source: "default",
      language: "en",
      region: "GB",
    },
    user: {
      guid: "0336397b-e29d-4b63-b94d-7e68a6fa3747",
      isActive: false,
      picture: "http://placehold.it/32x32",
      age: 30,
      name: {
        first: "Francine",
        last: "Oconnor",
      },
      company: "ACME",
      email: "francine.oconnor@ac.me",
      latitude: 51.507351,
      longitude: -0.127758,
      favoriteFruit: "banana",
    },
    body: {},
    cookies: {},
    query: {},
    params: {
      bucket: "photography",
    },
    header(name) {
      return {
        Authorization: "Bearer TEST_TOKEN",
      }[name];
    },
  };
  const res = { ...};
  const next = jest.fn();

  await blogPostController.loadBlogPosts(req, res, next);

  expect(res.json).toHaveBeenCalledTimes(1);
  expect(res.json).toHaveBeenCalledWith({
    posts: expect.arrayContaining([
      expect.objectContaining({
        title: "Test Post 1",
        subtitle: "This is the subtitle of Test Post 1",
        body: "The is the body of Test Post 1",
      }),
    ]),
  });
});

test("게시글이 없으면 빈 리스트를 반환합니다.", async () => {
  const req = {
    locale: {
      source: "default",
      language: "en",
      region: "GB",
    },
    user: {
      guid: "0336397b-e29d-4b63-b94d-7e68a6fa3747",
      isActive: false,
      picture: "http://placehold.it/32x32",
      age: 30,
      name: {
        first: "Francine",
        last: "Oconnor",
      },
      company: "ACME",
      email: "francine.oconnor@ac.me",
      latitude: 31.230416,
      longitude: 121.473701,
      favoriteFruit: "banana",
    },
    body: {},
    cookies: {},
    query: {},
    params: {
      bucket: "photography",
    },
    header(name) {
      return {
        Authorization: "Bearer TEST_TOKEN",
      }[name];
    },
  };
  const res = { ... };
  const next = jest.fn();

  await blogPostController.loadBlogPosts(req, res, next);

  expect(res.json).toHaveBeenCalledTimes(1);
  expect(res.json).toHaveBeenCalledWith({
    posts: [],
  });
});
```

위와 같이 `추상화를 전혀 하지 않을 경우` 테스트를 이해하고 유지 보수하는 부분이 불필요하게 어렵게 만들어진다.

이와 같은 경우가 발생하는 과정은 보통 아래와 예시와 같다.

---

1. 엔지니어 Joe가 팀에 합류합니다.
2. Joe는 테스트를 추가해야 합니다.
3. Joe는 자신의 필요에 맞는 것처럼 보이는 이전 테스트를 복사하고 자신의 사용 사례에 맞게 수정합니다. 리뷰어들은 테스트가 통과하는 것을 보고 Joe가 무엇을 하고 있는지 알고 있다고 가정합니다.
4. PR이 머지됩니다.

---

> 두 개의 유사한 테스트 단언(assertion)의 차이점과 그 차이를 일으키는 원인을 얼마나 쉽게 파악할 수 있는가?

### DRY(Don’t Repeat Yourself) 테스팅

DRY 원칙을 적용한다고해서 꼭 더 나은 선택은 아니다. 오히려 유지보수가 더 어려워지는 패턴이 발생하게 된다.

이와 같은 경우가 발생하는 과정은 보통 아래와 예시와 같다.

---

1. 엔지니어 Joe가 팀에 합류합니다.
2. Joe는 테스트를 추가해야 합니다.
3. Joe는 자신의 필요에 맞는 것처럼 보이는 이전 테스트를 복사하고 공통 테스트 유틸리티에 자신의 사례에 맞는 if문을 추가합니다.
4. 리뷰어들은 테스트가 통과하는 것을 보고 Joe가 무엇을 하고 있는지 알고 있다고 가정합니다.
5. PR이 머지됩니다.

---

또한 DRY에서 자주 보이는 패턴은 `describe`와 `it`의 중첩을 과하게 사용하고, 그 요소에 각각 `beforeEach`로 공용 변수를 초기화하는 것.

중첩된 구조와 공용 변수를 많이 사용할 수록 로직을 따라가기가 더 어려워진다.

참고) [React에서 테스트 격리하기](https://kentcdodds.com/blog/test-isolation-with-react)

### AHA(Avoid Hasty Abstraction) 테스팅

추상화의 필요성이 중요하고 이것이 AHA 프로그래밍의 기본 원칙이다.

```js
import * as blogPostController from "../blog-post";

// 애플리케이션 전체에서 사용할 데이터베이스 모의 객체를 불러옵니다.
jest.mock("../../lib/db");

function setup(overrides = {}) {
  const req = {
    locale: {
      source: "default",
      language: "en",
      region: "GB",
    },
    user: {
      guid: "0336397b-e29d-4b63-b94d-7e68a6fa3747",
      isActive: false,
      picture: "http://placehold.it/32x32",
      age: 30,
      name: {
        first: "Francine",
        last: "Oconnor",
      },
      company: "ACME",
      email: "francine.oconnor@ac.me",
      latitude: 51.507351,
      longitude: -0.127758,
      favoriteFruit: "banana",
    },
    body: {},
    cookies: {},
    query: {},
    params: {
      bucket: "photography",
    },
    header(name) {
      return {
        Authorization: "Bearer TEST_TOKEN",
      }[name];
    },
    ...overrides,
  };

  const res = { ... };
  const next = jest.fn();

  return { req, res, next };
}

test("로그인한 사용자의 블로그 게시글 목록을 가져옵니다", async () => {
  const { req, res, next } = setup();

  await blogPostController.loadBlogPosts(req, res, next);

  expect(res.json).toHaveBeenCalledTimes(1);
  expect(res.json).toHaveBeenCalledWith({
    posts: expect.arrayContaining([
      expect.objectContaining({
        title: "Test Post 1",
        subtitle: "This is the subtitle of Test Post 1",
        body: "The is the body of Test Post 1",
      }),
    ]),
  });
});

test("게시글이 없으면 빈 리스트를 반환합니다", async () => {
  const { req, res, next } = setup();
  req.user.latitude = 31.230416;
  req.user.longitude = 121.473701;

  await blogPostController.loadBlogPosts(req, res, next);

  expect(res.json).toHaveBeenCalledTimes(1);
  expect(res.json).toHaveBeenCalledWith({
    posts: [],
  });
});
```

위 코드를 확인해보면, 두번째 테스트 케이스인 "게시글이 없으면 빈 리스트를 반환합니다"를 확인해보면 "위치 기반 블로깅 서비스"라는 도메인을 파악할 수 있다.

이렇듯 DB 정보에 대한 추상화를 `setup`(`Test Object Factory`라고 불리는 구조)이라는 함수로 추상화시켜 필요한 정보를 테스트 케이스에서 뽑아서 사용하니 더욱 빠른 가독성과 유지 보수성이 추가되었다.

### React에서 AHA 테스팅

```js
import * as React from "react";
import { render, screen } from "@testing-library/react";
import userEvent from "@testing-library/user-event";
import LoginForm from "../login-form";

function renderLoginForm(props) {
  render(<LoginForm {...props} />);
  const usernameInput = screen.getByLabelText(/username/i);
  const passwordInput = screen.getByLabelText(/password/i);
  const submitButton = screen.getByText(/submit/i);
  return {
    usernameInput,
    passwordInput,
    submitButton,
    changeUsername: (value) => userEvent.type(usernameInput, value),
    changePassword: (value) => userEvent.type(passwordInput, value),
    submitForm: () => userEvent.click(submitButton),
  };
}

test("submit하면 submit handlers를 호출해야 합니다", () => {
  const handleSubmit = jest.fn();
  const { changeUsername, changePassword, submitForm } = renderLoginForm({
    onSubmit: handleSubmit,
  });
  const username = "chucknorris";
  const password = "ineednopassword";
  changeUsername(username);
  changePassword(password);
  submitForm();
  expect(handleSubmit).toHaveBeenCalledTimes(1);
  expect(handleSubmit).toHaveBeenCalledWith({ username, password });
});
```

컴포넌트 렌더링이 필요한 React에선 `renderLoginForm` 함수와 같이 기본적인 컴포넌트 렌더링과 유저 인터페이스를 지정하는 변수와 메서드를 반환하는 함수를 제공하면 재사용성과 유지 보수성이 매우 뛰어나진다.

주의: 만약 갯수가 적은 테스트 작성시 추상화는 불필요한 추상화이지만 에러 상태와 같은 세부 사항 테스트가 필요하다면 여전히 추상화가 유리하다.

### 출처

- [(번역) AHA 테스팅 💡](https://medium.com/@jiwoochoics/%EB%B2%88%EC%97%AD-aha-%ED%85%8C%EC%8A%A4%ED%8C%85-405c9e4b4c7c)
