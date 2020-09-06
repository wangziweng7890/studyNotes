## TCP/IP通信传输流

计算机网络需要通信，双方就必须基于相同的方法。像这样把与互联网关联的协议集合起来的总称为TCP/IP。

TCP/IP协议簇按层次分为以下四层

- 应用层：向用户提供应用服务时通信的活动。
- 传输层：提供处于网络连接中的两台计算机之间的数据传输；确保传输完整性，进行数据分割，并在各个报文上打上标记序号。
- 网络层：处理网络中流通的数据包。该层提供了怎么样的传输路线到目的计算机。搜索IP地址，一边搜索，一边传输。
- 链路层：与硬件相关。

![](.\TCP-IP传输流.png)

​	从上图可知，利用TCP/IP协议簇进行通信时，发送端从应用层往下走，接收端从链路层往上走。

## 各种协议与HTTP协议的关系

![](.\HTTP协议与各种协议的关系.png)



## HTTP1.0和HTTP1.1支持的请求方法

| 方法    | 说明                                    | 支持的http协议版本 |
| ------- | --------------------------------------- | ------------------ |
| GET     | 获取资源                                | 1.0、1.1           |
| POST    | 传输实体主体                            | 1.0、1.1           |
| PUT     | 传输文件                                | 1.0、1.1           |
| DELETE  | 删除文件                                | 1.0、1.1           |
| HEAD    | 获取报文首部                            | 1.0、1.1           |
| OPTIONS | 询问支持的方法，响应首部会带上Allow字段 | 1.1                |
| TRACE   | 追踪路径                                | 1.1                |
| CONNECT | 要求用隧道协议链接代理                  | 1.1                |
| LINK    | 建立和资源间的联系                      | 1.0                |
| UNLINK  | 断开链接关系                            | 1.0                |

备注：TRACE配合首部Max-Forwards字段，可以用来追查网络传输中问题。调查是哪台代理服务器出了问题。Max-Forwards是指定最多转发几次，到了最大次数后直接返回结果。



## 状态码类别

|      | 类别                           | 原因短语                 |
| ---- | ------------------------------ | ------------------------ |
| 1XX  | Informational(信息性状态码)    | 接受的请求正在处理       |
| 2XX  | Success(成功状态码)            | 请求正常处理完毕         |
| 3XX  | Redirection(重定向状态码)      | 需要进行附加操作完成请求 |
| 4XX  | Client Error(客户端错误状态码) | 服务器无法处理请求       |
| 5XX  | Server Error(服务器错误状态码) | 服务器处理请求出错       |

## 细类别

| 状态码                    | 描述                                                         |
| ------------------------- | ------------------------------------------------------------ |
| 200 ok                    | 成功                                                         |
| 204  No Content           | 请求处理成功，但没有资源可以返回                             |
| 206 Partial Content       | 进行 范围请求 成功后返回的状态码                             |
| 301 Moved Permanently     | 永久性重定向，资源的URI已重新分配地址，希望更新书签          |
| 302 Found                 | 临时重定向，希望本次请求用新的URI访问。（登录页跳转配合首部Location字段） |
| 303 See Other             | 同302，但是明确表示客户端应使用GET方法获取资源               |
| 304 Not Modified          | 服务器资源未改变，可直接使用客户端未过期的缓存               |
| 307 Temporary Redirect    | 同302，但是不会从POST变成GET                                 |
| 400 Bad Request           | 请求报文中语法错误                                           |
| 401 Unauthorized          | 本页面需要通过HTTP认证                                       |
| 403 Forbidden             | 禁止访问，一般是未获得文件访问授权                           |
| 404 Not Found             | 地址错误，找不到该资源                                       |
| 422 Unprocessable Entity  | 一般是参数校验失败                                           |
| 500 Internal Server Error | 服务端执行错误                                               |
| 503 Service Unavaliable   | 表明服务器处于超负荷或者停机维护阶段                         |

