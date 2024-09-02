# 좋은 리팩토링 vs 나쁜 리팩토링

## 정리

- 좀 더 보편적인 기술을 사용하면서 간결성과 가독성, 유지보수성을 고려하면서 의존성을 도입하자.
- 기존 코드를 이해하지 않고 "객체 지향적"으로 보이기위해 그룹화한다면, 더 복잡성이 올라가고 유지보수성이 올라갈 수 있다.
- 중요한 것은 "코드 베이스의 일관성"이다. 새로운 패턴을 도입할 경우, 일회성 불일치를 만드는 것 보다 먼저 팀의 동의 받도록 하자.
- 리팩토링의 가장 큰 문제는 코드를 이해하기 위한 리팩토링이다. 이건 좋지 않은 방식이고, 특정 코드를 6~9개월 정도 다루는 것을 권장하기도한다. 괜한 버그를 만들거나 성능을 해치지 않으려면.
- 비지니스의 맥락을 이해하면서, 기술을 도입하자, SSR, SSG, SPA의 상황에서 적절히
- 과도한 통합은 지양하자. 기본값을 제공하면서 추상화를 진행한다면, 추상화의 이점을 유지하면서 유연성을 가질수 있다. 코드 통합시 "더 깔끔한" 코드를 위해 유연성을 희생하지 말 것. 말그대로 "개선"에 초점을 가지자.

### 참고

- 주요 키워드: 리팩토링, 추상화, 간경설, 가독성, 유지보수성, 복잡성,
- 관련 기술: 함수형 프로그래밍, 객체지향 프로그래밍, 리팩토링,

## 무엇을 알았는지

리팩토링은 대부분의 경우 새로 리팩토링된 코드는 다른 개발자들이 이해하고 유지보수가 어렵고 느리거나 버그가 많은 경우가 있다. 리팩토링이 나쁘다는 것이 아니라 나쁜 리팩토링이 많다는 것.

더 나은 것을 만들려다 오히려 상황을 악화시키는 경우가 생긴다. 이제 좋은/나쁜 리팩토링의 차이점과 코드 베이스를 통해 알아보자.

### 리팩토링의 좋은점, 나쁜점, 추악한 점

추상화는 좋을 수도, 나쁠 수도 잇는데 핵심은 언제 어떻게 적용하는지가 중요하다.

1.  **코딩 스타일을 크게 바꾸는 것**

    다른 스타일, 배경을 가진 개발자가 리팩토링을 한다면 특정 패러다임에 대해 강한 의견을 가지는 경우가 생기는데 이런 상황을 살펴보자.

    ```tsx
    // 변경전
    function processUsers(users: User[]) {
      const result = [];
      for (let i = 0; i < users.length; i++) {
        if (users[i].age >= 18) {
          const formattedUser = {
            name: users[i].name.toUpperCase(),
            age: users[i].age,
            isAdult: true,
          };
          result.push(formattedUser);
        }
      }
      return result;
    }
    ```

    ```tsx
    // 변경후 - 나쁜 리팩토링
    import * as R from "ramda";

    const processUsers = R.pipe(
      R.filter(R.propSatisfies(R.gte(R.__, 18), "age")),
      R.map(
        R.applySpec({
          name: R.pipe(R.prop("name"), R.toUpper),
          age: R.prop("age"),
          isAdult: R.always(true),
        })
      )
    );
    ```

    위와 같은 리팩토링 된 버전은 함수형 프로그래밍을 좋아하는 사람이 작성한 코드다. 새로운 라이브러리(Ramda)를 도입하여 적용했는데, 이 방법은 익숙하지 않은 개발자에게 유지보수가 어렵다.

    ```tsx
    // 좋은 리팩토링
    function processUsers(users: User[]): FormattedUser[] {
      return users
        .filter((user) => user.age >= 18)
        .map((user) => ({
          name: user.name.toUpperCase(),
          age: user.age,
          isAdult: true,
        }));
    }
    ```

    `filter`와 `map`을 써서 더 보편적인 자바스크립트 메서드를 사용했고, 더 간결하고 가독성이 좋아졌다. 새로운 패러다임이나 외부 의존성도 도입하지 않는 케이스이다.

