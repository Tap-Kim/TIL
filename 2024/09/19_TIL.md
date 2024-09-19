# HTTP 개요(2)

## HTTP 흐름

클라이언트와 서버가 통신할때의 과정

1. TCP 연결: TCP 연결은 요청을 보내거나 응답을 는데 사용. 클라이언트는 새 연결을 열거나, 기존 연결을 재사용 또는 서버에 대한 여러 TCP 연결이 가능하다.
2. HTTP 메세지 전송: HTTP 메시지(HTTP/2 이전)는 인간이 읽을 수 있고 간단한 메세지가 프레임 속에 캡슐화되어 있어 직접 읽는게 불가능하지만 원칙은 동일하다.
   ```http
   GET / HTTP/1.1
   Host: developer.mozilla.org
   Accept-Language: fr
   ```
3. 서버에 의해 전송된 응답을 읽는다.

   ```http
   HTTP/1.1 200 OK
   Date: Sat, 09 Oct 2010 14:28:02 GMT
   Server: Apache
   Last-Modified: Tue, 01 Dec 2009 20:18:22 GMT
   ETag: "51142bc1-7449-479b075b2891b"
   Accept-Ranges: bytes
   Content-Length: 29769
   Content-Type: text/html

   <!DOCTYPE html... (here comes the 29769 bytes of the requested web page)
   ```

4. 연결을 닫거나 다른 요청들을 위해 재사용한다.

HTTP 파이프라이닝이 활성화되면 첫 번째 응답이 완전히 수신될 때까지 기다리지 않고 여러 요청을 전송할 수 있다. HTTP 파이프라이닝은 오래된 소프트웨어와 최신 버전이 공존하는 기존 네트워크에서는 구현하기 어려운 것으로 입증되었다. HTTP 파이프라이닝은 프레임 내에서 더 강력한 멀티플렉싱 요청으로 HTTP/2에서 대체되었습니다.

## HTTP 메세지

HTTP/1.1 및 이전 버전에 정의된 HTTP 메세지는 사람이 읽을 수 있따. HTTP/2에서는 메세지가 이진 구조인 프레임에 포함되어 헤더 압축 및 멀티 플렉싱과 같은 최적화가 가능하다. 이 버전의 HTTP에서는 원본 HTTP 메세지의 일부만 전송되더라도 각 메세지의 의미는 변경되지 않으며 클라이언트는 (가상으로) 원본 HTTP/1.1 요청을 재구성한다. 따라서 HTTP/1.1 형식의 HTTP/2 메세지를 이해하는 것이 유용하다.

### 요청

![](https://mdn.github.io/shared-assets/images/diagrams/http/overview/http-request.svg)

요청시 고려해야할 요소들

- 클라이언트에서 수행하려는 작업을 정의하는 HTTP [메서드](https://developer.mozilla.org/en-US/docs/Web/HTTP/Methods)로, 일반적으로 GET, POST와 같은 동사 또는 [OPTIONS](https://developer.mozilla.org/en-US/docs/Web/HTTP/Methods/OPTIONS), [HEAD](https://developer.mozilla.org/en-US/docs/Web/HTTP/Methods/HEAD)와 같은 명이다. 일반적으로 클라이언트는 리소스를 가져오거나(GET 사용) [HTML form](https://developer.mozilla.org/en-US/docs/Learn/Forms)의 값을 게시하지만(POST 사용), 다른 경우는 더 많은 작업이 필요할 수 있다.
- fetch할 리소스 URL에는 프로토콜(http://), 도메인(여기서는 developer.mozilla.org) 또는 TCP 포트(여기서는 80)와 같이 문백에 분명한 요소를 제거한 리소스의 URL 이다.
- HTTP 프로토콜의 버전
- 서버에 대한 추가 정보를 전달하는 Optional [headers](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers)
- POST와 같은 일부 메서드 경우 body로, 전송된 리소스를 포함하는 응답과 유사하다.

### 응답

![](https://mdn.github.io/shared-assets/images/diagrams/http/overview/http-response.svg)

응답시 고려해야할 요소들

- HTTP 프로토콜 버전
- 요청 성공 여부와 이유를 나타내는 [status code](https://developer.mozilla.org/en-US/docs/Web/HTTP/Status)
- status code에 대한 권한이 없는 짧은 상태 메세지
- 요청 헤더와 같은 HTTP
- 선택 사항으로 fetch한 리소스가 포함된 body

## HTTP 기반 API

HTTP를 기반으로 가장 일반적으로 사용되는 API는 자바스크립트 HTTP 요청시 사용하는 Fecth API이다, 이는 XMLHttpRequest API를 대체한다.

또 다른 API인 [서버 전송 이벤트](https://developer.mozilla.org/en-US/docs/Web/API/Server-sent_events)는 서버가 HTTP를 전송 메커니즘으로 사용하여 클라이언트에 이벤트를 전송할 수 있는 단방향 서비스다. 클라이언트는 [EventSource](https://developer.mozilla.org/en-US/docs/Web/API/EventSource) 인터페이스를 사용하여 연결을 열고 이벤트 핸들러를 사용한다.

클라이언트 브라우저는 HTTP 스트림에 도착하는 메세지를 적절한 이벤트 객체로 자동 변환하고 이벤트 유형에 등록된 이벤트 핸들러가 있는 경우 해당 핸들러로 유형별 이벤트 핸들러가 설정되지 않는 경우 [onmessage](https://developer.mozilla.org/en-US/docs/Web/API/EventSource/message_event) 이벤트 핸들러로 전달한다.

## 결론

**HTTP는 사용하기 쉬운 확장 가능한 프로토콜이다.**

클라이언트-서버 구조와 헤더 추가 기능이 결합되어 웹의 확장 기능과 함께 HTTP가 발전할 수 있다.

- HTTP/2는 성능 향상을 위해 프레임에 HTTP 메시지를 포함함으로써 약간의 복잡성을 추가했지만
- 메시지의 기본 구조는 HTTP/1.0 이후 동일하게 유지되고 있다.
- 세션 흐름은 기본으로 유지되어 HTTP 네트워크 모니터로 조사하고 디버깅할 수 있다.

## 출처

- [HTTP](https://developer.mozilla.org/en-US/docs/Web/HTTP/Overview)
