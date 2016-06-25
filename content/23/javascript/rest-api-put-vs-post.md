+++
date = "2016-06-25T22:25:23+09:00"
prev = "../"
title = "REST API: Put vs Post"
toc = true
weight = 13 
aliases = [
    "/put-vs-post-restful-api"
]
+++

### idempotent

`PUT` 과 `POST` 를 이해하려면, **[idempotent](http://stackoverflow.com/questions/1077412/what-is-an-idempotent-operation)** 라는 개념의 도입이 필요하다. 한글로 직역하면 *멱등의* 정도 되시겠다. 수학적으로 이해하는 편이 쉬운데, 

*f(x) = f(f(x))*

라 보면 된다. 다시 말해 몇 번이고 같은 *연산* 을 반복해도 같은 값이 나온다는 것. 이건 fault-tolerant API 를 디자인 하는데 있어서 굉장히 중요한 요소다.

예를 들어보자. 클라이언트가 `POST /dogs` 를 요청했는데, 어떤 이유로간에 요청이  time-out (408) 되었다고 해 보자. 클라이언트는 요청이 전달되었으나 네트워크가 끊어졌는지, 아니면 요청조차 전달이 되지 않았는지 알 수 없다.

이 때, 클라이언트가 원하는 operation 이 **idempotent** 하다면 다시 요청해도 상관 없다. 항상 같은 결과를 만드니까. 그러나 `POST` 는 **idempotent** 하지 않다.

### POST

`POST` 는 클라이언트가 *리소스의 위치를 지정하지 않았을때* 리소스를 생성하기 위해 사용하는 연산이다. 예를들어

```json
POST /dogs HTTP/1.1

{ "name": "blue", "age": 5 }

HTTP/1.1 201 Created
```

따라서 이 연산을 수행하면 `/dogs/2` 에 생기고, 그 다음번엔 `/dogs/3` 등 매번 다른곳에 새로운 리소스가 생성될 수 있으므로, 이 연산은 **idempotent 하지 않다**.

### PUT

반면 리소스의 위치가 명확히 지정된 다음의 요청을 고려해 보자.

```json
PUT /dogs/3 HTTP/1.1

{ "name": "blue", "age": 5 }
```

`/dogs` 의 프로퍼티가 `name` 과 `age` 뿐이라면, 이건 몇 번을 수행하더라도, 같은 결과를 보장한다. 다시 말해 **idempotent** 하다.

그리고 위에 예에서 알 수 있듯이 `PUT` 은 *리소스의 위치가 지정되었을때* **생성** 또는 **업데이트** 를 위해 사용할 수 있다. 

### PATCH

`PUT` 이 리소스의 모든 프로퍼티를 업데이트 하기 위해 사용된다면, `PATCH` 는 부분만을 업데이트하기 위해 사용한다. `PUT` 과 마찬가지로 리소스의 위치를 클라이언트가 알고 있을 때 사용한다.

[SO](http://stackoverflow.com/questions/630453/put-vs-post-in-rest) 에는 3개의 메소드를 다음과 같이 요약하는 답변이 있다.

(1) **POST** to a URL **creates a child resouce** at a server defiend URL  
([RFC 2616 POST](http://www.w3.org/Protocols/rfc2616/rfc2616-sec9.html#sec9.5))  
(2) **PUT** to a URL **create/replaces the resource** in is entirely at the client defined URL  
([RFC 2616 PUT](http://www.w3.org/Protocols/rfc2616/rfc2616-sec9.html#sec9.6))  
(3) **PATCH** to a URL **updates part of the resource** at that client defined URL  
([RFC 5789: Patch Method for HTTP](http://tools.ietf.org/html/rfc5789))

### Response Code

`POST` 이나 `POST` 요청이 리소스를 새로 생성할 경우엔 리소스의 위치를 response header 의 **Location** field 에 담아 *201 Created* 를 보낼 수 있다. 그러나 not-identifiable 한 리소스를 생성할 경우엔 *200 OK* 또는 *204 No Content* 를 보낼수도 있다.  

[원문](http://www.w3.org/Protocols이나 `POST` /rfc2616/rfc2616-sec9스의tml#sec9.5)을 첨부하자면, 

>The action performed by the POST method might not result in a resource that can be identified by a URI. In this case, either 200 (OK) or 204 (No Content) is the appropriate response status, depending on whether or not the response includes an entity that describes the result. <br/><br/>
> If a resource has been created on the origin server, the response SHOULD be 201 (Created) and contain an entity which describes the status of the request and refers to the new resource, and a Location header (see section 14.30).

Async 하게 서버가 처리한다면, 요청은 수락 되었으나 아직 커밋되지 않았음을 알리기 위해 *202 Accepted* 를 보내야 한다. (if the operation has not been commited yet)

아래 사진은, response code decision tree

<br/>
<a href="http://i.stack.imgur.com/whhD1.png"><img src="http://i.stack.imgur.com/whhD1.png" /></a><p align="center">(http://i.stack.imgur.com/whhD1.png)</p>
<br/>

### Safe Methods

리소스를 수정하지 않는 메소드들, `OPTIONS`, `GET`, `HEAD` 등을 *safe* 하다고 말한다. 대부분의 경우 *idempotent* 하면 *safe* 하다. 물론 예외도 있는데 `DELETE` 는 *idempotent* 하지만 리소스를 변경하므로 *safe* 하지 않다. 자세한 내용은 [RFC 7231: Safe Methods](http://tools.ietf.org/html/rfc7231#section-4.2) 를 보자. 참고로 [RFC 7231](http://tools.ietf.org/html/rfc7231#section-4.2.1) 은 `PUT`, `DELETE` 와 *safe methods* 를 *idempotent* 하다고 정의한다.


`HEAD` 는 Response-Body 없이 Header 만 얻기 위해 사용하고, `OPTIONS` 는 해당 리소스에 대해 가능한 operation 이 무엇인지 응답을 얻기 위해 사용한다. 만약 `OPTIONS` 에 대한 응답이 온다면 response `Allow` 에 가능한 operation 이 와야한다. [RFC2616](http://www.w3.org/Protocols/rfc2616/rfc2616-sec9.html) 에는 다음과 같이 나와있다.

> The `OPTIONS` method represents a request for information about the communication options available on the request/response chain identified by the Request-URI. This method allows the client to determine the options and/or requirements associated with a resource, or the capabilities of a server, without implying a resource action or initiating a resource retrieval. <br/><br/>
> Responses to this method are not cacheable.

### Cacheable Methods

왜 `OPTIONS` 메소드에 대한 응답은 캐시가 불가능한걸까? SO 에서 이 [답변](http://stackoverflow.com/questions/13073313/http-options-not-cacheable) 이 제일 나은것 같아서 가져왔다.

> The `OPTIONS` HTTP request returns the available methods which can be performed on a resource. (The objects methods)

> I can not say for certain why you can not cache the response, but its most likely a precaution. Caching would have little value for the `OPTIONS` http method.

> A Resource is "any information that can be given a name", that name is its URI. the response from the OPTIONs request is only a list of methods that can be requested on this resource (e.g. "GET PUT POST" maybe the response). To actually get at the information stored, you must use the GET method.

> History, more than anything; OPTIONS was defined that way to start with. The underlying reason is that HTTP caches are defined in terms of representations, which means the way you get something out of the cache is GET. This is why OPTIONS, PROPFIND, etc. caching are problematic.

다시 말해서, 리소스는 주어진 URI 에 대한 정보인데 `OPTIONS` 는 정보를 가지고 오는 것이 아니라, 정보에 대해 어떤 연산이 가능한지를 알려준다. HTTP 에서는 정보에 대해 캐싱하므로, `GET` 이나 `HEAD` 같이 정보를 돌려주는 연산만 캐싱할 수 있다.

### Trace, Connect

`TRACE` 는 클라이언트가 방금 보낸 요청을 다시 달라고, 서버에게 요청하는 것이고 `CONNECT` 는 HTTP 터널링을 할때 쓰인다. 중간의 프록시 서버를 위해서는 `CONNECT` 로 요청하고, 마지막 프록시에서 end-point 로는 `GET` 또는 `CONNECT` 를 날린다. `HTTPS` 라면 `CONNECT` 를, `HTTP` 라면 둘 중 아무거나 써도 상관 없다. 더 자세한건 [SO 답변](http://stackoverflow.com/questions/11697943/when-should-one-use-connect-and-get-http-methods-at-http-proxy-server) 으로

원문을 첨부하면,

> **CONNECT:** This method could allow a client to use the web server as a proxy.

> **TRACE:** This method simply echoes back to the client whatever string has been sent to the server, and is used mainly for debugging purposes. This method, originally assumed harmless, can be used to mount an attack known as Cross Site Tracing, which has been discovered by Jeremiah Grossman (see links at the bottom of the page).

### Summary

HTTP 메소드에 대해서 알아보았는데, 조금 더 찾아보니 HTTP 는 0.9 -> 1.0 -> 1.1 순으로 변화했다고 한다. 0.9 에선 `GET` 을 이용한 Read-only 버전이었고 1.0 에 들어와서야 `HEAD`, `POST` 등을 이용해 서버로 데이터 전송이 가능해졌다.   HTTP 1.1 (RFC 2616) 에 와서야 `DELETE`, `PUT` 등이 추가되면서 변경, 삭제까지 가능해졌다.

다음번에 HTTP 를 또 살펴 볼 일이 생긴다면, [RFC 7243: Caching](http://tools.ietf.org/html/rfc7234) 과 [RFC 7235: Authentication](http://tools.ietf.org/html/rfc7235) 에 대해서 보지 않을까 싶다.

### References

(1) [REST Cookbook: PUT vs POST](http://restcookbook.com/HTTP%20Methods/put-vs-post/)  
(2) [HTTP status code for UPDATE and DELETE](http://stackoverflow.com/questions/2342579/http-status-code-for-update-and-delete)  
(3) [PUT vs POST in REST](http://stackoverflow.com/questions/630453/put-vs-post-in-rest/18243587#18243587)  
(4) [REST Coookbook: idempotency](http://restcookbook.com/HTTP%20Methods/idempotency/)  
(5) [HTTP OPTIONS Method](http://zacstewart.com/2012/04/14/http-options-method.html)  
(6) [NO OPTIONS](https://www.mnot.net/blog/2012/10/29/NO_OPTIONS)  
(7) [HTTP History](http://www.mkexdev.net/Article/Content.aspx?parentCategoryID=1&categoryID=11&ID=119)  
(8) [HTTP OPTIONS not cacheable](http://stackoverflow.com/questions/13073313/http-options-not-cacheable)  
(9) [CONNECT vs GET in proxy](http://stackoverflow.com/questions/11697943/when-should-one-use-connect-and-get-http-methods-at-http-proxy-server)