301,302,303相应状态码返回时，几乎所有浏览器都会把POST改成GET。但其实301,302的标准是禁止将POST改为GET的。所以只有307不会从POST变成GET；

401 响应体应包含WWW-Authenticate的首部，浏览器初步接触到401相应，会弹出认证用的对话窗口



## HTTP报文

HTTP报文由报文首部，空行（CR+LF），报文主体（想要获取的资源）构成。

报文首部又分为

- 请求行：包含请求方法，URI，HTTP版本。
- 状态行：包含状态码，原因短语和HTTP版本
- 首部字段：包含请求和响应的各种条件以及属性
  - 通用首部：请求和响应共同具有的属性
  - 请求首部
  - 响应首部
  - 实体首部：和报文实体相关的属性（补充了资源内容的更新时间及相关信息）
- 其它

## HTTP/1.1 首部字段一览

### **通用首部字段**

| 首部字段名                       | 说明                       |
| -------------------------------- | -------------------------- |
| Cache-Control                    | 控制缓存行为               |
| Connection                       | 逐条首部，连接的管理       |
| Date                             | 创建报文的日期时间         |
| Pragma                           | 报文指令                   |
| Trailer                          | 报文末端的首部一览         |
| Transfer-Encoding                | 指定报文主体的传输编码方式 |
| Upgrate（例：Upgrate:Websocket） | 升级为其他协议             |
| Via(例Via: proxy1,proxy2)        | 代理服务器的相关信息       |
| Warning                          | 错误通知                   |

**详细描述：**

1.Cache-Control

**例：**

```http
Cache-Control: private,max-age=0,no-cache
```

**缓存请求指令**

| 指令            | 参数   | 说明                   |
| --------------- | ------ | ---------------------- |
| no-cache        | 无     | 强制向源服务器再次验证 |
| no-store        | 无     | 不缓存                 |
| max-age=[秒]    | 必须   | 响应的最大Age值        |
| max-stale(=秒)  | 可省略 | 能容忍的最大过期时间   |
| min-fresh=[秒]  | 必须   | 能容忍的小新鲜度       |
| no-transform    | 无     | 代理不可更改媒体类型   |
| only-if-cached  | 无     | 从缓存获取资源         |
| cache-extension | -      | 新指令标记（token）    |

**缓存响应指令**

| 指令             | 参数   | 说明                                               |
| ---------------- | ------ | -------------------------------------------------- |
| public           | 无     | 可向任意方提供响应的缓存                           |
| private          | 可省略 | 仅向特定用户返回响应                               |
| no-cache         | 可省略 | 强制向服务器确认是否采用缓存                       |
| no-store         | 无     | 不缓存请求或响应任何内容                           |
| no-transform     | 无     | 不可更改媒体类型                                   |
| must-revalidate  | 无     | 可缓存但必须再向服务器确认                         |
| proxy-revalidate | 无     | 要求缓存服务器对缓存有效性在确认                   |
| max-age=[秒]     | 必需   | 缓存最大时间                                       |
| s-maxage=[秒]    | 必需   | 代理服务器的缓存最大时间（会忽略max-age及expires） |
| cache-extension  | -      | 新指令标记（token）                                |

**注意：**请求和响应相同的字段含义可能有所不同

#### 浏览器缓存一览

<img src=".\浏览器缓存机制.webp"  />

**2.Connection**

Connection首部字段具备如下两个功能

- 控制不再转发的首部字段

  ```
  客户端首部
  GET / HTTP/1.1
  Upgrade: HTTP/1.1
  Connection: Upgrate //告诉代理服务器不再转发的字段
  代理服务器删除字段后转发
  GET / HTTP/1.1
  ```

- 持久化连接

  ```
  GET / HTTP/1.1
  Connection: Keep-Alive
  ```



**3.Pragma**

HTTP1.1之前版本遗留字段，仅做兼容使用

使用的唯一形式

```
Pragma: no-cache
```

**4.Upgrade**

