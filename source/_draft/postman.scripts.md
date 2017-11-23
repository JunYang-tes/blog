# 设置变量
# Test scripts
<!--
With Postman you can write and run tests for each request using the JavaScript language.

-->
用Postman 你可以用JavaScript为每个请求编写和执行一些脚本。

![](https://s3.amazonaws.com/postman-static-getpostman-com/postman-docs/randomFullTests2.png)

# 应用
有一些API需要加上授权Header，这个Header是一个bearar的token。为了方便可将这个token设置为变量，在其它需要授权token的地方引用该变量，那么就可以只用修改这个变量来达到修改所有需要使用它的请求的Header的目的。
但这样一来，在对接口进行访问的时候需要两个步骤，一是获取Token，二是修改变量。而且修改变量的操作步骤还挺多。好在我们可以使用上面的Test script来一步完成这两项工作。
```javascript
var ret = JSON.parse(responseBody)
postman.setEnvironmentVariable("bearer",ret.access_token)
```
https://www.getpostman.com/docs/postman/environments_and_globals/variables
https://www.getpostman.com/docs/postman/scripts/test_examples