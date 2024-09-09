# `Turborepo` Caching

## 정리

- Turborepo는 JavaScript 및 TypeScript 코드베이스를 위한 고성능 빌드 시스템이다. 이 시스템은 모노레포 확장을 위해 설계되었으며 단일 패키지 작업 공간의 워크플로도 더 빠르게 만든다. Turborepo는 레포지토리에서 실행해야 하는 작업을 최적화하는 가벼운 접근 방식을 추구한다.
- 그 중에서 빌드 캐싱이 있다. 로컬 캐싱, 원격 캐싱, 커스텀 원격 캐싱 세가지 방식으로 빌드 캐싱이 가능하다.
- 캐싱의 구조는 프로젝트 빌드시 Aartifacts가 생성되고, 이때 global-hash, task hash가 생성되어 두 해시 중 작업이 변경되면 해시는 캐시를 미스하면 새로운 Artifacts가 생성되고 히트가 되면 캐싱된 Aartifacts를 내려주기 때문에 빌드 결과물의 리소스와 시간을 절약할 수 있다.

### 참고

- 주요 키워드: turborepo, caching, local caching, remote caching, custom remote caching, Vercel
- 관련 기술: cache, build cache, custom remote cache API

## 무엇을 알았는지

### 터보레포 캐싱

터보레포는 캐싱을 사용해서 빌드 속도를 높이고 동일한 작업을 두번하지 않도록한다. 작업이 캐시 가능한 경우 터보레포는 작업이 처음 실행된 시점의 지문을 사용해서 캐시에서 작업 결과를 복원한다.

![](https://turbo.build/_next/image?url=%2F_next%2Fstatic%2Fmedia%2Fwhy-turborepo-solution.02448c98.png&w=828&q=75&dpl=dpl_E1LZbFPm4kUpUJSLx1X1sZ3XhBGi)

최초 프로젝트를 빌드하면 [Task inputs](https://turbo.build/repo/docs/crafting-your-repository/caching#task-inputs)이 이전에 실행한 적이 없기 때문에 캐시 미스가 난다. input은 로컬 파일 시스템 또는 [원격 캐시](https://turbo.build/repo/docs/core-concepts/remote-caching)에서 체크할 해시로 변환된다.

> `input`은 터보레포에 의해 해시되어 작업을 위한 "지문(fingerprint)"을 생성한다. 이 지문이 일치하면 캐시에 기록이된다. 내부적으로 터보레포는 [`global hash`](https://turbo.build/repo/docs/crafting-your-repository/caching#global-hash-inputs)와 [`task hash`](https://turbo.build/repo/docs/crafting-your-repository/caching#global-hash-inputs)라는 두 개의 해시를 생성하는데, 두 해시 중 하나가 변경되면 작업 해시는 캐시를 놓치게 된다.

그 다음 빌드시 캐시가 히트되면 아래와 같은 메시지가 표시된다.

![](https://turbo.build/_next/image?url=%2F_next%2Fstatic%2Fmedia%2Ffull-turbo.0fd6323e.png&w=828&q=75&dpl=dpl_E1LZbFPm4kUpUJSLx1X1sZ3XhBGi)

이미 빌드 결과물이 캐시에 있기 때문에 애플리케이션을 다시 빌드할 이유가 없고 캐시에서 이전 빌드 결과물을 복원해서 리소스와 시간을 절약할 수 있다.

### 무엇을 캐시할까?

터보레포는 `turbo.json`의 [output key](https://turbo.build/repo/docs/reference/configuration#outputs)에 정의된 작업의 파일 출력을 캐시에 저장한다. 캐시 히트가 발생하면 터보레포는 캐시에서 파일을 복원한다.

### 로컬 캐싱

터보레포는 `원격 캐싱`을 통해 로컬 캐싱을 활성화하여 빌드 완료시 빌드 결과물 artifact를 로컬 디렉토리에 저장해두고, 나중에 같은 해쉬로 빌드를 시도할 때 이를 그대로 이용한다.
(로컬 저장소 예시 : ./node_modules/.cache/turbo/78awdk123. tar.zs)

하지만 로컬 캐싱의 유용성은 크지 않은데, 보통 빌드 후 실행보다는 개발용 서버를 띄워 사용한다. 이는 CI에서 활용한다해도 CI가 모두 동일한 머신에서 돌지 않기 때문에 각각의 빌드 완료시 각각의 CI 머신의 로컬에만 빌드 캐싱 결과물이 저장된
다. 다시말해 `로컬 캐싱 방식은 CI에선 활용될 일이 많지 않다`.

### 원격 캐싱

![](https://turbo.build/_next/image?url=%2Fimages%2Fdocs%2Fremote-caching-dark.png&w=1080&q=75&dpl=dpl_E1LZbFPm4kUpUJSLx1X1sZ3XhBGi)

로컬 캐싱의 문제점을 해결하기위해 주로 원격 캐싱이 사용된다. 이는 한 머신에서 빌드한 결과물을 다른 환경에서도 공유하기 위한 방식이다. 특정 캐시 서버가 존재하고, 빌드 결과 Artifacts를 보유한다. 그리고 같은 해쉬값의 빌드 요청이 온 경우, 해당 artifacts를 그대로 내려주는 방식이다.

하지만 원격 캐싱은 강력한 기능이임에 책임도 크다. 먼저 캐싱이 올바르게 이루어지는지 확인하고 [환경 변수 처리](https://turbo.build/repo/docs/crafting-your-repository/using-environment-variables)를 다시 한번 확인해야 한다. 또한 터보레포는 로그를 Artifacts로 취급하므로 콘솔에 적히는 내용에 유의해야한다.

특별히 설정 없이 기본 설정값으로 build를 실행하게되면, Vercel의 자체적인 원격 캐싱만 사용하게 된다.

```json
{
  "$schema": "https://turbo.build/schema.json",
  "remoteCache": {
    // signature 활성화되어 있는지 여부를 나타냅니다.
    "signature": true
  }
}
```

### 커스텀 원격 캐싱 서버

Vercel과 같은 자체 호스팅 서비스를 사용하지 않으면 원격 캐싱 서버를 자동으로 사용할수가 없다. 이때 터보레포 캐싱 API를 지원하는 서버를 직접 구현하거나 사용해서 원격 캐싱을 적용 가능하다. 이런 방식을 `커스텀 리모트 캐시`라고 부른다.

```bash
turbo run build --api="https://my-server.example.com" --token="xxxxxxxxxxxxxxxxx
```

예를 들면 위와 같이 `my-server.example.com`라는 커스텀 원격 캐싱 서버를 구축해서 해당 서버에서 특정 캐시를와 빌드 결과 Artifacts를 보관한다. 그리고 다음 빌드 요청시 해당 서버에서 해쉬값의 빌드 요청이 들어오면 변경 여부를 확인하여 캐싱된 Artifacts를 그대로 내려보낼지 새로운 Artifacts를 내려보낼지 결정해서 성능을 향상 시킬수 있다.

### 출처

- [turborepo - caching](https://turbo.build/repo/docs/crafting-your-repository/caching)
- [turborepo - Remote Caching](https://turbo.build/repo/docs/core-concepts/remote-caching)
- [[React] Turborepo 용 custom remote cache 서버 구축하기 - 1️⃣ 터보레포의 캐싱 구조
  ](https://programming119.tistory.com/279)
