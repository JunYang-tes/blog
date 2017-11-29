<!--The HTTP OPTIONS method is used to describe the communication options for the target resource. The client can specify a specific URL for the OPTIONS method, or an asterisk (*) to refer to the entire server.
-->
HTTP OPTIONS方法用来描述对目标资源的通信选项。客户端可以指定一个为OPTIONS方法特殊的URL，或用*来表示整个服务器。

<!--
Request has body	No
Successful response has body	Yes
Safe	Yes
Idempotent	Yes
Cacheable	No
Allowed in HTML forms
-->

|||
|-|-|
|请求有body|No|
|成功的响应有body|Yes|
|安全|Yes|
|可缓存|No|
|可在HTML表单中使用|No|

# 语法

```
OPTIONS /index.html HTTP/1.1
OPTIONS * HTTP/1.1
```

# 示例

## 辨别支持的请求方法


为了找出服务器支持的请求方法，可以使用`curl`来发一个`OPTIONS`请求：
```shell
curl -X OPTIONS http://example.org -i
```
>注：`-i` 即`--include`,表示打印响应的header

服务器的响应包括一个`Allow`header，表示受支持的方法。

```
HTTP/1.1 200 OK
Allow: OPTIONS, GET, HEAD, POST
Cache-Control: max-age=604800
Date: Thu, 13 Oct 2016 11:45:00 GMT
Expires: Thu, 20 Oct 2016 11:45:00 GMT
Server: EOS (lax004/2813)
x-ec-custom-error: 1
Content-Length: 0
```

## 作为CORS场景的预请求
<!--
In CORS, a preflight request with the OPTIONS method is sent, so that the server can respond whether it is acceptable to send the request with these parameters. The Access-Control-Request-Method header notifies the server as part of a preflight request that when the actual request is sent, it will be sent with a POST request method. The Access-Control-Request-Headers header notifies the server that when the actual request is sent, it will be sent with a X-PINGOTHER and Content-Type custom headers.  The server now has an opportunity to determine whether it wishes to accept a request under these circumstances.
-->
在跨域的情况下，会预先发一个OPTIONS的请求，那么服务器就可以告诉客户端是否可用这些参数来发送请求。（下例）OPTIONS 请求中的`Access-Control-Request-Method` 这个Header通知服务器，后面正式的请求会使用POST这个方法。`Access-Control-Request-Headers` 这个Header告诉服务器，后面的正式的请求需有`X-PINGOTHER`和`Content-Type`这两个响应头。服务器可根据这些信息来决定它是否希望收到后面这个正式的请求。
```
OPTIONS /resources/post-here/ HTTP/1.1 
Host: bar.other 
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8 
Accept-Language: en-us,en;q=0.5 
Accept-Encoding: gzip,deflate 
Accept-Charset: ISO-8859-1,utf-8;q=0.7,*;q=0.7 
Connection: keep-alive 
Origin: http://foo.example 
Access-Control-Request-Method: POST 
Access-Control-Request-Headers: X-PINGOTHER, Content-Type
```
<!--
The server responds with Access-Control-Allow-Methods and says that POST, GET, and OPTIONS are viable methods to query the resource in question. This header is similar to the Allow response header, but used strictly within the context of CORS.
-->
服务器的响应中的`Access-Control-Allow-Methods`表明`POST`、`GET`和`OPTIONS`对于该资源(即OPTIONS中的 /resources/post-here/)的访问是被允许的。

```
HTTP/1.1 200 OK
Date: Mon, 01 Dec 2008 01:15:39 GMT 
Server: Apache/2.0.61 (Unix) 
Access-Control-Allow-Origin: http://foo.example 
Access-Control-Allow-Methods: POST, GET, OPTIONS 
Access-Control-Allow-Headers: X-PINGOTHER, Content-Type 
Access-Control-Max-Age: 86400 
Vary: Accept-Encoding, Origin 
Content-Encoding: gzip 
Content-Length: 0 
Keep-Alive: timeout=2, max=100 
Connection: Keep-Alive 
Content-Type: text/plain
```