2.  **불필요한 추상화**

    기본 코드를 이해하지 않고 새로운 추상화를 추가하여 그룹화해선 안되는 것들을 그룹화하여 '객체 지향적'으로 보일 수 있으나, 더 복잡해지고 어려워지면서 예상치 못한 오류가 발생할 수 있다.

    ```tsx
    // 변경전
    function processUsers(users: User[]) {
      const result = [];
      for (let i = 0; i < users.length; i++) {
        if (users[i].age >= 18) {
          const formattedUser = {
            name: users[i].name.toUpperCase(),
            age: users[i].age,
            isAdult: true,
          };
          result.push(formattedUser);
        }
      }
      return result;
    }
    ```

    ```tsx
    // 변경후 - 나쁜 리팩토링
    class UserProcessor {
      private users: User[];

      constructor(users: User[]) {
        this.users = users;
      }

      public process(): FormattedUser[] {
        return this.filterAdults().formatUsers();
      }

      private filterAdults(): UserProcessor {
        this.users = this.users.filter((user) => user.age >= 18);
        return this;
      }

      private formatUsers(): FormattedUser[] {
        return this.users.map((user) => ({
          name: this.formatName(user.name),
          age: user.age,
          isAdult: true,
        }));
      }

      private formatName(name: string): string {
        return name.toUpperCase();
      }
    }

    const processUsers = (users: User[]): FormattedUser[] => {
      return new UserProcessor(users).process();
    };
    ```

    ```tsx
    // 좋은 리팩토링
    const isAdult = (user: User): boolean => user.age >= 18;

    const formatUser = (user: User): FormattedUser => ({
      name: user.name.toUpperCase(),
      age: user.age,
      isAdult: true,
    });

    function processUsers(users: User[]): FormattedUser[] {
      return users.filter(isAdult).map(formatUser);
    }
    ```

    위 방식을 불필요한 복잡성은 도입하지 않고 재사용 가능한 로직으로 작은 함수로 분해했다.

3.  **일관성 부족**

    개발자들이 더 나은 코드 베이스를 가지려다 다르게 동작하는 코드를 만들 수 있다. 이는 다른 스타일 사이에서 컨텍스트 전환시 다른 개발자들과 혼선이 생길 수 있다.

    예를 들어, React-Query를 일관되게 사용하여 페칭을 처리하는 로직이 있다.

    ```tsx
    import { useQuery } from "react-query";

    function UserProfile({ userId }) {
      const { data: user, isLoading } = useQuery(["user", userId], fetchUser);

      if (isLoading) return <div>Loading...</div>;
      return <div>{user.name}</div>;
    }
    ```

    Redux Toolkit를 적용했다고 가정한다.

    ```tsx
    import { useEffect } from 'react';
    import { useDispatch, useSelector } from 'react-redux';
    import { fetchPosts } from './postsSlice';

    function PostList() {
    const dispatch = useDispatch();
    const { posts, status } = useSelector((state) => state.posts);

    useEffect(() => {
        dispatch(fetchPosts());
    }, [dispatch]);

    if (status === 'loading') return <div>Loading...</div>;
    return <div>{posts.map(post => <div key={post.id}>{post.title}</div>)}</div>;
    ```

    이와 같은 방식은 한 컴포넌트에서만 완전히 다른 상태 관리 패턴을 도입하여 좋은 코드 베이스는 아니다.

    ```tsx
    import { useQuery } from "react-query";

    function PostList() {
      const { data: posts, isLoading } = useQuery("posts", fetchPosts);

      if (isLoading) return <div>Loading...</div>;
      return (
        <div>
          {posts.map((post) => (
            <div key={post.id}>{post.title}</div>
          ))}
        </div>
      );
    }
    ```

    React-Query를 사용하여 애플리케이션 전체적인 데이터 페칭 부분의 일관성을 가지는 로직을 유지하고, 다른 개발자들이 새로운 패턴을 학습할 필요도 없다.

    중요한 것은 `코드 베이스의 일관성`이다. 새로운 패턴을 도입할 경우, 일회성 불일치를 만드는 것 보다 먼저 팀의 동의 받도록 하자.

4.  **코드 이해 없이 리팩토링하기**

    리팩토링의 가장 큰 문제는 코드를 이해하기 위한 리팩토링이다. 이건 좋지 않은 방식이고, 특정 코드를 6~9개월 정도 다루는 것을 권장하기도한다. 괜한 버그를 만들거나 성능을 해치지 않으려면.

    ```tsx
    // 변경전
    function fetchUserData(userId: string) {
      const cachedData = localStorage.getItem(`user_${userId}`);
      if (cachedData) {
        return JSON.parse(cachedData);
      }

      return api.fetchUser(userId).then((userData) => {
        localStorage.setItem(`user_${userId}`, JSON.stringify(userData));
        return userData;
      });
    }
    ```

    ```tsx
    // 변경후 - 나쁜 리팩토링
    // 🚩 캐싱은 어디로?
    function fetchUserData(userId: string) {
      return api.fetchUser(userId);
    }
    ```

    ```tsx
    // 변경 후 - 좋은 리팩토링
    async function fetchUserData(userId: string) {
      const cachedData = await cacheManager.get(`user_${userId}`);
      if (cachedData) {
        return cachedData;
      }

      const userData = await api.fetchUser(userId);
      await cacheManager.set(`user_${userId}`, userData, { expiresIn: "1h" });
      return userData;
    }
    ```

    기존 캐시 기능을 유지하면서 만료 기능을 갖춘 더 정교한 캐시 관리자를 사용하여 잠재적으로 캐시 동작까지 개선 가능한 코드가 되었다.

