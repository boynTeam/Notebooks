HTTP协议以面试题的方式给出

# HTTP的各个版本区别

## HTTP0.9

0.9十分不完善,其不支持请求头,只支持GET方法

## HTTP1.0

1.0版本在HTTP请求的首行中加入了HTTP版本

加入了请求和响应中的首部字段

增加了状态码

可以传输除了文本文件以外的其他文件

## HTTP1.1

1.1主要解决了网络的问题.可以复用TCP连接,无需每次发一次请求响应就建立一次TCP连接.这个称之为HTTP长连接

支持流水线传输,一个请求发送出去了,无需等待响应就可以接着发第二个请求

增加了缓存控制的机制,可以在头部中指定`cache-control`字段命令浏览器是否缓存该请求

## HTTP2

可以并行发送多个HTTP请求

允许服务端进行PUSH,即主动对客户端做cache

# HTTP各个方法

GET: 通常用于请求服务器发送某些资源

HEAD: 请求资源的头部信息, 并且这些头部与 HTTP GET 方法请求时返回的一致. 该请求方法的一个使用场景是在下载一个大文件前先获取其大小再决定是否要下载, 以此可以节约带宽资源

OPTIONS: 用于获取目的资源所支持的通信选项

POST: 发送数据给服务器

PUT: 用于新增资源或者使用请求中的有效负载替换目标资源的表现形式

DELETE: 用于删除指定的资源

PATCH: 用于对资源进行部分修改

CONNECT: HTTP/1.1协议中预留给能够将连接改为管道方式的代理服务器

TRACE: 回显服务器收到的请求，主要用于测试或诊断

# GET和POST有什么区别

数据传输方式不同：GET请求通过URL传输数据，而POST的数据通过请求体传输。

安全性不同：POST的数据因为在请求主体内，所以有一定的安全性保证，而GET的数据在URL中，通过历史记录，缓存很容易查到数据信息。

数据类型不同：GET只允许 ASCII 字符，而POST无限制

GET无害： 刷新、后退等浏览器操作GET请求是无害的，POST可能重复提交表单

特性不同：GET是安全（这里的安全是指只读特性，就是使用这个方法不会引起服务器状态变化）且幂等（幂等的概念是指同一个请求方法执行多次和仅执行一次的效果完全相同），而POST是非安全非幂等

# HTTP报文结构

HTTP的报文比较简单

第一行是指定HTTP方法,URL和HTTP的版本

后面若干行指定HTTP的头部信息

如果是POST等有消息体的方法,就会再空一行,放入传输的信息

