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

# 어포던스(Affordance - 행동 유도성)

> 출처: https://brunch.co.kr/@booungsae/4

## 정리
### 어떤 문제가 있었는지
- 어포던스(Affordance)에 대한 개념이 모호했음.
- UX 영역에서 자주 회자되는 용어로, 사람마다 시대를 거쳐감에 따라 용어에 대한 해석이 달라지게 되어 혼란스러움.

### 무엇을 알았는지
1. `깁슨의 어포던스`
  - '비직접-지각'과 '직접-지각'의 관점
    - 비직접-지각: 전통적 심리 관점, 외부로 자극 받을때 머릿속에 이미지를 생성하고 (왜곡된)기억하는 관점
    - 직접-지각: 왜곡된 기억을 가지지 않고 사물을 있는 그대로 받아드리는 관점.
  - 정의: _특정 대상_ 이 가지고 있는 본질적, 외면적 속성이 우리가 받아드리는 **무한한 가짓수의 행동을 유발**하는 상태
2. `노먼의 어포던스`
    > 사물의 지각된(Perceived) 특성 또는 사물이 갖고 있는 실제적 특성을 말하는 것으로, 특히 그것을 어떻게 사용할 수 있느냐를 결정하는 근본적 속성을 말한다. (…) 행동유도성은 사물을 어떻게 다루면 될 것인가에 관한 강력한 단서를 제공한다.
  
  - 즉, "어떻게 사용할 수 있느냐를 결정하는 근본적인 속성"이란 '특정'한 행동을 유발해야함.
  -> 만약 사용자가 제작자의 의도와 다른 행동을 취한다면 `잘못된 디자인`이라는 것
  - 깁슨의 모든 행동에 대한 유발 가능성의 반대인 **특정 해동의 유발 가능성**을 제시함.

3. `게이버의 어포던스`
  - 어포던스와 지각 정보(Perceptual Information)의 유무로 어포던스의 영역을 나뉨
  - 게이버는 "어포던스는 지각(사물을 바라봤을 때)되었을 대 결국 (행위자에게) 의미가 있다"라고 의미
  ![](https://c2.staticflickr.com/2/1662/23940976100_ee33c98e6b_d.jpg)

### 무엇을 새롭게 알았는지
- UX라는 영역에서 심리학적인 관점의 다양한 논의를 알아볼수 있었다.
- 과거 깁슨 > 노먼 > 게이버 > 그 이후 시간이 지날수록 어포던스라는 의미를 다양하게 받아드리며, 그 범위를 좁힐지 넓힐지 합칠지에 따라 관점과 해석이 달라진다.
- 어포던스라는 의미 자체가 추상적이기 때문에 앞으로 UX 분야에서 어떻게 해석될지 궁금해진다.