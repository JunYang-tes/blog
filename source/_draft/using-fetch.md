[原文](https://developer.mozilla.org/en-US/docs/Web/API/Fetch_API/Using_Fetch)

<!--
This kind of functionality was previously achieved using XMLHttpRequest. Fetch provides a better alternative that can be easily used by other technologies such as Service Workers. Fetch also provides a single logical place to define other HTTP-related concepts such as CORS and extensions to HTTP.

Note that the fetch specification differs from jQuery.ajax() in mainly two ways that bear keeping in mind:

The Promise returned from fetch() won’t reject on HTTP error status even if the response is an HTTP 404 or 500. Instead, it will resolve normally (with ok status set to false), and it will only reject on network failure or if anything prevented the request from completing.
By default, fetch won't send or receive any cookies from the server, resulting in (导致) unauthenticated requests if the site relies on maintaining a user session (to send cookies, the credentials init option must be set).

-->
以前由`XMLHttpRequest`来完成该功能。`Fetch`提供了一个更好的方式，这种方式也易于被像Service Workers之类的其它技术使用。`Fetch`也提供一个单一的地方来定义HTTP相关的概念，比如CORS。

注意，`Fetch`标准和jQuery.ajax有两个主要的不同，应记在心中：

+ fetch 返回的Promise不会因为HTTP的状态码属于错误一类的而reject掉，而是正常的resolve这个Promise（会将ok状态设置为false，注：这里应该指的是Promise resolve之后的值上的ok属性）。该Promise只会在网络请求失败或有其它阻止请求完成的事情的时候reject。

+ 默认情况下，fetch 不会发送和保存cookie，这会在某站点需要依靠用户会话的情况下导致未授权的请求。（为了发送cookie，`credentials`选项必须被设置）

<!--

# Making fetch requests

A basic fetch request is really simple to set up. Have a look at the following code:

var myImage = document.querySelector('img');

fetch('flowers.jpg').then(function(response) {
  return response.blob();
}).then(function(myBlob) {
  var objectURL = URL.createObjectURL(myBlob);
  myImage.src = objectURL;
});

-->

# 使用fetch来发请求
发送一个基本的Fetch是很简单的，看看下面的代码
```javascript

var myImage = document.querySelector('img');

fetch('flowers.jpg').then(function(response) {
  return response.blob();
}).then(function(myBlob) {
  var objectURL = URL.createObjectURL(myBlob);
  myImage.src = objectURL;
});

```
<!--
Here we are fetching an image across the network and inserting it into an <img> element. The simplest use of fetch() takes one argument — the path to the resource you want to fetch — and returns a promise containing the response (a Response object).

This is just an HTTP response of course, not the actual image. To extract the image body content from the response, we use the blob() method (defined on the Body mixin, which is implemented by both the Request and Response objects.)
-->
这里我们通过网络请求取回了一个图片并将其设置到一个图片元素上。这个极简单的`fetch`调用使用了一个参数，即图片资源的路径，然后它返回了一个promise对象，这个promise对象包含了响应对象。

<!--Note: The Body mixin also has similar methods to extract other types of body content; see the Body section for more.-->
>注：Body里还有一些类似的方法来抽取其它类型的数据，见下文Body一节
<!--
An objectURL is then created from the extracted Blob, which is then inserted into the img.

Fetch requests are controlled by the connect-src directive of Content Security Policy rather than the directive of the resources it's retrieving.
-->

从抽取出来的`Blob`对象创建了一个`objectURL`,然后把它设置到了`img`上。

Fetch请求是受内容安全策略（CSP，Content Security Policy）的`connect-src`控制的。

<!--
Supplying request options

The fetch() method can optionally accept a second parameter, an init object that allows you to control a number of different settings:


-->

# 使用请求选项
`fetch`方法可以接受第二个参数，这个参数是一个对象，你可以使用这个对象来控制许多不同的选项。
```javascript
var myHeaders = new Headers();

var myInit = { method: 'GET',
               headers: myHeaders,
               mode: 'cors',
               cache: 'default' };

fetch('flowers.jpg', myInit).then(function(response) {
  return response.blob();
}).then(function(myBlob) {
  var objectURL = URL.createObjectURL(myBlob);
  myImage.src = objectURL;
});
```
关于该参数的所有选项可参考[这里](https://developer.mozilla.org/en-US/docs/Web/API/WindowOrWorkerGlobalScope/fetch)

# 发送包含凭证的请求
<!--
To cause browsers to send a request with credentials included, even for a cross-origin call, add credentials: 'include' to the init object you pass to the fetch() method.
-->

在fetch的第二个参数中添加`credentials:'include'`可以让浏览器在请求（包括跨越请求）中包含用户凭证

```javascript
fetch('https://example.com', {
  credentials: 'include'  
})
```

<!--
If you only want to send credentials if the request URL is on the same origin as the calling script, add credentials: 'same-origin'.
-->
如果只希望在同一个域的时候包含用户凭证，则应当把`credentials`设为`same-origin`

```javascript
// The calling script is on the origin 'https://example.com'

fetch('https://example.com', {
  credentials: 'same-origin'  
})
```
<!--

To instead ensure browsers don’t include credentials in the request, use credentials: 'omit'.

-->
使用`omit`来确保浏览器不会在请求中包含凭证
```javascript

fetch('https://example.com', {
  credentials: 'omit'  
})
```
<!--
Checking that the fetch was successful
--->
# 检查fetch是否成功
<!--
A fetch() promise will reject with a TypeError when a network error is encountered or CORS is misconfigured on the server side, although this usually means permission issues or similar — a 404 does not constitute a network error, for example.  An accurate check for a successful fetch() would include checking that the promise resolved, then checking that the Response.ok property has a value of true. The code would look something like this:
-->
fetch调用返回的promise会在网络错误或CORS没配的的时候以TyepError的形式reject。404状态并不会导致网络错误。准确的检查fetch是否成功的方式是在该promise resolve之后来来`response.ok`的值，如：

```javascript
fetch('flowers.jpg').then(function(response) {
  if(response.ok) {
    return response.blob();
  }
  throw new Error('Network response was not ok.');
}).then(function(myBlob) { 
  var objectURL = URL.createObjectURL(myBlob); 
  myImage.src = objectURL; 
}).catch(function(error) {
  console.log('There has been a problem with your fetch operation: ' + error.message);
});
```
<!--
Supplying your own request object
-->
<!--
Instead of passing a path to the resource you want to request into the fetch() call, you can create a request object using the Request() constructor, and pass that in as a fetch() method argument:

-->
除了传一个路径给fetch方法，你也可以直接传一个request给它，你可以使用`Request`构造函数来创建request对象。
```javascript
var myHeaders = new Headers();

var myInit = { method: 'GET',
               headers: myHeaders,
               mode: 'cors',
               cache: 'default' };

var myRequest = new Request('flowers.jpg', myInit);

fetch(myRequest).then(function(response) {
  return response.blob();
}).then(function(myBlob) {
  var objectURL = URL.createObjectURL(myBlob);
  myImage.src = objectURL;
});
```
<!--
Request() accepts exactly the same parameters as the fetch() method. You can even pass in an existing request object to create a copy of it:
-->
`Request`和`fetch`拥有同样的参数，你甚至可以传递一个Request对象到Request构造函数中去创建一另一个Request对象。
```javascript
var anotherRequest = new Request(myRequest, myInit);
```
<!--
This is pretty useful, as request and response bodies are one use only. Making a copy like this allows you to make use of the request/response again, while varying the init options if desired.  The copy must be made before the body is read, and reading the body in the copy will also mark it as read in the original request.
-->

这是很有用的，因为request和response body都是一次性的。创建一个副本让你可以重用这个request。必须在读body之前创建这个副本，而且在副本中读body也会导致原请求的body被标记为已读。

<!--
There is also a clone() method that creates a copy. Both methods of creating a copy will fail if the body of the original request or response has already been read, but reading the body of a cloned response or request will not cause it to be marked as read in the original.
-->
>注：还有一个clone方法用来创建request的副本。这两种方式在原request或reponse body已读的情况下都会创建失败。但是以读clone 方式创建的副本的body不会导致原request和response被标记为已读。

