# DRY - 잘못된 추상화의 일반적인 원인

> 출처: https://swizec.com/blog/dry-the-common-source-of-bad-abstractions/

![](https://swizec.com/static/96961c7ae5eec58f69d9228861724008/e11e5/Reading-DRY-code-vs-simple-code-ach4f7.webp)

## 정리
### 어떤 문제가 있었는지
- 코드 작성을 위해 DRY(Don't Repeat Yourself)를 하지 않기 위한 추상화를 시도한다. 
- `반복`의 키워드에 집중한 나머지 무분별하게 추상화하게 된다면, 가독성, 유지보수가 떨어질 가능성을 초래한다. 

### 어떻게 해결했는지
- 디자인 패턴(글에선 '팩토리 패턴'을 예시), 상황에따라 DRY를 버리기, 매개변수 추가하기, 추상화 다시하기 등의 순서로 잘못된 추상화를 고쳐나간다.

### 무엇을 알았는지
- 최종적으로 더 나은 추상화를 작성하기 위해 기존 코드 베이스에서 `역할에 맞는 컴포넌트` 분리(ex. 메뉴, 버튼), 추상화된 컴포넌트를 `합성 패턴`으로 조합.(상황에 따라 여러가지 패턴으로 해결)
- 각 컴포넌트 별로 관심사를 분리하여 동일한 기능 또는 동일한 디자인을 가지는 컴포넌트를 기능이나 디자인 별로 관심사를 분리해서 작성하여, DRY에 매몰되어 가독성과 유지보수가 떨어지는 추상화에서 벗어나는 구조를 가질수 있음.

## 코드 베이스
### 1. 일반적 구조
   
```tsx
const NavigationMenu = () => {
    return (
        <ul>
        <li>
            <a href="/about">
            <img src="question-icon.png" />
            About
            </a>
        </li>
        <li>
            <a href="/contact">
            <img src="person-icon.png" />
            Contact
            </a>
        </li>
        <li>
            <a href="/buy">
            <img src="cash-icon.png" />
            Buy
            </a>
        </li>
        // ...
        </ul>
    );
};
```

### 2. 잘못된 DRY
    
```tsx
const NavigationMenu = () => {
const items = [
    {
        url: "/about",
        icon: "question-icon.png",
        label: "About",
    },
    {
        url: "/contact",
        icon: "person-icon.png",
        label: "Contact",
    },
    {
        url: "/buy",
        icon: "cash-icon.png",
        label: "Buy",
    },
    // ...
]

return (
    <ul>
    {items.map((item) => (
        <li>
        <a href={item.url}>
            <img src={item.icon} />
            {label}
        </a>
        </li>
    ))}
    </ul>
)
}
```

### 3. 팩토리 패턴을 더한 DRY(나쁜 추상화)

```tsx
function makeNavItem(url, icon, label) {
  return { url, icon, label }
}

const NavigationMenu = () => {
  const items = [
    makeNavItem("/about", "question-icon.png", "About"),
    makeNavItem("/contact", "person-icon.png", "Contact"),
    makeNavItem("/buy", "cash-icon.png", "Buy"),
    // ...
  ]

  return (
    <ul>
      {items.map((item) => (
        <li>
          <a href={item.url}>
            <img src={item.icon} />
            {label}
          </a>
        </li>
      ))}
    </ul>
  )
}
```

### 4. 관심사의 분리를 통한 DRY(with. 합성 패턴)
```tsx
const MenuItem = ({ href, style, icon, children }) => (
	<li style={style}><a href={href}><img src={icon} />{children}</a>
)

const NavigationMenu = () => {
	return (
		<ul>
			<MenuItem href="/about" icon="question-icon.png">About</MenuItem>
			<MenuItem href="/contact" icon="person-icon.png">Contact</MenuItem>
			<MenuItem href="/buy" icon="cash-icon.png" style={{ border: '1px solid red' }}>Buy</MenuItem>
		</ul>
	)
}
```