用来使用不同的协议进行通信，仅限于邻接服务器,所以一般带上Connection,对于此请求，服务器一般返回101 Switching Protocols 状态码

```
GET /index.html HTTP/1.1
Upgrade: web-socket
Connection: web-socket
```



### **请求首部字段**

| 首部字段名          | 说明                                              |
| ------------------- | ------------------------------------------------- |
| Accept              | 用户代理可处理的媒体类型                          |
| Accept-Charset      | 优先的字符集                                      |
| Accept-Encoding     | 优先的内容编码                                    |
| Accept-Language     | 优先的语言                                        |
| Authorization       | Web认证信息                                       |
| Expect              | 期待服务器的特定行为                              |
| From                | 用户的电子邮箱地址                                |
| Host                | 请求资源所在的服务器                              |
| if-Match            | 比较实体标记（ETag）                              |
| if-Modified-Since   | 比较资源的更新时间                                |
| if-None-Match       | 比较实体标记（与if-Match相反）                    |
| if-Range            | 资源未更新时发送实体Byte的范围请求                |
| if-Unmodified-Since | 比较资源的更新时间（与if-modified-Since相反）     |
| Max-Forwards        | 最大逐跳数                                        |
| Proxy-Authorization | 代理服务器要求客户端的认证信息                    |
| Range               | 实体的字节范围请求                                |
| Referer             | 对请求中URI的原始获取方,知道URI是从哪个页面发起的 |
| TE                  | 传输编码的优先级                                  |
| User-Agent          | HTTP客户端程序的信息                              |

**字段详解**：

1.If-Range

If-Range的值若跟Etag值或更新日期值一致，则返回范围请求206 Partial，否则返回忽略范围请求，返回全部资源

```
GET / HTTP/1.1
If-Range: "123456",
Range: bytes=5001-10000
```

### **响应首部字段**

| 首部字段名         | 说明                                 |
| ------------------ | ------------------------------------ |
| Accept-Ranges      | 是否接受字节范围请求                 |
| Age                | 推算字节创建经过时间                 |
| ETag               | 资源的匹配信息                       |
| Location           | 令客户端重定向至指定URI              |
| Proxy-Autnenticate | 代理服务器对客户端的认证信息         |
| Retry-After        | 对再次发起请求的时机要求             |
| Server             | HTTP服务器的安装信息                 |
| Vary               | 代理服务器缓存的管理信息             |
| WWW-Authenticate   | 服务器对客户端的认证信息（基本不用） |

**字段详解：**

1.Vary

Vary 字段指定的字段一致，才能从缓存返回，否则从源服务器返回

```
Vary: Accept-Language //指定自然语言一致
```

### **实体首部字段**

| 首部字段名       | 说明                                     |
| ---------------- | ---------------------------------------- |
| Allow            | 资源支持的HTTP方法                       |
| Content-Encoding | 实体主体适应的编码方式                   |
| Content-Language | 实体主体的自然语言                       |
| Content-Length   | 实体主体的大小（单位:字节）              |
| Content-Location | 替代对应资源的URI                        |
| Content-MD5      | 实体主体的报文摘要，判断报文的传输完整性 |
| Content-Type     | 实体主体的媒体类型                       |
| Expires          | 实体主体的过期日期时间                   |
| Last-Modified    | 资源的最后修改日期时间                   |
| Content-Range    | 实体主体的位置范围                       |

### **非HTTP/1.1首部字段**

例如 Cookie,Set-Cookie,Content-Disposition

#### **Set-Cookie字段的属性**

| 属性         | 说明                                               |
| ------------ | -------------------------------------------------- |
| NAME=VALUE   | 赋予Cookie的名称及值                               |
| expires=DATE | Cookie的有效期（若不指定则浏览器关闭则清除）       |
| path=PATH    | Cookie的适用路径，将服务器上的文件路径作为适用对象 |
| domain=域名  | 作为cookie适用对象的域名                           |
| Secure       | 仅在HTTPS安全通信时才会发送Cookie                  |
| HttpOnly     | 使得Cookie无法通过js访问                           |
| SameSite     | 可以限制第三方Cookie                               |

