# 웹 접근성 & WAI-ARIA

## 개요

HTML 페이지 고려 사항중 우베 접근성이 있다. 이는 시각 장애인들이 웹 페이지를 원활하게 이용할 수 있도록하는 "가이드 라인"이다.

예를 들어 일반 사람들은 input, button, a 태그가 있어도 이용 가능하지만 시각 장애인은 클릭할 수 없어 탭 같은 걸로 요소를 이동할 수 밖에 없다.

따라서 탭 이동시 어느 순서의 초점으로 이동하는지, 초점이 잡혔을 때 안내 메시지가 제대로 나오는지 여부가 중요하다.

이런 설정들을 HTML native 요소만으로 처리하기 어렵기 때문에 W3C는 WAI-ARIA라는 것을 정의했다.

## 1. WAI-ARIA

WAI-ARIA(Web Accessibility Initiative - Accessible Rich Internet Applications)는 W3C에서 정의한 기술로 웹 접근성을 위해 지원되는 여러 가지 특성들을 의미한다.

일반 사용자가 보기에 정상인 화면이어도 HTML 요소에 따라 스크린 리더 등의 보조기기에서 제대로 읽히지 않을 수 있기 때문에 WAI-ARIA에서 개선을 위해 역할(Role), 속성(Property), 상태(State) 정보 추가가 가능하다.

### 1.1. Role

`Role`은 HTML 요소의 역할을 정의한다. 일반적으로는 HTML native 요소만으로 처리하는게 가장 이상적이다.

버튼은 button, 링크는 a, 체크박스는 input checkbox 등 이미 존재하는 요소로 충분히 표현 가능하다.

하지만 이미지에 버튼 클릭 이벤트를 준다던가 좀더 세세하고 다양한 설정이 하고 싶을 때는 native 요소만으로는 부족할 수 있는데, 이때 `role`을 사용하여 역할을 명시해준다.

