# 3xx(Redirection) 상태 코드 알아보기

## 3xx Redirection

3xx 번대 상태 코드는 리다이렉션을 의미하고 이는 요청을 완료하려면 추가적인 작업(페이지 이동)이 필요하다.

클라이언트가 관심 있어하는 리소스에 대해 다른 위치를 사용하라고 말해주거나 그 리소스의 내용 대신 다른 대안 응답을 제공한다.

![](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FNfKa3%2FbtrTA6mZFph%2F1pIkhrYqpLTQWp7PEeO3I1%2Fimg.png)

HTTP에 사용되는 리다이렉션은 3가지로 나뉜다.

1. 영구 리다이렉션(Permanent): 특정 리소스의 URL이 `영구적 이동`
2. 일시 리다이렉션(Temporary): 특정 리소스의 URL이 `일시적 이동`
3. 특수 리다이렉션(Special): 캐시를 활용할 것인지에 대한 여부

종류 마다 HTTP Status Code가 다르다.

<table style="border-collapse: collapse; width: 100%;" border="1" >
<tbody>
<tr>
<td style="width: 16.6667%;">&nbsp;</td>
<td style="width: 16.6667%;"><b>301</b></td>
<td style="width: 16.6667%;"><b>308</b></td>
<td style="width: 16.6667%;"><b>302</b></td>
<td style="width: 16.6667%;"><b>303</b></td>
<td style="width: 16.6667%;"><b>307</b></td>
</tr>
<tr>
<td style="width: 16.6667%;">리다이렉션 종류</td>
<td style="width: 16.6667%;">영구(Permanent)</td>
<td style="width: 16.6667%;">영구(Permanent)</td>
<td style="width: 16.6667%;">일시(Temporary)</td>
<td style="width: 16.6667%;">일시(Temporary)</td>
<td style="width: 16.6667%;">일시(Temporary)</td>
</tr>
<tr>
<td style="width: 16.6667%;">리다이렉션시 메소드를..</td>
<td style="width: 16.6667%;"><span style="color: #ef5369;">변경</span></td>
<td style="width: 16.6667%;"><span style="color: #009a87;">유지</span></td>
<td style="width: 16.6667%;"><span style="color: #ef5369;">변경</span></td>
<td style="width: 16.6667%;"><span style="color: #ef5369;">변경 (보장)</span></td>
<td style="width: 16.6667%;"><span style="color: #009a87;">유지</span></td>
</tr>
</tbody>
</table>

![](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FcmOWpu%2FbtrTEyW6aAu%2Fmz6JIOjR9MK4pxQbcdmuo0%2Fimg.png)

## 300 Multiple Choices

Multiple Choices란 요청에 대해 둘 이상의 가능한 응답을 나타낸다. 클라이언트가 동시에 여러 시소스를 가리키는 URL 요청시, 리소스의 목록과 함께 반환된다.
어떤 서버가 하나의 HTML 문서를 영어와 프랑스어 모두 제공하는 경우 사용 가능하다. 그러나 응답 중 하나를 선택하는 표준화된 방법이 없어 이 응답코드는 `실무에선 거의 사용되지 않는다.`

![](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FczuVvv%2FbtrTOfB2pXj%2FgHwMfxxmHok1d9kqYV76aK%2Fimg.png)

## 301 Moved Permanently

- `Moved Permanently` 영구적 이동. `영구 리다이렉션`
- 요청된 리소스가 Location 헤더가 지정한 URL로 이동되었음을 나타낸다.
- 해당 URL로 리다이렉션 되면, `요청 메서드가 GET으로 변하고 본문이 제거`될 수 있다.

따라서 `301` 코드는 `GET or HEAD 방법에 대한 응답`으로만 사용하고, 대신 `POST, PUT` 경우 같은 영구 리다이렉션인 `308 Permanent Redirect`를 사용하는 것이 좋다.

### 301 상태코드 흐름

![](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FYh1LC%2FbtrTDZGa2ao%2FkxIKktYk1bDfTlTxGRlBc0%2Fimg.png)