5.  **비지니스 맥락을 이해하자**

    만약 SEO에 크게 의지하고 느리고 비대한 SPA 구축이 된 애플리케이션을 사용시, 유지보수하기 어려운 복제품 코드만 작성하기 쉽다.

    ```tsx
    // 변경전 - 나쁜 코드
    // 🚩 SEO 중심 사이트를 위한 단일 페이지 앱은 나쁜 생각!
    function App() {
      return (
        <Router>
          <Switch>
            <Route path="/product/:id" component={ProductDetails} />
          </Switch>
        </Router>
      );
    }
    ```

    ```tsx
    // 변경후 - 좋은 리팩토링
    // ✅ 서버가 SEO 중심 사이트를 렌더링할 수 있다.
    export const getStaticProps: GetStaticProps = async () => {
      const products = await getProducts();
      return { props: { products } };
    };

    export default function ProductList({ products }) {
      return <div>...</div>;
    }
    ```

    Next.js 기반 SSG를 사용하면 SEO와 더 나은 성능을 제공한다.(Remix에도 적합하다.)

6.  **과도한 코드 통합**

    ```tsx
    // 변경전 - 😕 코드베이스에 같은 코드가 40번 이상 있었다면, 아마도 통합할 수 있을 것.
    export const quickFunction = functions
    .runWith({ timeoutSeconds: 60, memory: '256MB' })
    .https.onRequest(...);

    export const longRunningFunction = functions
    .runWith({ timeoutSeconds: 540, memory: '1GB' })
    .https.onRequest(...);
    ```

    ```tsx
    // 변경후 - 나쁜 리팩토링
    // 🚩 통합해서는 안 되는 설정을 맹목적으로 통합하는 경우
    const createApi = (handler: RequestHandler) => {
      return functions
        .runWith({ timeoutSeconds: 300, memory: "512MB" })
        .https.onRequest((req, res) => handler(req, res));
    };

    export const quickFunction = createApi(handleQuickRequest);
    export const longRunningFunction = createApi(handleLongRunningRequest);
    ```

    ```tsx
    // 변경후 - 좋은 리팩토링
    // ✅ 적절한 기본값을 설정하되 누구나 재정의할 수 있도록 허용한다.
    const createApi = (handler: RequestHandler, options: ApiOptions = {}) => {
      return functions
        .runWith({ timeoutSeconds: 300, memory: "512MB", ...options })
        .https.onRequest((req, res) => handler(req, res));
    };

    export const quickFunction = createApi(handleQuickRequest, {
      timeoutSeconds: 60,
      memory: "256MB",
    });
    export const longRunningFunction = createApi(handleLongRunningRequest, {
      timeoutSeconds: 540,
      memory: "1GB",
    });
    ```

    이와 같이 추상화의 이점을 유지하면서 유연성을 가질수 있다. 코드 통합시 "더 깔끔한" 코드를 위해 유연성을 희생하지 말 것. 말그대로 "개선"에 초점을 가지자.

### 올바른 리팩토링 방법

1. `점진적으로 작업하세요.` 대대적인 수정보다는 `관리 가능한 작은 변경`을 하세요.
2. 중요한 리팩토링이나 새로운 추상화를 도입하기 전에 `코드를 깊이 이해`하세요.
3. `기존의 코딩 스타일과 맞추세요.` 일관성은 유지보수의 핵심입니다.
4. `너무 많은 새로운 추상화를 피하세요.` 복잡성이 정말 필요한 경우가 아니면 간단하게 유지하세요.
5. `팀의 동의 없이` 새로운 라이브러리, 특히 `매우 다른 프로그래밍 스타일의 라이브러리를 추가하지 마세요.`
6. `리팩토링 전에 테스트를 작성하고, 진행하면서 업데이트하세요.` 이를 통해 원래 기능이 유지되는지 확인할 수 있습니다.
7. 이러한 원칙을 동료들이 준수하도록 하세요.

![](https://camo.githubusercontent.com/084df6e4f3d310ac8b7ca4d65bf2ca6e62ec6aa0bf37ee63b43ac27d08b1de5b/68747470733a2f2f63646e2e6275696c6465722e696f2f6170692f76312f696d6167652f617373657473253246594a494762346930316a7677305352644c35427425324664393739336662326330623134393632623838656531323366333566313164363f666f726d61743d776562702677696474683d32303030)

### 결론

리팩토링은 소프트웨어 개발에 필수이지만 기존 코드 베이스와 팀의 역할을 존중하면서 신중하게 이루어져야한다. 외부 동작을 변경하지 않으면서 내부 동작을 개선하는 것이 "리팩토링의 목표"이다.

리팩토링의 최종 목표는 외부 사용자에게는 눈에 안보이지만, 개발자의 작업을 쉽게 만드는 것. 가독성, 유지보수성, 효율성을 향상하면서 전체 시스템을 방해하지 않도록 한다.

### 출처

- [good-vs-bad-refactoring](https://www.builder.io/blog/good-vs-bad-refactoring)
