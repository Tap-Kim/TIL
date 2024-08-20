# 모노레포에서 Internal Packages를 관리하는 3가지 관리 방법(JIT, Compiled, Publishable)

## 정리

- 모노레포에서 패키지를 관리할 수 있는 방법은 JIT, Compiled, Publishable 이 세가지로 가능하다. 다른 아키텍처에서는 불가능
- JIT(Just-In Time) Packages: 빠른 개발이 가능하고 turborepo 사용시 쉽게 사용이 가능하지만 패키지가 커질수록 빌드 타임이 커지게된다.
- Compiled Packages: JIT에 비해 구성 셋팅과 사용방법이 까다롭지만 빌드 결과물을 배포하기 때문에 캐싱과 빌드 속도에 유리하다.
- Publishable Packages: 모노레포에서 구성한 패키지도 외부 라이브러리로 배포가 가능하다. 하지만 추가적인 라이브러리도 필요하고 공수 관리가 유리하진 않다.

### 참고

- 주요 키워드: monorepo, build, bundle, packages
- 관련 기술: turborepo

## 무엇을 알았는지

### 일반적인 모노레포 프로젝트 구성

|          |                                                                        |
| -------- | ---------------------------------------------------------------------- |
| packages | 공통적으로 사용할 수 있는 모듈, configuration, ui 등의 코드조각이 포함 |
| apps     | 실질적인 프로젝트에 대한 코드가 포함.                                  |

일반적으로 모노레포를 구성하게되면 위와 같은 구성을 가지게 되고, 모노레포 안에 위치하게되는 `packages`와 같은 내부 프로젝트를 `Internal Packages`라고 부른다.

외부에 퍼블리싱하는 패키지가 아닌 모노레포 프로젝트 `내부`에서만 존재하는 의미

### JIT(Just-In Time) Packages

- 장점
  - 최소한의 Configuration으로 패키지 분할이 가능해짐
- 단점
  - 패키지를 사용하는 측이 트랜스파일링, 빌드를 수행해줘야한다
  - 빌드 결과물을 캐싱하는게 불가능하다(애초에 빌드를 안함)
  - 그로인해 빌드시간이 길어질 수 있다.

```json
// turborepo의 ./packages/ui/package.json
{
  "name": "@repo/ui",
  // 진입점
  "exports": {
    "./button": "./src/button.tsx",
    "./card": "./src/card.tsx"
  },
  "scripts": {
    "lint": "eslint . --max-warnings 0",
    "check-types": "tsc --noEmit"
  }
}
```

```tsx
import { Button } from "@repo/ui/button";

export default function Home() {
  return <Button appName="hello">hello</Button>;
}
```

JIT는 export 필드를 통해 src 폴더 내부의 .tsx 파일을 직접 참조하여 패키지의 진입점을 열어준다.

따라서 사용한 측에서 해당 코드를 트랜스 파일링하고 번들링할 책임을 위임하게된다.

### 번외) next.js의 packages.json 구성

`main`를 보면 `./dist` 폴더의 `next.js`를 진입점으로 하고 있다. 이는 빌드 결과물들이 들어있는 폴더(dist) 내부의 파일.

즉 진입점를 통해 외부로 코드를 배포한다.(아래의 complied packages 과정)

```json
// next의 package.json
{
  "name": "next",
  "version": "14.2.4",
  "description": "The React Framework",
  "main": "./dist/server/next.js", // 진입점
  "license": "MIT",
  "repository": "vercel/next.js",
  "bugs": "https://github.com/vercel/next.js/issues",
  "homepage": "https://nextjs.org",
  "types": "index.d.ts"
}
```

### Compiled Packages

Compiled는 빌드 과정을 거친 결과물을 내보낸다.

따라서 JIT이 비해 오버헤드가 발생하는데 이유는 코드를 빌드할 책임이 외부 사용자에서 패키지 내부로 넘어오기 때문이다.

즉, 외부 사용자에게 빌드된 결과물을 서빙해줘야한다는 책임이 부여된다.

```json
{
  "name": "@repo/ui",
  "exports": {
    "./button": {
      "types": "./src/button.tsx",
      "default": "./dist/button.js"
    },
    "./card": {
      "types": "./src/card.tsx",
      "default": "./dist/card.js"
    }
  },
  "scripts": {
    "build": "tsc"
  }
}
```

1. exports 필드에서 내보내는 파일이 `./dist` 내부의 js 파일이 되었다.
2. scripts에 `build 프로세스`가 추가되었다.

Compiled의 가장 큰 핵심

- 패키지에서 빌드 수행
- 빌드 결과물을 외부에 공개

- 장점
  - 빌드 캐싱 가능
  - 패키지의 규모가 크다면 빌드 캐싱으로인해 빌드 시간에 큰 이점을 가질수 있다.
- 단점
  - 더 많은 추가 configuration이 필요해짐
  - 적절한 패키지를 빌드하기위해 러닝 커브가 생김
  - 빌드 순서에 의존성이 생김(Complied Packages에 의존하는 Packages가 모두 빌드되기 전까지 빌드되지 않는다)

### Publishable Packages

Publishable는 일반적으로 npm과 같은 패키지 매니저를 통해 외부로 공개된(publish) 패키지이다.

이떄 외부 사용자들은 다양한 케이스를 고려해서 구현해줘야하며, 버저닝, 로그를 관리하기 위해 changesets와 같은 도구들를 활용해야할 수 있다.

참고) [Changeset을 활용한 모노레포 자동 배포 구축하기](https://jinyisland.kr/post/changeset/)

- 장점
  - 버저닝이 용이하다.
  - 범용적인 라이브러리로 배포가 가능하다.
  - 멀티 -> 모노 통합 과정에서 SSOT를 준수하고 싶은 경우
  - 모노레포에서 분리하여 레포의 코드의 크기를 줄일 경우
- 단점
  - 많은 구성과 도구가 필요하다(ex. changeset)
  - 시간과 관리의 비용이 크다.

### 출처

- [모노레포에서 Internal Packages를 관리하는 3가지 방법](https://xionwcfm.tistory.com/m/464)
- [Turborepo - Internal Packages](https://turbo.build/repo/docs/core-concepts/internal-packages)