1. 클라이언트는 `/event` 라는 URL 을 웹서버에 전송하여 이벤트 페이지에 대한 GET 요청한다.
2. 하지만, `/event` 는 더 이상 사용하지 않는 URL 이며, 현재는 `/new-event` 를 사용하고 있다고 하여 서버에서 301 리다이렉션 상태 코드와 함께 헤더의 Location 필드에 새로운 URL 인 `/new-event` 를 담아 응답메시지를 전송한다.
3. 응답메시지를 전달받은 클라이언트는 301 상태 코드와 Location 필드의 URL 을 참고하여, 새로운 URL 로 리다이렉션하라는 것을 인지한다.
4. 클라이언트는 리다이렉션 하기 위해 '`new-event` 라는 URL 을 웹서버에 전송하여 이벤트 페이지에 대한 GET 요청한다.
5. 서버는 200 OK 와 함께 이벤트 페이지의 html 을 전송해주게 된다.

### 301 상태 코드를 POST 요청시 사용하면 안되는 이유

![](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FAnZ1f%2FbtrTtB030rw%2FzWGWuVz4SWwaXHG0vwVjxK%2Fimg.png)

1. 클라이언트는 `/event` 의 이벤트 페이지에 'name' 과 'age' `바디 정보를 담은 POST 요청`을 한다.
2. 이 때 서버는 `301 상태코드`와 `Location 필드`를 활용하여 이벤트 페이지의 주소가 `/new-event` 라는 새로운 주소로 `영구적 변경`되었다는 사실을 알리고 브라우저로 하여금 리다이렉트를 지시한다.
3. 그럼 클라이언트는 해당 Location으로 요청하게 되는데, 그런데 `요청 메서드를 GET 으로 변경`시키고 `/new-event`에 요청하게 된다. 또한 `이때 본문이 제거`되는데, `POST 가 GET 으로 변경` 될 때 'name' 과 'age' 정보는 **본문에 해당하므로 제거**되었기 때문이다.
4. 따라서 `본래 클라이언트의 의도대로 흘러가지 않게 되어 결국은 본문 정보를 다시 작성하여 POST 요청을 보내야 할 것`이다. 이러한 문제 때문에 `POST 요청은 308 Permanent Redirect를 사용하기를 권고`한다.

## 302 Found

- `Found`는 다른 URL에서 리소스를 찾은 `일시적 리다이렉션`이다.
- 요청된 리소스가 Location 헤더가 지정한 URL로 일시적으로 이동되었음을 나타낸다. (말이 일시이지 새 URL로 리다이렉트 되는건 같음)
- _302는 301과 같이 요청 메서드가 GET으로 변하고 본문이 제거 될 수 있지만_, **무조건적으로 변경하지 않기 때문**에 브라우저에 따라 작업 수행이 달라지는 불확실성을 가질 수 있다는 특징이 있다.
- 이러한 **불확실성 때문에 303, 307 상태 코드가 추가**되었다. 그래서 만일 사용된 *메서드를 GET로 변경하려는 경우 대신 303 See Other를 사용*하는게 좋다.

## 301 vs 302 차이점(일시 vs 영구 리다이렉션)

301은 요청 정보가 새로운 주소로 영구적으로 옮겨졌다는 것을 말함, 302는 일시적으로 옮겨졌다는 것을 말함

![](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FeepNYB%2FbtrTz07I9SL%2FQSTckzxklADD9IwFEcB1l1%2Fimg.png)

둘을 구분하는 것은 사람이 아니라 `검색 엔진, 검색 봇`이다. 또한 어떤 리다이렉션을 쓰느냐에 따라 검색엔진 최적화에 미치는 영향은 크다.

## PRG(Post - Redirect - Get) 패턴

301 Moved Permanently와 302 Found 모두 리다이렉션 뒤 요청 메서드를 GET으로 바꾸도록 설계되어 있다. why?

예를들어 쇼핑 사이트에서 POST를 통해 주문을 요청하고, 새로고침을 가정하자. 새로고침은 이전의 요청을 재전송하기 때문에 POST 요청을 다시하게 되고, 따라서 서버에 중복 주문이 들어갈 가능성이 생긴다.

