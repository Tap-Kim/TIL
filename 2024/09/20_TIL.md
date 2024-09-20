# HTTP 프로토콜

## HTTP 프로토콜이란?

HTTP(Hypertext Transfer Protocol)는 통신 프로토콜이며, 상호간 정의한 규칙을 의미하여 특정 기기 간에 데이터를 주고받기 위해 정의된다. 웹에선 브라우저와 서버 간에 데이터를 주고받기 위한 방식으로 HTTP 프로토콜을 사용하고 있다.

## 특징

HTTP 프로토콜은 `stateless`이다. 상태가 없다는 의미는 데이터를 주고 받기 위해 데이터 요청이 서로 독립적으로 관리 된다는 뜻. **이전 데이터 요청과 다음 데이터 요청이 서로 관련이 없다**.

이런 이점으로 서버에서 세션과 같은 추가 정보를 관리하지 않아도 되고, 다수의 요청 처리 및 서버 부하를 줄일 수 있는 성능 상의 이점을 가진다.

일반적으로 TCP/IP 통신 위에서 동작하며 기본 포트트 80번 포트이다.

## HTTP Request & HTTP Response

![](https://joshua1988.github.io/images/posts/web/http/request-response.png)

클라이언트란 _요청을 보내는 쪽을 의미_, 일반적으로 웹에서는 "브라우저"가 된다. 서버란 받는 쪽을 의미하고 일반적으로 데이터를 보내주는 "원격 컴퓨터"가 된다.

## URL

![](https://joshua1988.github.io/images/posts/web/http/url-structure.png)

URL(Uniform Resource Locators)은 서버에 자원을 요청하기 위한 주소이다.

## HTTP 요청 메서드

일반적으로 HTTP 요청 메서드(HTTP Request Methods)는 아래와 같다.

- GET : 존재하는 자원에 대한 **요청**
- POST : 새로운 자원을 **생성**
- PUT : 존재하는 자원에 대한 **변경**
- PATCH: 리소스의 부분만 **수정**
- DELETE : 존재하는 자원에 대한 **삭제**

기타 요청 메서드

- HEAD : 서버 헤더 정보를 획득. GET과 비슷하나 Response Body를 반환하지 않음
- OPTIONS : 서버 옵션들을 확인하기 위한 요청. CORS에서 사용

## HTTP 상태코드

### 2xx - 성공

200번대의 상태 코드는 대부분 성공을 의미

- 200 : GET 요청에 대한 성공
- 204 : No Content. 성공했으나 응답 본문에 데이터가 없음
- 205 : Reset Content. 성공했으나 클라이언트의 화면을 새로 고침하도록 권고
- 206 : Partial Content. 성공했으나 일부 범위의 데이터만 반환

### 3xx - 리다이렉션

300번대의 상태 코드는 대부분 클라이언트가 이전 주소로 데이터를 요청하여 서버에서 새 URL로 리다이렉트를 유도하는 경우.

- 301 : Moved Permanently, 요청한 자원이 새 URL에 존재
- 303 : See Other, 요청한 자원이 임시 주소에 존재
- 304 : Not Modified, 요청한 자원이 변경되지 않았으므로 클라이언트에서 **캐싱된 자원을 사용하도록 권고**. **ETag**와 같은 정보를 활용하여 변경 여부를 확인

### 4xx - 클라이언트 에러

400번대 상태 코드는 대부분 클라이언트의 코드가 잘못된 경우. 유효하지 않은 자원을 요청했거나 요청이나 권한이 잘못된 경우 발생한다. 가장 익숙한 상태 코드는 404 코드이다. 요청한 자원이 서버에 없다는 의미한다.

- 400 : Bad Request, 잘못된 요청
- 401 : Unauthorized, 권한 없이 요청. Authorization 헤더가 잘못된 경우
- 403 : Forbidden, 서버에서 해당 자원에 대해 접근 금지
- 405 : Method Not Allowed, 허용되지 않은 요청 메서드
- 409 : Conflict, 최신 자원이 아닌데 업데이트하는 경우. ex) 파일 업로드 시 버전 충돌

### 5xx - 서버 에러

500번대 상태 코드는 서버 쪽에서 오류가 난 경우.

- 501 : Not Implemented, 요청한 동작에 대해 서버가 수행할 수 없는 경우
- 503 : Service Unavailable, 서버가 과부하 또는 유지 보수로 내려간 경우

## 전체적인 HTTP 요청과 응답

![](https://joshua1988.github.io/images/posts/web/http/http-full-structure.png)

## 출처

- [프런트엔드 개발자가 알아야하는 HTTP 프로토콜 Part 1](https://joshua1988.github.io/web-development/http-part1/)
