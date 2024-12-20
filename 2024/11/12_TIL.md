# Thinking in React(5가지 기본 원칙)

리액트는 디자인을 바라보는 방식과 앱을 빌드하는 방식을 바꿀 수 있다. 리액트로 사용자 인터페이스를 구축할 때는 먼저 컴포넌트 조각을 나누고 각 컴포넌트에 대해 서로 다른 시각적 상태를 지정한다. 마지막으로는 서로 연결하여 데이터가 흐르도록 한다. 지금부터는 검색 가능한 제품 데이터 테이블을 구축하는 "사고 과정"을 살펴보자.

![](https://react.dev/images/docs/s_thinking-in-react_ui.png)

리액트에서 UI를 구현하려면 일반적으로 동일한 5단계를 따른다.

## 1단계: UI를 컴포넌트 계층 구조로 나누기

- 프로그래밍: 새 함수나 객체를 생성할지 여부를 결정할 때도 동일한 기법이다. 단일 책임 원칙(SRP), 컴포넌트는 이상적으로 한 가지 일만 수행해야 한다. 컴포넌트가 커지면 더 작은 하위 컴포넌트로 분해해야한다.
- CSS: 클래스 선택자를 무엇을 위해 만들어야할지 고려해야한다.(하지만 컴포넌트는 덜 세분화되어 있다.)
- 디자인: 디자인의 레이어를 어떻게 구성할지 고려하자.

JSON이 잘 구조화되어 있다면 UI 컴포넌트 구조에 자연스럽게 매핑되는 경우가 많다. UI와 데이터 모델이 동일한 정보 아키텍처, 즉 동일한 형태를 가지고 있는 경우가 많기 때문이다. UI를 컴포넌트로 분리하고 각 컴포넌트가 데이터 모델의 한 부분과 일치하도록 하자.

![](https://react.dev/images/docs/s_thinking-in-react_ui_outline.png)

1. `FilterableProductTable`(회색)에는 전체 앱이 포함되어 있습니다.
2. `SearchBar`(파란색)은 사용자 입력을 수신합니다.
3. `ProductTable`(라벤더)는 사용자 입력에 따라 목록을 표시하고 필터링합니다.
4. `ProductCategoryRow`(녹색)은 각 카테고리에 대한 제목을 표시합니다.
5. `ProductRow`(노란색)은 각 제품에 대한 행을 표시합니다.

`ProductTable`(라벤더색)을 보면 테이블 헤더('이름' 및 '가격' 레이블이 포함된)가 자체 구성 요소가 아니다. 이것은 선호도의 문제이며 어느 쪽이든 사용할 수 있다. 이 예제에서는 `ProductTable의` 목록 안에 표시되므로 `ProductTable의` 일부이다. 그러나 이 헤더가 복잡해지면(예: 정렬을 추가하는 경우) 자체 `ProductTableHeader` 컴포넌트로 이동할 수 있다.

이제 목업에서 컴포넌트를 식별했으므로 계층 구조로 정렬한다. 목업의 다른 컴포넌트 안에 있는 컴포넌트는 계층 구조에서 하위로 나타나야 한다.

- FilterableProductTable
  - SearchBar
  - ProductTable
    - ProductCategoryRow
    - ProductRow

## 2단계: 정적인 UI 만들기

가장 간단한 접근 방식은 아직 `상호 작용 기능을 추가하지 않고 데이터 모델에서 UI를 렌더링하는 버전을 빌드`하는 것이다. 정적 버전을 먼저 빌드하고 나중에 상호 작용을 추가하는 것이 더 쉬운 경우가 많다. 정적 버전을 구축하려면 타이핑은 많이 하고 생각은 많이 하지 않아도 되지만 상호 작용 기능을 추가하려면 타이핑은 많이 하지 않고 생각은 많이 해야 한다.

데이터 모델을 렌더링하는 정적 버전의 앱을 빌드하려면 `다른 컴포넌트를 재사용`하고 `prop을 사용하여 데이터를 전달하는 컴포넌트`를 빌드하는 것이 좋다. 프로퍼티는 부모에서 자식으로 데이터를 전달하는 방법. (상태의 개념에 익숙하다면 이 정적 버전을 빌드할 때 상태를 전혀 사용하지 말자. 상태는 상호 작용, 즉 시간이 지남에 따라 변경되는 데이터에만 사용된다.)

계층 구조에서 상위 컴포넌트부터 작성하는 '하향식'(예: `FilterableProductTable`) 또는 하위 컴포넌트부터 작성하는 '상향식'(예: `ProductRow`)으로 작성할 수 있다. 간단한 예시에서는 일반적으로 하향식으로 작성하는 것이 더 쉽고, 대규모 프로젝트에서는 상향식으로 작성하는 것이 더 쉽다.

컴포넌트를 빌드하고 나면 **데이터 모델을 렌더링하는 재사용 가능한 컴포넌트 라이브러리**를 갖게 된다. 이 앱은 정적 앱이므로 컴포넌트는 JSX만 반환한다. 계층 구조의 맨 위에 있는 컴포넌트(`FilterableProductTable`)는 데이터 모델을 `prop`으로 사용한다. 데이터가 최상위 컴포넌트에서 트리의 하단에 있는 컴포넌트로 흘러내리기 때문에 이를 단방향 데이터 흐름이라고 한다.

## 3단계: 최소한의 상태 정의하기

UI를 대화형으로 만들려면 사용자가 기본 데이터 모델을 변경할 수 있도록 해야 한다. 이를 위해 state를 사용한다.

상태는 **앱이 기억해야 하는 최소한의 변경 데이터 집합**이라고 생각하자. 상태를 구조화할 때 가장 중요한 원칙은 [DRY(반복하지 않기)](https://en.wikipedia.org/wiki/Don%27t_repeat_yourself) 유지하는 것. 애플리케이션에 필요한 상태의 최소한의 표현을 파악하고 그 외의 모든 것은 온디맨드 방식으로 계산한다. 예를 들어 쇼핑 목록을 작성하는 경우 항목을 상태의 배열로 저장할 수 있다. 목록에 있는 항목 수까지 표시하려면 항목 수를 다른 상태 값으로 저장하지 말고 배열의 길이를 읽으면 된다.

1. 원래 제품 목록
2. 사용자가 입력한 검색어
3. 체크박스의 값
4. 필터링된 제품 목록

이 중 어느 것이 상태는? 속하지 않는 것을 식별하자.

- 시간이 지나도 변하지 않는가? 🙅‍♂️
- 부모로부터 props를 통해 전달되나? 🙅‍♂️
- 컴포넌트에 있는 기존 상태나 props를 기반으로 계산할 수 있나? 🙅‍♂️

다시 한번 살펴보자.

- 원래 제품 목록은 props로 전달되므로 상태 🙅‍♂️
- 검색어는 시간이 지남에 따라 바뀌고 어떤 것에서도 계산될 수 없으므로 상태 🙆‍♂️
- 체크박스의 값은 시간이 지남에 따라 변하고 어떤 것으로부터도 계산될 수 없으므로 상태 🙆‍♂️
- 필터링된 제품 목록은 원래 제품 목록을 가져와서 검색어와 체크박스 값에 따라 필터링 하여 계산할 수 있기 때문에 상태 🙅‍♂️

즉, 검색 텍스트와 체크박스 값만 상태이다!

## 4단계: 상태가 존재해야할 위치 파악하기

최소 상태 데이터를 식별한 후에는 이 **상태 변경을 담당하는 컴포넌트 또는 상태를 소유하는 컴포넌트를 식별**해야 한다.

> 기억: 리액트는 단방향 데이터 흐름을 사용하여 부모 컴포넌트에서 자식 컴포넌트로 컴포넌트 계층구조를 따라 데이터를 전달한다. 어떤 컴포넌트가 어떤 상태를 소유해야 하는지 즉시 명확하지 않을 수 있다.

1. 해당 상태에 따라 무언가를 렌더링하는 모든 구성 요소를 식별한다.
2. 계층 구조에서 모든 구성 요소 위에 있는 가장 가까운 공통 부모 구성 요소를 찾는다.
3. state가 어디에 위치해야 하는지 결정하자
   1. 종종 해당 상태를 공통 부모에 직접 넣을 수 있다.
   2. 또한 상태를 공통 부모 위에 있는 일부 구성 요소에 넣을 수도 있다.
   3. 상태를 소유하는 것이 합리적인 구성 요소를 찾을 수 없는 경우, 상태를 보관하기 위한 새로운 구성 요소를 만들고 공통 부모 구성 요소 위의 계층 구조 어딘가에 추가한다.

이전 단계에서는 이 애플리케이션에서 검색 입력 텍스트와 확인란의 값이라는 두 가지 상태를 발견했다. 이 예제에서는 **항상 함께 표시되므로 같은 위치에 배치하는 것이 좋다**.

1. 상태를 사용하는 컴포넌트를 식별
   - `ProductTable`은 해당 상태(검색 텍스트 및 확인란 값)를 기준으로 제품 목록을 필터링해야 한다.
   - `SearchBar`는 해당 상태(검색 텍스트 및 확인란 값)를 표시해야 한다.
2. 공통 부모 찾기: 두 컴포넌트가 공유하는 첫 번째 부모 컴포넌트는 `FilterableProductTable`다.
3. 상태가 어디에 있는지 결정: 필터 텍스트와 체크된 상태 값은 `FilterableProductTable`에 보관한다.

따라서 상태 값은 `FilterableProductTable`에 저장된다. `useState() Hook`을 사용하여 컴포넌트에 상태를 추가한다. `FilterableProductTable`의 상단에 상태 변수 두 개를 추가하고 초기 상태를 지정한다.

```jsx
// FilterableProductTable
function FilterableProductTable({ products }) {
  const [filterText, setFilterText] = useState("");
  const [inStockOnly, setInStockOnly] = useState(false);

  // 중략

  <div>
    <SearchBar filterText={filterText} inStockOnly={inStockOnly} />
    <ProductTable
      products={products}
      filterText={filterText}
      inStockOnly={inStockOnly}
    />
  </div>;
}
```

```jsx
// SearchBar
function SearchBar({ filterText, inStockOnly }) {
  return (
    <form>
      <input type="text" value={filterText} placeholder="Search..." />
      <label>
        <input type="checkbox" checked={inStockOnly} /> Only show products in
        stock
      </label>
    </form>
  );
}
```

```jsx
// ProductTable
function ProductTable({ products, filterText, inStockOnly }) {
  const rows = [];
  let lastCategory = null;

  products.forEach((product) => {
    if (product.name.toLowerCase().indexOf(filterText.toLowerCase()) === -1) {
      return;
    }
    if (inStockOnly && !product.stocked) {
      return;
    }
    if (product.category !== lastCategory) {
      rows.push(
        <ProductCategoryRow
          category={product.category}
          key={product.category}
        />
      );
    }
    rows.push(<ProductRow product={product} key={product.name} />);
    lastCategory = product.category;
  });

  return (
    <table>
      <thead>
        <tr>
          <th>Name</th>
          <th>Price</th>
        </tr>
      </thead>
      <tbody>{rows}</tbody>
    </table>
  );
}
```

하지만 아직 입력과 같은 사용자 작업에 응답하는 코드를 추가하지 않았다.

## 5단계: 역방향 데이터 흐름 제어하기

현재 앱은 소품과 상태가 계층 구조 아래로 흐르면서 올바르게 렌더링된다. 그러나 사용자 입력에 따라 상태를 변경하려면 계층 구조의 깊은 곳에 있는 양식 컴포넌트가 `FilterableProductTable`의 상태를 업데이트해야 하는 등 다른 방식으로 데이터가 흐르도록 지원해야 한다.

**리액트는 이 데이터 흐름을 명시적으로 만들지만 양방향 데이터 바인딩보다 조금 더 많은 타이핑이 필요**하다. 위의 예제에서 입력하거나 확인란을 선택하면 리액트가 사용자의 입력을 무시하는 것을 볼 수 있다. 이는 의도적인 것이다. `<input value={filterText} />`를 작성하면 입력의 값 prop이 항상 `FilterableProductTable`에서 전달된 `filterText` 상태와 같도록 설정된다. `filterText` 상태는 설정되지 않으므로 입력은 변경되지 않는다.

사용자가 `form` 입력을 변경할 때마다 해당 변경 사항을 반영하여 상태가 업데이트되도록 만들고 싶다. 이 상태는 `FilterableProductTable`이 소유하므로 이 상태만 `setFilterText` 및 `setInStockOnly`를 호출할 수 있다. `SearchBar`가 `FilterableProductTable`의 상태를 업데이트하도록 하려면 이러한 함수를 `SearchBar`에 전달해야 한다.

```jsx
function FilterableProductTable({ products }) {
  const [filterText, setFilterText] = useState('');
  const [inStockOnly, setInStockOnly] = useState(false);

  return (
    <div>
      <SearchBar
        filterText={filterText}
        inStockOnly={inStockOnly}
        onFilterTextChange={setFilterText}
        onInStockOnlyChange={setInStockOnly} />
```

```jsx
function SearchBar({
  filterText,
  inStockOnly,
  onFilterTextChange,
  onInStockOnlyChange
}) {
  return (
    <form>
      <input
        type="text"
        value={filterText}
        placeholder="Search..."
        onChange={(e) => onFilterTextChange(e.target.value)}
      />
      <label>
        <input
          type="checkbox"
          checked={inStockOnly}
          onChange={(e) => onInStockOnlyChange(e.target.checked)}
```

# 출처

- [Thinking in React](https://react.dev/learn/thinking-in-react)
