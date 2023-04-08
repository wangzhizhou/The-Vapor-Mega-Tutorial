
!!! tip "HTTP 协议"
    HTTP(Hyper Text Transfer Protocol)超文本传输协议，是Web的基础内容。

    有两个实体：客户端(Client)、服务端(Server)

    客户端向服务端发起请求，索要资源。服务端向客户端返回需要的资源。如果服务端满足不了客户端的需要，需要返回相关的说明。

    [HTTP 参考文档][http ref]

# HTTP 请求(Request)

请求包含几个部分：

!!! note inline end "HTTP 2.0"
    `HTTP 2.0`在效率和延迟上对`HTTP 1.1`进行了扩展，单个的请求和`HTTP 1.1`没有什么区别，但是`HTTP 2.0`的请求可以并行执行。服务器可以预测客户端的请求，从而在请求还没有发出的情况下把客户端可能需要的资源提前推送给客户端。 

    **Vapor支持HTTP 1.1和HTTP 2.0**

- 请求行。例如： `GET /about.html HTTP/1.1`, 它指定了请求的方式为`GET`，请求的资源uri是`/about.html`文件, 使用的传输协议是`HTTP`，传输协议的版本号是`1.1`

- 主机名。可能有多个服务器共用同一个IP地址，为了区别它们，主机名被用来处理这种情况。

- 请求头。包含请求时添加的额外信息：`Authorization`、 `Accept`、`Cache-Control`、`Content-Length`、`Content-Type`等。请求头是键-值对的集合。
  
- 可选请求数据。例如： POST方式请求可以携带一些请求数据。

客户端发起请求时可以用多种方式进行选择:

- [GET](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Methods/GET)
- [HEAD](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Methods/HEAD)
- [POST](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Methods/POST)
- [PUT](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Methods/PUT)
- [DELETE](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Methods/DELETE)
- [CONNECT](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Methods/CONNECT)
- [OPTIONS](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Methods/OPTIONS)
- [TRACE](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Methods/TRACE)
- [PATCH](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Methods/PATCH)

!!! tip
    Web浏览器只能使用GET和POST两种方式的请求，并且不能修改请求头，要修改请求头只能依赖`java script`。
    
    其它客户端像iOS应用可以使用所有的HTTP请求方式，更加灵活。

# HTTP 响应(Response)

响应包含几个部分： 

!!! tip inline end "HTTP响应码"
    - [标准响应码列表参考][http status code]
-  状态行。包含状态码、协议及其版本号、信息文本。状态码和状态信息文本表明客户端请求的结果是否成功。状态码按第一位数字分成五类：
    - 1 - 信息响应
    - 2 - 请求成功，正常返回。例如： 200 OK。 204表示请求成功，但返回内容为空。
    - 3 - 重定向响应。响应的内容是在请求指向的服务器之外的其它服务器拿到后返回的。
    - 4 - 客户端请求错误。418是一个著名的状态码，有着特殊的意义
    - 5 - 服务端错误。
-  响应头。例如: `Set-cookie`、`WWW-Authenticate`、`Cache-Control`、`Content-Length`等。响应头与请求头类似，也是由键值对(key-value)组成的。
-  可选响应体。带着响应返回的数据，可以是HTML页面内容、图片或者JSON数据


[http ref]: <https://developer.mozilla.org/zh-CN/docs/Web/HTTP>
[http status code]: <https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Status>