![](https://blog.kakaocdn.net/dn/LcGM5/btrTzkRNwdH/rzZ5yfgFtTWZ32hNHVLzJK/img.png)

1. 클라이언트는 주문창에서 mouse 제품과 수량은 1개로 선택하고 POST 메서드로 주문 요청을 서버에 전송한다.
2. 서버에서는 주문 요청을 데이터베이스에 저장한다.
3. 그리고 200 OK 를 반환한다.
4. 사용자가 브라우저를 새로고침(refresh) 하였다.
5. 이때 refresh 되면서 이전 POST 요청이 그대로 재전송되고, 같은 주문이 중복되어 생성되는 문제가 발생하게 된다.

이런 중복 문제를 방지하는 취지에서 설계되었다고 보면 된다.

이처럼 `PRG 패턴`은 일시적 리다이렉션에서 중복 요청을 방지하기 위해 사용하는 일종의 `HTTP 패턴`이다. 배달, 쇼핑 앱에서 주문 끝난 후 자동으로 주문 완료를 알리는 결과 창으로 이동하게 되는 동작들이 PRG 패턴의 사례이다.

위 사례에서 PRG 패턴이 적용된 302 코드를 적용해보자.

![](https://blog.kakaocdn.net/dn/brPbRD/btrTACjTSHF/mQrDBOsKfqeXynReuIi820/img.png)

1. 클라이언트는 주문창에서 mouse 제품과 수량은 1개로 선택하고 POST 메서드로 주문 요청을 서버에 전송한다.
2. 서버에서는 주문 요청을 데이터베이스에 저장한다.
3. 그리고 리다이렉트할 Location 헤더와 함께 302 Found를 반환한다.
4. 클라이언트는 헤더 메세지를 읽고 주문 결과 화면인 '/order-result/19' 로 리다이렉트 준비를 한다.
5. 클라이언트는 '/order-result/19' 로 요청을 한다. 이때 메서드를 GET으로 변경하고 body를 제거한다. 따라서 클라이언트는 문 결과 화면에 대해 단순 조회 요청을 전송하게 된다.
6. 서버는 데이터베이스에서 해당 주문을 조회한다.
7. 그리고 200 OK 를 반환한다.
8. 주문 결과 화면으로 이동하게 된 클라이언트는, 새로고침를 반복하더라도 주문 결과 화면에 대한 조회 요청만을 반복적으로 전송하게 된다. (5번으로 이동됨)

## 303 See Other

- `See Other`은 다른 URL에서 리소스를 찾는 `일시적 리다이렉션`이다.
- `302 Found` 과 기능은 동일하게 **일시 리다이렉트 시 요청 메서드가 GET로 고정되고 본문을 제거**한다.
- 다만 302 Found 와 다르게 `요청 메서드의 변경과 본문 제거 행위를 무조건적으로 보장`한다.
- 따라서 클라이언트가 요청한 리소스를 다른 URI에서 GET 요청을 통해 얻어야 할 때, 서버가 클라이언트로 보내는 응답으로 이용된다.
- 보통 PUT 또는 POST의 결과로서 요청에 대한 응답으로 클라에게 리소스의 위치를 알려주는 용도로 쓰인다.

> 참고: 303 상태 코드는 GET 메서드를 보장하여 사용자 에이전트가 잘못 해석할 가능성이 적기 때문에 302 상태 코드 대신 사용된다. 하지만 대부분의 브라우저가 이미 메서드 변경과 본문 삭제를 지원하고 있기 때문에, 실무에서는 302 Found 도 많이 사용하고 있고 사실상 문제가 없다고 한다.

## 304 Not Modified

- `Not Modified`는 수정되지 않은 최신 리소스이니 *`캐시`를 이용하자라는 의미*이다.
- 리소스 복사본 상태가 수정 되지 않아 최신 상태이므로 캐시를 이용하라는 `특수 리다이렉션`
- 즉, 클라이언트가 리소스를 요청했는데 서버로부터 304 응답이 오면, **클라이언트가 요청한 리소스가 수정되지 않은 최신 상태이니, 캐시에 가지고 있는 리소스를 그대로 사용해도 된다는 의미**를 가진다.
- 따라서 클라이언트는 캐시에서 리소스를 재사용하게 되고 이를 통해 네트워크 트래픽을 줄일수 있게 된다.
- 주의할 점은 `304 응답 메시지는 Body 본문에 어떠한 데이터도 포함해서는 안된다`. (클라이언트가 로컬에 있는 캐시에서 리소스를 가져와 사용하여야 하기 때문)
- 304 응답 코드를 반환받기 위해선, 클라이언트는 `If-Modified-Since 헤더` 혹은 `If-None-Match 헤더`를 **포함한 조건부 요청을 전송**해야 한다.
- 304 응답은 `GET과 HEAD 메소드`에만 동작한다.

> 참고: CSS, JavaScript 또는 이미지와 같은 정적 콘텐츠를 보다 효율적으로 전달하기 위해 CDN에서 자주 사용되는 기법이다.

### 캐시(Cache) 와 리다이렉션(Redirection) 관계

304 상태는 다른 URL을 전이하는게 아닌 캐시를 이용한 상태 코드인데 이것이 왜 3XX 리다이렉션과 무슨 관련이 있는 것일까?

우선, 리다이렉션을 단순히 다른 URL로 전이시키는 것으로 이해하고만 있을텐데, 정확한 기술 의미는 `페이지 단위의 실제 리소스, 폼 혹은 전체 웹 애플리케이션이 다른 URL에 위치하고 있는 상태에서 링크를 종속시키는 기술`을 말한다.

사실 `웹서버, 프록시, 캐시, 게이트웨이` 모두 클라이언트가 그들에게 HTTP 요청을 보내고 그들이 그것을 처리한다는 관점에서, 클라이언트에게 있어 모두 서버라고 할 수 있다.

![](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FbECxau%2FbtrTKcmNZvR%2FieG2jSLxNhQH4BTEYzvET1%2Fimg.png)

따라서 로컬 캐시라고 할지라도 클라이언트 입장에선 멀리 떨어져있든 바로 옆에 있든 리소스 제공자 역할을 하는 서버(Server)로 치부하기 때문에 캐시를 다른 URL로 보아 캐시에 링크를 종속시키니까 리다이렉트라는 표현을 쓰는 것이다.

### 304 상태 흐름

#### If-Modified-Since 헤더 사용

![](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FK9aCu%2FbtrTtWdOMy1%2FPx5HQjXAQGon5WkZL6YkO1%2Fimg.webp)

1. 서버가 클라이언트로부터 요청을 받으면 If-Modified-Since 헤더가 있는지 확인
2. 없으면 최초 요청이기 때문에 서버는 최신 콘텐츠를 클라이언트에 전달 (200 OK)
3. 반면에 If-Modified-Since 헤더가 있으면, 서버는 브라우저가 파일에 마지막으로 액세스한 이후 파일이 편집되었는지 확인
4. 파일이 수정된 상태라면, 서버는 클라이언트에 새 파일 복사본을 반환 (200 OK)
5. 파일이 최신 상태라면, 브라우저는 캐시에서 콘텐츠를 검색할 수 있음 (304 Not Modified)

<br/>

1. 클라이언트는 서버에게 이미지(logo.gif) 요청을 한다.
2. 서버는 리소스와 더불어 헤더에 마지막으로 수정된 날짜와 시간을 나타내는 `Last-Modified` 를 설정하여 응답해준다.

![](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2Fc8Nk4Y%2FbtrTyCFhGlK%2FCrNFpAIlWbsJMMRxJxzkB0%2Fimg.png)

3. 클라이언트가 같은 리소스에 대해 재요청을 한다. 이때 최초 응답 시 받은 `Last-Modified` 헤더를 참조하여 `If-Modified-Since` 헤더를 설정하여 요청한다.
4. 서버는 `If-Modified-Since` 요청 헤더값과 원본 이미지 수정 시간을 비교하여 변경 사항이 없으면 304 Not Modified로  응답한다.

![](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2Fb8dy5n%2FbtrTAA0Kdil%2FS40MAzurjSZj7l94ue6wxK%2Fimg.png)

5. 클라이언트는 안심하고 로컬 캐시에서 이미지 리소스를 가져와 사용한다.
![](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FbPgfRa%2FbtrTKs31ICQ%2FrCma4OB3100kkrfMz2JnaK%2Fimg.png)

#### If-None-Match 헤더 사용

If-Modified-Since가 날짜와 시간을 기준으로 파일 상태를 판별하는 것이라면 `If-None-Match` 는 `ETag 헤더`를 이용하여 파일 상태를 판단한다.

> ETag 헤더는 **특정 버전의 리소스를 식별하는 고유 식별자**이다. `서버는 파일이 변경될 때마다 새 ETag 값을 생성하고 이전 ETag 값을 유지`한다.

![](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FbdQnME%2FbtrTtW5Uzdw%2FEftZS8571V5SI9P8vhGebK%2Fimg.webp)

1. 클라이언트가 파일을 요청하면, 서버에선 리소스와 `ETag` 헤더를 수신

![](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FdcTxf5%2FbtrTELaRGcw%2FeskvzzlwjNOM4Ew1Rqfz4k%2Fimg.png)

2. 클라이언트가 같은 파일을 재요청. 이때 `If-None-Match` 헤더와 함께 서버에 전송
3. 서버는 `If-None-Match` 와 `ETag` 가 일치하는지 확인
4. 일치 하지 않은 경우 서버는 파일의 새 복사본을 보내고, 일치하면 304 Not Modified 를 반환.

![](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FdMZjYj%2FbtrTBkkO0XS%2FPfM6RawyBZnKGuysMlGQe0%2Fimg.png)

## 305 Use Proxy

- `Use Proxy`는 리소스가 프록시를 통해서만 액세스될 수 있음을 표현
- 요청한 응답은 반드시 프록시를 통해서 접속해야 하는 것을 알려준다.
- 보안 문제로 인해 더 이상 사용되지 않는 HTTP 상태 코드

## 306 Switch Proxy

- `Switch Proxy`는 클라이언트가 대체 프록시를 사용하도록 리다이렉션(switch) 시킨다
- 이 응답 코드는 더이상 사용되지 않으며, 현재는 미래에 사용을 위해 예약되어 있다.

## 307 Temporary Redirect

- `Temporary Redirect` 일시 리다이렉션
- `302 Found`와 기능은 동일하지만, `일시 리다이렉트 시 요청 메서드와 본문을 유지`한다
- 즉, 클라이언트가 처음에 POST 메서드와 함께 생성할 리소스 정보를 Body 에 담아 전송하였다면, 새로운 URL 로 리다이렉트하더라도 그 메서드와 Body 내용이 유지되는 것이다.
- **다만 첫 요청에 POST가 사용되었다면, 두번째 요청도 반드시 POST를 사용해야 한다.**
- 클라이언트가 요청한 리소스가 다른 URI에 있으며, 이전 요청과 동일한 메소드를 사용하여 요청해야 할 때 응답으로 이용된다.

> 307과 308 차이는 영구 리다이렉션이냐, 일시 리다이렉션이냐 차이이며, 둘다 리다이렉트 시 요청 메서드와 본문 body를 유지한다는 점은 동일하다.

## 308 Permanent Redirect

- `Permanent Redirect` 영구 리다이렉션
- `301 Moved Permanently`과 기능은 동일하지만, `영구 리다이렉트 시 요청 메서드와 본문을 유지`한다
- 즉, 클라이언트가 처음에 POST 메서드와 함께 생성할 리소스 정보를 Body 에 담아 전송하였다면, 새로운 URL 로 리다이렉트하더라도 그 메서드와 Body 내용이 유지되는 것이다.
- **다만 첫 요청에 POST가 사용되었다면, 두번째 요청도 반드시 POST를 사용해야 한다.**
- 클라이언트가 요청한 리소스가 다른 URI에 있으며, 이전 요청과 동일한 메소드를 사용하여 요청해야 할 때 응답으로 이용된다

### 308 상태 흐름

![](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FNIWi4%2FbtrTxJDslbb%2F8DUzjP4MSbPElUMm78PaRK%2Fimg.png)

1. 클라이언트는 '/event' 의 이벤트 페이지에 'name' 과 'age' 바디 정보를 담은 POST 요청을 한다.
2. 이 때 `서버는 301 Moved Permanently` 상태코드와 Location 필드를 활용하여 이벤트 페이지의 주소가 '/new-event' 라는 새로운 주소로 영구적 변경되었다는 사실을 알리고 브라우저로 하여금 리다이렉트를 지시한다.
3. 클라이언트는 해당 Location으로 자동 리다이렉트 된다.
4. 여기까지 301과 동일하지만, 클라이언트가 처음 사용했던 POST 메서드가 유지되고, 'name' 과 'age' 정보가 본문에 유지되어 있는 모습을 확인할 수 있다.

복습해보자면 `301 Moved Permanently` 와 `308 Permanent Redirect` 는 동일하게 영구적 리다이렉트 요청 기능을 수행하지만, **메서드를 GET 으로 변경하고 본문 내용을 삭제하느냐에 있어 중요한 차이점**을 가지게 된다.

## 출처

- [🌐 3XX (Redirection) 상태 코드 - 총정리 모음](https://inpa.tistory.com/entry/HTTP-🌐-3XX-Redirection-상태-코드-제대로-알아보기)
