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
```
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