**详解：**

1.SameSite有三个值

- Strict：完全禁止第三方Cookie,跨站点时，任何情况都不会发送Cookie

- Lax：只支持三种导航到目标网址的GET请求携带

  - 链接：<a href="..."></a>
  - 预加载：<link rel="prerender" href="..."/>
  - GET表单：<form method="GET" action="...">

- None：允许携带，但是必须和Secure同时设置 

  - ```
    Set-Cookie: widget_session=abc123; SameSite=None; Secure
    ```

    

## HTTPS

​	HTTP加上加密技术和认证以及完整性保护及是HTTPS

​	HTTPS并非应用层的一种新协议，只是HTTP通信接口部分用SSL和TLS协议代替而已。

​	通常HTTP直接和TCP通信。当使用SSL时，HTTP先和SSL通信，再由SSL和TCP通信。

​	SSL协议是独立于HTTP的应用层协议，所以其它应用层协议也可以配合SSL使用。

**共享密钥加密(对称加密)**

加密解密用同一个密钥，但是无法保证传输密钥过程密钥不被窃听，失去了通信加密的意义。

**使用两把密钥加密的公开密钥加密（非对称加密）**

将公钥传给对方，让对方使用公钥对通信内容进行加密后传输过来，自己在用私钥解密。因为私钥保存在本地，所以不存在通信劫持风险。

**HTTPS采取混合加密机制**

因为公开密钥加密处理速度慢2-100倍，因此在交换密钥环节进行公开加密，通信交换报文阶段使用共享秘钥加密。

**证明公开密钥的正确性**

​	公开密钥无法证实是货真价实的公开密钥，可能被中间人攻击，换成自己的公钥。

​	为了解决上述问题。可以使用数字证书认证机构颁发的公开密钥证书。认证证书会事先植入到浏览器中。

**HTTPS的通信步骤**

1. 客户端向服务端发送Client Hello报文开始SSL通信。该报文带有客户端支持的SSL版本以及加密组件
2. 服务器端向客户端发送Server Hello报文，确定通信所用SSL版本和加密组件
3. 紧接着服务器端继续向客户端发送Certificate报文，该报文含有公开密钥证书
4. 然后服务器端向客户端发送Server Hellw Done，标志着第一次SSL握手协商部分结束
5. 结束后客户端发送Client key exchange报文作为回应，该报文含有一种Pre-master secret的随机密码串，该密码串由服务端传送过来的公开密钥加密。
6.  然后客户端继续发送 change cipher spec报文告诉服务器端以后用Pre-master secret密钥进行加密解密
7. 最后客户端发送Finish报文，该报文拥有之前报文的校验值
8. 服务器端同样发送change cipher spec
9. 服务器端同样发送Finish
10. 客户端和服务器端的finish报文交换完毕，就算完成了SSL请求。可以开始HTTP请求了。



## HTTP/2简介

HTTP/2主要是为了解决现HTTP 1.1性能不好的问题才出现的。当初Google为了提高HTTP性能，做出了SPDY，它就是HTTP/2的前身，后来也发展成为HTTP/2的标准。

HTTP/2兼容HTTP 1.1，例如HTTP Method，Status code，URI以及大部分Header Fields。

HTTP/2通过以下方法减少latency，用来改进页面加载的速度，

1. HTTP Header的压缩，采用的是HPack算法。
2. HTTP/2的Server Push，非常重要的一个特性。
3. 请求的pipeline。
4. 修复在HTTP 1.x的队头阻塞问题。
5. 在单个TCP连接里多工复用请求。

HTTP/2支持HTTP 1.1里的大部分use case，例如桌面浏览器、移动浏览器、Web API、Web Server、代理服务器、反向代理服务器、防火墙和CDN等。