# TIL 

## 어떤 공부를 했는지 or 문제가 있었는지
- [naver D2 - infer, never만 보면 두려워지는 당신을 위한 고급 TypeScript](https://d2.naver.com/helloworld/7472830)
- [startTransition](https://ko.react.dev/reference/react/useTransition) 로딩 동시성 제어
  - 현재 state를 사용하여 로딩에 대한 동시성 제어에 대해 문제가 아직 있었음. fetch는 끝났으나, 실제 화면이 이동함에 있어서 끝나지 않았는데 로딩이 끝나버리는 것
  - 이떄 transition을 사용해봤으나 동작 원리에 대해서 제대로 파악하지 못하고 있었음.

## 내가 시도해본 것들
- 예전에 동시성 제어를 위해 useTransition을 이용해서 로딩 액션을 제어해보려 했으나 실패했던 경험이 있었음

## 어떻게 해결했는지
- 문서를 잘 살펴보니 startTransition 함수만 사용해서 fetch에 대한 비동기 제어를 할수 있다는 것을 알게되었고, 마지막 finally를 사용하지 않고 로딩을 false하는 로직을 star†Transition 콜백에 넣어 사용

## 무엇을 새롭게 알았는지
- [React가 state 업데이트를 transition으로 처리하지 않습니다](https://ko.react.dev/reference/react/useTransition#react-doesnt-treat-my-state-update-as-a-transition)
  - state 업데이트를 transition으로 래핑할때 `startTransition 호출 동중에 발생해야함.
    ```ts
    startTransition(() => {
    // ✅ startTransition 호출 *도중* state 설정
      setPage('/about');
    });
    ```
  - `startTransition`에 전달하는 함수는 `동기식`이어야 한다.
    ```ts
    startTransition(() => {
      // ❌ startTransition 호출 *후에* state 설정
      setTimeout(() => {
        setPage('/about');
      }, 1000);
    });
    ```
    ```ts
    setTimeout(() => {
      startTransition(() => {
        // ✅ startTransition 호출 *도중* state 설정
        setPage('/about');
      });
    }, 1000);
    ```
  - 업데이트와 같은 Transition으로 표시할 수 없다.
    ```ts
    startTransition(async () => {
      await someAsyncFunction();
      // ❌ startTransition 호출 *후에* state 설정
      setPage('/about');
    });
    ```
    ```ts
    await someAsyncFunction();
    startTransition(() => {
      // ✅ startTransition 호출 *도중* state 설정
      setPage('/about');
    });
    ```
