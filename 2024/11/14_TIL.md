# 리액트 상태 구조 구성 방법

## 상태 구조화 원칙

1. **관련된 상태를 그룹화하기**: 항상 두개 이상의 상태 변수를 동시에 업데이트하는 경우 하나의 상태 변수로 병합하는 것이 좋다.
2. **모순된 상태를 피하기**: 여러 상태들이 서로 모순되게 구성하면 실수의 여지가 생긴다.
3. **여러 상태 피하기**: 렌더링 중에 컴포넌트의 prop이나 기존 상태 변수에서 일부 정보를 계산할 수 있다면 해당 정보를 해당 컴포넌트의 상태에 넣지 않아야 한다.
4. **복제 상태 피하기**: 동일한 데이터가 여러 상태 변수 또는 중첩된 개체 내에 중복되면 동기화 상태를 유지하기가 어렵다.
5. **깊게 중첩된 상태 피하기**: 계층 구조가 깊은 상태는 업데이트하기가 편하지 않다. 가능하면 상태를 flat하게 구조화하는 것이 좋다.

이런 원칙의 목표는 실수 없이 상태를 쉽게 업데이트할 수 있도록 하는 것. 상태에서 중복 및 복제 데이저를 제거하면 모든 데이터가 동기화 상태를 유지하는데 도움이 되고, DB 엔지니어가 버그 발생 가능성을 줄이기 위해 DB 구조를 "정규화"하는 것과 비슷하다. `"상태를 가능한 단순하게 만들되, 그보다 더 단순해선 안된다"`

## 상태 모순 방지

컴포넌트가 복잡할수록 어떤 일이 발생했는지 파악하기 어렵다. `isSending`과 `isSent`는 동시에 참이 되어서는 안 되므로 세 가지 유효한 상태 중 하나를 취할 수 있는 `하나의 상태 변수로 대체`하는 것이 좋다. 예를 들면 `typing (initial)`, `sending`, and `sent`.

```jsx
// Bad
const [isSending, setIsSending] = useState(false);
const [isSent, setIsSent] = useState(false);

async function handleSubmit(e) {
  e.preventDefault();
  setIsSending(true);
  await sendMessage(text);
  setIsSending(false);
  setIsSent(true);
}

// Good
const [status, setStatus] = useState("typing");

async function handleSubmit(e) {
  e.preventDefault();
  setStatus("sending");
  await sendMessage(text);
  setStatus("sent");
}

const isSending = status === "sending";
const isSent = status === "sent";
```

## 여러 상태 피하기

렌더링 중 컴포넌트 props나 기존 상태 변수에 일부 정보를 계산할 수 있는 경우 해당 정보를 해당 컴포넌트의 상태에 넣지 말아야한다.

예를 들어 이 양식을 살펴보자. 작동하지만 중복 상태를 찾을 수 있을까?

```jsx
// Bad
const [firstName, setFirstName] = useState("");
const [lastName, setLastName] = useState("");
const [fullName, setFullName] = useState("");

function handleFirstNameChange(e) {
  setFirstName(e.target.value);
  setFullName(e.target.value + " " + lastName);
}

function handleLastNameChange(e) {
  setLastName(e.target.value);
  setFullName(firstName + " " + e.target.value);
}

// Good
const [firstName, setFirstName] = useState("");
const [lastName, setLastName] = useState("");

const fullName = firstName + " " + lastName;

function handleFirstNameChange(e) {
  setFirstName(e.target.value);
}

function handleLastNameChange(e) {
  setLastName(e.target.value);
}
```

따라서 onChange 핸들러는 이를 업데이트하기 위해 특별한 작업을 수행할 필요가 없다. `setFirstName` 또는 `setLastName`을 호출하면 다시 렌더링이 트리거되고 다음 전체 이름이 새 데이터에서 계산된다.

## 상태를 props에 미러링하지 말 것

```jsx
function Message({ messageColor }) {
  const [color, setColor] = useState(messageColor);
  //
}
```

`color` 상태 변수는 `messageColor` 프로퍼티로 초기화된다. 문제는 나중에 부모 컴포넌트가 다른 값(예: '파란색' 대신 '빨간색'을 전달하면 색상 상태 변수가 업데이트되지 않는다!) **상태는 첫 번째 렌더링 중에만 초기화된다**.

그렇기 때문에 상태 변수에 일부 프로퍼티를 `미러링`하면 혼동을 일으킬 수 있다. 대신 코드에서 `messageColor` 프로퍼티를 직접 사용하자. 더 짧은 이름을 지정하려면 상수를 사용권장.

```jsx
function Message({ messageColor }) {
  const color = messageColor;
  //
}
```