![](http://imageblog.boyn.top/202002012044_667.png)

# HTTP常用头部

## 通用头部

`Cache-Control`控制缓存

`Connection`管理连接的方式,断开的时间

`Transfor-Encoding`报文主体的编码格式

## 请求头部

`Accept`客户端可以处理的媒体类型

`Authorization`认证信息

`Host`请求资源所在服务器

`User-Agent`客户端的程序信息

## 响应头部

`Location`令客户端重定向的URL

`ETag`表示返回资源的唯一字符串

`Server`服务器信息

# HTTP常用状态码

## 2XX 成功

*200* OK 表示请求被正确处理并返回响应

*206* 进行范围请求

## 3XX 重定向

*301* 永久性重定向 表示该资源已经被分配了新的URL

*302* 临时性重定向 表示该资源被临时分配了新的URL

*304* 表示服务器允许访问资源,但发生请求未满足条件的情况

## 4XX 客户端错误

*400* Bad Request 表示报文存在语法上的错误

*401* Unauthorized 未认证

*403* Forbidden 表示请求资源的请求被服务器拒绝

*404* Not Found 表示请求的资源不存在

*408* Request Timeout 请求资源超时

## 5XX服务器错误

*500* Server Internal Error 服务器在执行的时候出现了错误

# Http Keep-alive字段

在HTTP1.0中,每一次的请求和响应对应着一个TCP的连接和断开,这样无疑十分费时和耗费资源,为了减少消耗,就要对TCP连接进行复用,在`Connection`字段中设置为`keep-alive`使得TCP连接可以被复用

# GET 和 POST 比较

## 作用

GET 用于获取资源，而 POST 用于传输实体主体。

## 参数

GET 和 POST 的请求都能使用额外的参数，但是 GET 的参数是以查询字符串出现在 URL 中，而 POST 的参数存储在实体主体中。不能因为 POST 参数存储在实体主体中就认为它的安全性更高，因为照样可以通过一些抓包工具（Fiddler）查看。

因为 URL 只支持 ASCII 码，因此 GET 的参数中如果存在中文等字符就需要先进行编码。例如 `中文` 会转换为 `%E4%B8%AD%E6%96%87`，而空格会转换为 `%20`。POST 参数支持标准字符集。

```
GET /test/demo_form.asp?name1=value1&name2=value2 HTTP/1.1
POST /test/demo_form.asp HTTP/1.1
Host: w3schools.com
name1=value1&name2=value2
```

## 安全

安全的 HTTP 方法不会改变服务器状态，也就是说它只是可读的。

GET 方法是安全的，而 POST 却不是，因为 POST 的目的是传送实体主体内容，这个内容可能是用户上传的表单数据，上传成功之后，服务器可能把这个数据存储到数据库中，因此状态也就发生了改变。

安全的方法除了 GET 之外还有：HEAD、OPTIONS。

不安全的方法除了 POST 之外还有 PUT、DELETE。

## 幂等性

幂等的 HTTP 方法，同样的请求被执行一次与连续执行多次的效果是一样的，服务器的状态也是一样的。换句话说就是，幂等方法不应该具有副作用（统计用途除外）。

所有的安全方法也都是幂等的。

在正确实现的条件下，GET，HEAD，PUT 和 DELETE 等方法都是幂等的，而 POST 方法不是。

GET /pageX HTTP/1.1 是幂等的，连续调用多次，客户端接收到的结果都是一样的：

```
GET /pageX HTTP/1.1
GET /pageX HTTP/1.1
GET /pageX HTTP/1.1
GET /pageX HTTP/1.1
```

POST /add_row HTTP/1.1 不是幂等的，如果调用多次，就会增加多行记录：

```
POST /add_row HTTP/1.1   -> Adds a 1nd row
POST /add_row HTTP/1.1   -> Adds a 2nd row
POST /add_row HTTP/1.1   -> Adds a 3rd row
```

DELETE /idX/delete HTTP/1.1 是幂等的，即使不同的请求接收到的状态码不一样：

```
DELETE /idX/delete HTTP/1.1   -> Returns 200 if idX exists
DELETE /idX/delete HTTP/1.1   -> Returns 404 as it just got deleted
DELETE /idX/delete HTTP/1.1   -> Returns 404
```

## 可缓存

如果要对响应进行缓存，需要满足以下条件：

- 请求报文的 HTTP 方法本身是可缓存的，包括 GET 和 HEAD，但是 PUT 和 DELETE 不可缓存，POST 在多数情况下不可缓存的。
- 响应报文的状态码是可缓存的，包括：200, 203, 204, 206, 300, 301, 404, 405, 410, 414, and 501。
- 响应报文的 Cache-Control 首部字段没有指定不进行缓存。

## XMLHttpRequest

为了阐述 POST 和 GET 的另一个区别，需要先了解 XMLHttpRequest：

> XMLHttpRequest 是一个 API，它为客户端提供了在客户端和服务器之间传输数据的功能。它提供了一个通过 URL 来获取数据的简单方式，并且不会使整个页面刷新。这使得网页只更新一部分页面而不会打扰到用户。XMLHttpRequest 在 AJAX 中被大量使用。

- 在使用 XMLHttpRequest 的 POST 方法时，浏览器会先发送 Header 再发送 Data。但并不是所有浏览器会这么做，例如火狐就不会。
- 而 GET 方法 Header 和 Data 会一起发送。

