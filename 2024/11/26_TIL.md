# 자바스크립트의 가비지 컬렉션

## 가비지 컬렉션의 선정 기준

자바스크립트는 _도달 가능성_(reachability) 개념을 사용해 메모리 관리를 수행한다.
"_도달 가능한_(reachability)" 값은 어떻게든 접근하거나 사용할 수 있는 값을 의미한다. 이런 값은 메모리에서 삭제 되지 않는다.

- 현재 함수의 지역 변수와 매개변수
- 중첩 함수의 체인에 있는 함수에서 사용되는 변수와 매개변수
- 전역 변수
- 기타 등등

이런 값은 루트(root)라고 부른다.

루트가 참조하는 값이나 체이닝으로 루트에서 참조할 수 있는 값은 "도달 가능한 값"이 된다.

전역 변수에 객체가 저장되있다고 가정해보자. 이 객체의 프로퍼티가 또 다른 객체를 참조한다면, 프로퍼티가 참조하는 객체는 도달 가능한 값이 된다.

자바스크립트 엔진 내에선 [가비지 컬렉터(garbage collector)](<https://en.wikipedia.org/wiki/Garbage_collection_(computer_science)>)가 끊임없이 동작한다. 가비지 컬렉터는 모든 객체를 모니터링하고, 도달할 수 없는 객체는 삭제한다.

## 예시1 - 참조 1개

```js
// user엔 객체 참조 값이 저장
let user = {
  name: "John",
};
```

![](https://ko.javascript.info/article/garbage-collection/memory-user-john.svg)

이 그림에서 화살표는 "객체 참조"를 나타낸다. 전역 변수 `user`는 `{name: "John"}` (줄여서 John)이라는 객체를 참조한다.

John의 프로퍼티 `name`은 원시값을 저장하고 있기 때문에 객체 안에 표현했다.

`user`의 값을 다른 값으로 덮어쓰면 참조(화살표)가 사라진다.

```js
user = null;
```

![](https://ko.javascript.info/article/garbage-collection/memory-user-john-lost.svg)

이제 John은 도달할 수 없는 상태가 되었다. John에 접근할 방법도, John을 참조하는 것도 모두 사라졌다. 가비지 컬렉터는 이제 John에 저장된 데이터를 삭제하고, John을 메모리에서 삭제한다.

## 예시2 - 참조 2개

참조를 `user`에서 `admin`으로 복사했다고 가정해보자.

```js
// user엔 객체 참조 값이 저장
let user = {
  name: "John",
};

let admin = user;
```

![](https://ko.javascript.info/article/garbage-collection/memory-user-john-admin.svg)

그리고 위에서 한것 처럼 user의 값을 다른 값으로 덮어써 보자.

```js
user = null;
```

전역 변수 `admin`을 통하면 여전히 객체 John에 접근할 수 있기 때문에 John은 메모리에서 삭제되지 않는다. 이 상태에서 `admin`을 다른 값(null 등)으로 덮어쓰면 John은 메모리에서 삭제될 수 있다.

## 예시3 - 연결된 객체들

```js
function marry(man, woman) {
  woman.husband = man;
  man.wife = woman;

  return {
    father: man,
    mother: woman,
  };
}

let family = marry(
  {
    name: "John",
  },
  {
    name: "Ann",
  }
);
```

함수 `marry`는 매개변수로 받은 *두 객체를 서로 참조*하게 하면서 'marry' 시키고, 두 객체를 포함하는 새로운 객체를 반환한다.

메모리 구조는 아래와 같이 나타난다.

![](https://ko.javascript.info/article/garbage-collection/family.svg)

지금은 `모든 객체가 도달 가능한 상태`이다.

이제 참조 두 개를 지워보자.

```js
delete family.father;
delete family.mother.husband;
```

![](https://ko.javascript.info/article/garbage-collection/family-delete-refs.svg)

삭제한 두 개의 참조 중 하나만 지웠다면, 모든 객체가 여전히 도달 가능한 상태이다.

하지만 참조 두 개를 지우면 John으로 들어오는 참조(화살표)는 모두 사라져 John은 도달 가능한 상태에서 벗어난다.

![](https://ko.javascript.info/article/garbage-collection/family-no-father.svg)

외부로 나가는 참조는 도달 가능한 상태에 영향을 주지 않는다. 외부에서 들어오는 참조만이 도달 가능한 상태에 영향을 준다. John은 이제 도달 가능한 상태가 아니기 때문에 메모리에서 제거된다. John에 저장된 데이터(프로퍼티) 역시 메모리에서 사라진다.

가비지 컬렉션 후 메모리 구조는 아래와 같다.

![](https://ko.javascript.info/article/garbage-collection/family-no-father-2.svg)

## 도달할 수 없는 섬

객체들이 연결되어 섬 같은 구조를 만드는데, 이 섬에 도달할 방법이 없는 경우, 섬을 구성하는 객체 전부가 메모리에서 삭제된다.

근원 객체 `family`가 아무것도 참조하지 않도록 해 보자.

```js
family = null;
```

이제 메모리 내부 상태는 다음과 같아진다.

![](https://ko.javascript.info/article/garbage-collection/family-no-family.svg)

도달할 수 없는 섬 예제는 도달 가능성이라는 개념이 얼마나 중요한지 보여준다.

John과 Ann은 여전히 서로를 참조하고 있고, 두 객체 모두 외부에서 들어오는 참조를 갖고 있지만, 이것만으로는 충분하지 않다는걸 보여준다.

"family" 객체와 루트의 연결이 사라지면 루트 객체를 참조하는 것이 아무것도 없게 된다. 섬 전체가 도달할 수 없는 상태가 되고, 섬을 구성하는 객체 전부가 메모리에서 제거된다.

## 내부 알고리즘

`mark-and-sweep`이라 불리는 가비지 컬렉션 기본 알고리즘에 대해 알아보자.

`가비지 컬렉션`은 대개 다음 단계를 거쳐 수행된다.

- 가비지 컬렉터는 루트(root) 정보를 수집하고 이를 `mark(기억)` 한다.
- 루트가 참조하고 있는 모든 객체를 방문하고 이것들을 `mark` 한다.
- mark 된 모든 객체에 방문하고 그 객체들이 참조하는 객체도 `mark` 한다. 한번 방문한 객체는 전부 `mark` 하기 때문에 같은 객체를 다시 방문하는 일은 없다.
- 루트에서 도달 가능한 모든 객체를 방문할 때까지 위 과정을 반복한다.
- mark 되지 않은 모든 객체를 메모리에서 삭제한다.

다음과 같은 객체 구조가 있다고 해보자.

![](https://ko.javascript.info/article/garbage-collection/garbage-collection-1.svg)

오른편에 '도달할 수 없는 섬’이 보인다. 이제 가비지 컬렉터의 `mark-and-sweep` 알고리즘이 이것을 어떻게 처리하는지 보자.

첫 번째 단계에선 루트를 `mark` 한다.

![](https://ko.javascript.info/article/garbage-collection/garbage-collection-2.svg)

이후 루트가 참조하고 있는 것들을 `mark` 한다.

![](https://ko.javascript.info/article/garbage-collection/garbage-collection-3.svg)

도달 가능한 모든 객체를 방문할 때까지, `mark` 한 객체가 참조하는 객체를 계속해서 `mark` 한다.

![](https://ko.javascript.info/article/garbage-collection/garbage-collection-4.svg)

방문할 수 없었던 객체를 메모리에서 삭제한다.

![](https://ko.javascript.info/article/garbage-collection/garbage-collection-5.svg)

루트에서 페인트를 들이붓는다고 상상하면 이 과정을 이해하기 쉽. 루트를 시작으로 참조를 따라가면서 도달가능한 객체 모두에 페인트가 칠해진다고 생각하면 된다. 이때 페인트가 묻지 않은 객체는 메모리에서 삭제된다.

지금까지 가비지 컬렉션이 어떻게 동작하는지에 대한 개념을 알아보았다. 자바스크립트 엔진은 실행에 영향을 미치지 않으면서 가비지 컬렉션을 더 빠르게 하는 다양한 최적화 기법을 적용한다.

## 최적화 기법

- **generational collection(세대별 수집)** – 객체를 '새로운 객체’와 '오래된 객체’로 나눈다. 객체 상당수는 생성 이후 제 역할을 빠르게 수행해 금방 쓸모가 없어지는데, 이런 객체를 '새로운 객체’로 구분한다. 가비지 컬렉터는 이런 객체를 공격적으로 메모리에서 제거한다. 일정 시간 이상 동안 살아남은 객체는 '오래된 객체’로 분류하고, 가비지 컬렉터가 덜 감시한다.
- **incremental collection(점진적 수집)** – 방문해야 할 객체가 많다면 모든 객체를 한 번에 방문하고 mark 하는데 상당한 시간이 소모된다. 가비지 컬렉션에 많은 리소스가 사용되어 실행 속도도 눈에 띄게 느려지는데, 자바스크립트 엔진은 이런 현상을 개선하기 위해 가비지 컬렉션을 여러 부분으로 분리한 다음, 각 부분을 별도로 수행한다. 작업을 분리하고, 변경 사항을 추적하는 데 추가 작업이 필요하긴 하지만, 긴 지연을 짧은 지연 여러 개로 분산시킬 수 있다는 장점이 있다.
- **idle-time collection(유휴 시간 수집)** – 가비지 컬렉터는 실행에 주는 영향을 최소화하기 위해 CPU가 유휴 상태일 때에만 가비지 컬렉션을 실행한다.
  이 외에도 다양한 최적화 기법과 가비지 컬렉션 알고리즘이 있다. 다양한 기법과 알고리즘을 소개하고 싶지만, 엔진마다 세부 사항이나 기법이 다르기 때문에 따로 알아보는게 좋다. 엔진이 발전하면 기법도 달라지기 때문에 학습해야 할 이유가 진짜 없다면 ‘심화’ 학습은 그리 가치 있지 않다고 생각한다.

## 요약

- 가비지 컬렉션은 엔진이 자동으로 수행하므로 개발자는 이를 억지로 실행하거나 막을 수 없다.
- 객체는 도달 가능한 상태일 때 메모리에 남는다.
- 참조된다고 해서 도달 가능한 것은 아니다. 서로 연결된 객체들도 도달 불가능할 수 있다.

모던 자바스크립트 엔진은 좀 더 발전된 가비지 컬렉션 알고리즘을 사용한다.

## 참조

- [가비지 컬렉션](https://ko.javascript.info/garbage-collection#ref-322)
