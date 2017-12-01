[原文](https://developer.mozilla.org/en-US/docs/Web/API/Fetch_API/Using_Fetch)

<!--
This kind of functionality was previously achieved using XMLHttpRequest. Fetch provides a better alternative that can be easily used by other technologies such as Service Workers. Fetch also provides a single logical place to define other HTTP-related concepts such as CORS and extensions to HTTP.

Note that the fetch specification differs from jQuery.ajax() in mainly two ways that bear keeping in mind:

The Promise returned from fetch() won’t reject on HTTP error status even if the response is an HTTP 404 or 500. Instead, it will resolve normally (with ok status set to false), and it will only reject on network failure or if anything prevented the request from completing.
By default, fetch won't send or receive any cookies from the server, resulting in unauthenticated requests if the site relies on maintaining a user session (to send cookies, the credentials init option must be set).

-->
以前由`XMLHttpRequest`来完成该功能。`Fetch`提供了一个更好的方式，