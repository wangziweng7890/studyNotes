postman集成了一个强大的，基于nodejs的script引擎，借助它，您可以为requests和collections添加动态的行为

1. 在发送request之前，编写pre-request script，定制化request。
2. 收到response之后，用test script，处理返回的数据。

Pre-request script => request=>response=>Tests script

例如：在egg中，为了防止第三方网站进行跨域请求，需要我们在请求头中需要带上x-csrf-token字段，否则会报403 forbidden；而此字段会放在在正常的http请求中的response中的cookie中。

解决方法：

```js
let csrf_token = postman.getResponseCookie("csrftoken").value;//从cookie中拿到此字段
postman.clearGlobalVariable("csrftoken");//先清除掉此全局变量
postman.setGlobalVariable("csrftoken", csrf_token);//在重新给此全局变量设值
```

然后我们在Headers中添加字段即可

| key          | value         |
| ------------ | ------------- |
| x-csrf-token | {{csrftoken}} |

注意：通过{{}}可以拿到全局变量