이렇게 하면 부모 컴포넌트에서 전달된 props와 동기화되지 않는다. props를 상태로 `미러링`하는 것은 `특정 props에 대한 모든 업데이트를 무시하려는 경우에만 의미가 있다`. 관례에 따라 props 이름을 `initial` 또는 `default`으로 시작하여 새 값이 무시된다는 것을 명심하자.

```jsx
function Message({ initialColor }) {
  // `color` 상태 변수는 `initialColor`의 *첫 번째* 값을 보유한다.
  // `initialColor` 프로퍼티에 대한 추가 변경 사항은 무시된다.
  const [color, setColor] = useState(initialColor);
  //
}
```

## 중복 상태 피하기

```jsx
const [items, setItems] = useState(initialItems);
const [selectedItem, setSelectedItem] = useState(items[0]);
```

현재는 선택한 항목을 `selectedItem` 상태 변수에 객체로 저장한다. 하지만 좋은 방식은 아니다. `selectedItem`의 내용은 항목 목록 내의 항목 중 하나와 동일한 객체다. 즉, 항목 자체에 대한 정보가 두 곳에 중복된다. 이것이 왜 문제일까? 각 항목을 편집 가능하게 만들어 보자.

선택한 항목도 업데이트할 수 있지만 더 쉬운 방법은 **중복을 제거하는 것**. 이 예제에서는 `selectedItem` 객체(항목 내부의 객체와 중복을 생성하는) 대신 `selectedId`를 상태로 유지한 다음 **항목 배열에서 해당 ID를 가진 항목을 검색하여 selectedItem을 가져온다**.

```jsx
const [items, setItems] = useState(initialItems);
const [selectedId, setSelectedId] = useState(0);

const selectedItem = items.find((item) => item.id === selectedId);
```

중복 상태는 사라지고 필수 상태만 유지된다.

이제 선택한 항목을 편집하면 아래 메시지가 즉시 업데이트된다. 이는 `setItems`가 다시 렌더링을 트리거하고 `items.find(...)`가 업데이트된 제목의 항목을 찾기 때문이다. **선택한 ID만 필수적이므로 선택한 항목을 상태로 유지할 필요가 없다**. 나머지는 렌더링 중에 계산하면 된다.

무엇보다 객체 내부의 하나의 요소만 바뀌었는데, 전체 객체를 수정하게 되면 불필요한 상태 업데이트로 인해 무분별한 렌더링이 발생할 수 있다.

## 깊게 중첩된 상태 피하기

[상태가 너무 중첩](https://react.dev/learn/updating-objects-in-state#updating-a-nested-object)되어 있어 쉽게 업데이트할 수 없는 경우 `flat`하게 만드는 것이 좋다. 이 데이터를 재구성할 수 있는 한 가지 방법이 있다. 각 장소가 하위 장소의 배열을 갖는 트리와 같은 구조 대신 **각 장소가 하위 장소 ID의 배열을 보유**하도록 할 수 있다. 그런 다음 각 장소 ID에서 해당 장소로의 매핑을 저장한다.

```jsx
// Bad
export const initialTravelPlan = {
  id: 0,
  title: "(Root)",
  childPlaces: [
    {
      id: 1,
      title: "Earth",
      childPlaces: [
        {
          id: 2,
          title: "Africa",
          childPlaces: [
            {
              id: 3,
              title: "Botswana",
              childPlaces: [],
            },
          ],
          // ...
        },
      ],
    },
  ],
};

// Good
export const initialTravelPlan = {
  0: {
    id: 0,
    title: "(Root)",
    childIds: [1, 42, 46],
  },
  1: {
    id: 1,
    title: "Earth",
    childIds: [2, 10, 19, 26, 34],
  },
  2: {
    id: 2,
    title: "Africa",
    childIds: [3, 4, 5, 6, 7, 8, 9],
  },
  // ...
};
```

이제 상태가 `flat`(`정규화`라고도 함)이 되었으므로 중첩된 항목을 업데이트하기가 더 쉬워진다. 이제 place를 제거하려면 두 가지 수준의 상태만 업데이트하면 된다. 부모 place의 업데이트된 버전은 `childIds` 배열에서 제거된 ID를 제외해야 합니다. 루트 `table` 객체의 업데이트된 버전은 부모 place의 업데이트된 버전을 포함해야 한다.

상태는 얼마든지 중첩할 수 있지만 `flat`으로 만들면 여러 가지 문제를 해결할 수 있다. 상태를 더 쉽게 업데이트할 수 있고 중첩된 객체의 다른 부분에 중복이 생기지 않도록 하는 데 도움이 된다.

때로는 중첩된 상태의 일부를 하위 컴포넌트로 이동하여 상태 중첩을 줄일 수도 있다. 이는 아이템의 마우스오버 `hover`와 같이 **저장할 필요가 없는 임시 UI 상태에 효과적**이다.

## 출처

- [Choosing the State Structure](https://react.dev/learn/choosing-the-state-structure#principles-for-structuring-state)