Role의 종류는 [여기서](https://www.w3.org/TR/wai-aria/#roles_categorization) 확인하자.

```html
<!-- role example -->
<li role="menuitem">Open file…</li>
```

### 1.2 상태(State) and 속성(Property)

- 속성(Property)는 해당 요소의 특징이나 상황을 정의해서 `aria-` 접두사를 사용한다.
- 상태(State)는 요소의 현재 상태를 나타난다.

```html
<!-- property state example 1 -->
<li role="checkbox" aria-checked="true">체크박스 아이템</li>

<!-- property state example 2 -->
<div role="alert" aria-live="assertive">올바르지 않은 입력입니다.</div>
```

## 2. ARIA 속성들

### 2.1 role="checkbox" aria-checked="true"

```html
<span
  role="checkbox"
  aria-checked="false"
  tabindex="0"
  aria-labelledby="chk1-label"
></span>
<label id="chk1-label">Remember my preferences</label>

<!-- native 요소로 사용할 수 있는 경우 -->
<input type="checkbox" id="chk1-label" />
<label for="chk1-label">Remember my preferences</label>
```

- [w3c - Checkbox Example](https://www.w3.org/TR/wai-aria-practices/examples/checkbox/checkbox-1/checkbox-1.html)
- [ARIA: checkbox role](https://developer.mozilla.org/en-US/docs/Web/Accessibility/ARIA/Roles/checkbox_role)

특정 요소에 체크박스 역할을 부여하고, `aria-checked` 사용하여 체크 여부를 판단 가능하다. focus 되지 않은 요소인 경우 `tabindex`를 사용하기도 한다.

가능하면 HTML native checkbox를 사용하는게 권장되나 사용할 수 없는 경우 ARIA 속성을 사용한다.

### 2.2 aria-label, aria-labelledby, aria-describedby

- [ARIA Label](https://developers.google.com/web/fundamentals/accessibility/semantics-aria/aria-labels-and-relationships?hl=ko)

특정 요소를 `설명`하는데 사용되는 ARIA 속성이다.

1. `aria-label`은 흔히 알고 잇는 label 목적을 위한 속성이다. 특정 요소에 대한 설명을 그대로 적으면 된다.

   ```html
   <button aria-label="menu" class="button"></button>
   ```

2. `aria-labelledby`는 `aria-label`과 비슷하나 조금 다르다. `aria-label`은 요소의 설명을 직접 적는 반면, `aria-labelledby`는 다른 요소의 ID 값을 매칭시킨다.

   `aria-labelledby`는 label 자체를 재정의하기 때문에 다른 모든 label 속성들, `aria-label` 또는 HTML native label과 함께 쓰여도 항상 `aria-labelledby`을 우선한다.

   ```html
   <span id="rg-label">음료수 옵션</span>
   <div role="radiogroup" aria-labelledby="rg-label"></div>
   ```

3. `aria-describedby`는 `aria-labelledby`와 같은 방식으로 다른 요소의 ID를 매칭하여 설명을 나타낸다.

   둘의 쓰임새가 비슷하지만 목적이 다르다. label 관련 속성들과의 가장 큰 차이점은 label이 요소의 필수 설명이라면 `aria-describedby `은 어디까지나 부연 설명이라는 뜻이다. 그래서 `label` 태그와 함께 사용이 가능하다.

   ```html
   <label for="pw">Password:</label>
   <input type="password" id="pw" aria-describedby="pw-help" />
   <div id="pw-help">비밀번호는 12 자 이상으로 이루어져야 합니다</div>
   ```

### 2.3 aria-live

- [ARIA live regions](https://developer.mozilla.org/ko/docs/Web/Accessibility/ARIA/ARIA_Live_Regions)
- [콘텐츠 숨기기 및 업데이트](https://developers.google.com/web/fundamentals/accessibility/semantics-aria/hiding-and-updating-content?hl=ko)

Javascript를 상요하면 페이지를 새로 로드하지 않고 일부만 동적으로 변경이 가능하다.

동적 변경은 페이지를 볼 수 있는 사용자들은 알 수 있지만, 시각 장애인들은 알기가 어렵다.

그래서 동적인 변경을 알려줄 수 있는 `aria-live`라는 속성을 제공한다.

```html
<input type="text" name="email" />
<div role="alert" aria-live="assertive">이메일 형식이 올바르지 않습니다.</div>
```

회원가입 또는 로그인 페이지에서 사용하는 email input은 실시간으로 email 형식을 검사하여 올바른 형식이 아니라면 에러 메세지를 사용자에게 노출한다.

페이지를 볼 수 있는 상요자는 정보를 실시간으로 확인이 가능하나 시각장애인은 알지 못한다.

이럴때 `role="alert"`로 경고 역할을 갖고 요소를 만들어 실시간 변화를 감지하여 알려주는 `aria-live` 속성을 추가할 수 있다.

`aria-live` 값은 다음과 같다.

- off(default)
- polite: 현재 진행중인 음성 또는 타이핑 이후 알림
- assertive: 현재 진행중인 알림을 중단하고 즉시 알림(사용자 현재 작업에 방해할 수 있어 신중하게 적용해야함)

### 2.4 role="alert"

- [ARIA: alert role](https://developer.mozilla.org/en-US/docs/Web/Accessibility/ARIA/Roles/Alert_Role)

`alert` 역할은 사용자에게 동적인 변화를 알려줄 때 사용한다.

스크린 리더는 `alert` 역할이 붙은 요소가 업데이트되면 바로 읽기 시작한다.

`role="alert"`로 설정한다는 건 `aria-live="assertive" aria-atomic="true"`와 동일하다.

### 2.5 role="timer"

- [ARIA: timer role](https://developer.mozilla.org/en-US/docs/Web/Accessibility/ARIA/Roles/timer_role)

현재 요소가 timer로 사용되고 있다는 것을 의미한다.

초를 계속 세주려면 `aria-live` 속성을 켜주면 되지만, 1초 마다 갱신되는 컨텐츠는 알림이 부자연스럽게 끊기는 이슈가 존재한다.

이때 timer에 초점을 맞추어 남은 시간만 읽어주주길 바란다면 `role="timer"`만 추가해주자.

## 참고

- [W3C - 웹 접근성이란?](https://www.w3.org/WAI/fundamentals/accessibility-intro/ko)
- [WAI-ARIA 웹퍼블리싱](https://www.biew.co.kr/entry/WAI-ARIA-%EC%9B%B9%ED%8D%BC%EB%B8%94%EB%A6%AC%EC%8B%B1)
- [W3C - WAI ARIA](https://www.w3.org/TR/wai-aria)
