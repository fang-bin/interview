## HTTP

#### HTTP简介
HTTP协议是Hyper Text Transfer Protocol（超文本传输协议）的缩写,是用于从万维网（WWW:World Wide Web ）服务器传输超文本到本地浏览器的传送协议。。

HTTP是一个基于TCP/IP通信协议来传递数据（HTML 文件, 图片文件, 查询结果等）。

HTTP协议工作于客户端-服务端架构上。浏览器作为HTTP客户端通过URL向HTTP服务端即WEB服务器发送所有请求。

HTTP默认端口号为80，但是你也可以改为8080或者其他端口。

* **HTTP是无连接** 无连接的含义是限制每次连接只处理一个请求。服务器处理完客户的请求，并收到客户的应答后，即断开连接。采用这种方式可以节省传输时间。
* **HTTP是媒体独立的** 这意味着，只要客户端和服务器知道如何处理的数据内容，任何类型的数据都可以通过HTTP发送。客户端以及服务器指定使用适合的MIME-type内容类型。
* **HTTP是无状态** HTTP协议是无状态协议。无状态是指协议对于事务处理没有记忆能力。缺少状态意味着如果后续处理需要前面的信息，则它必须重传，这样可能导致每次连接传送的数据量增大。另一方面，在服务器不需要先前信息时它的应答就较快。


#### HTTP消息结构

**请求消息**
客户端发送一个HTTP请求到服务器的请求消息包括以下格式：请求行（request line）、请求头部（header）、空行和请求数据四个部分组成，下图给出了请求报文的一般格式。
![response](https://github.com/fang-bin/interview/blob/master/image/request.png)

**响应消息**
HTTP响应也由四个部分组成，分别是：状态行、消息报头、空行和响应正文。
![request](https://github.com/fang-bin/interview/blob/master/image/response.jpg)

#### HTTP请求的方法
HTTP/1.0定义了三种请求方法： GET, POST 和 HEAD方法。

HTTP/1.1新增了六种请求方法：OPTIONS、PUT、PATCH、DELETE、TRACE 和 CONNECT 方法。

| 方法 | 描述 |
| --- | --- |
| GET | 请求指定的页面信息，并返回实体主体。 |
| HEAD | 类似于 GET 请求，只不过返回的响应中没有具体的内容，用于获取报头 |
| POST | 向指定资源提交数据进行处理请求（例如提交表单或者上传文件）。数据被包含在请求体中。POST 请求可能会导致新的资源的建立和/或已有资源的修改。 |
| PUT | 从客户端向服务器传送的数据取代指定的文档的内容。 |
| DELETE | 请求服务器删除指定的页面。 |
| CONNECT | HTTP/1.1 协议中预留给能够将连接改为管道方式的代理服务器。 |
| OPTIONS | 允许客户端查看服务器的性能。(fetch中都会先发送一个options的预请求) |
| TRACE | 回显服务器收到的请求，主要用于测试或诊断。|
| PATCH | 是对 PUT 方法的补充，用来对已知资源进行局部更新 。 |

#### HTTP响应头信息

| 响应头 | 说明 |
| --- | --- |
| Allow | 服务器支持哪些请求方法（如GET,POST）|
| Content-Encoding | 文档编码 |
| Content-Length	| 内容长度 |
| Content-Type | 文档属于什么MIME类型。 Servlet默认为text/plain，但通常需要显式地指定为text/html。由于经常要设置Content-Type，因此HttpServletResponse提供了一个专用的方法setContentType。|
| Date | 当前的GMT时间 |
| Expires	| 文档过期时间，过期之后则不再缓存该文档 |
| Last-Modified | 文档的最后改动时间。客户可以通过If-Modified-Since请求头提供一个日期，该请求将被视为一个条件GET，只有改动时间迟于指定时间的文档才会返回，否则返回一个304（Not Modified）状态。Last-Modified也可用setDateHeader方法来设置。 |
| Location | 表示客户应当到哪里去提取文档。Location通常不是直接设置的，而是通过HttpServletResponse的sendRedirect方法，该方法同时设置状态代码为302。|
| Refresh | 表示浏览器应该在多少时间之后刷新文档，以秒计。除了刷新当前文档之外，你还可以通过setHeader("Refresh", "5; URL=http://host/path")让浏览器读取指定的页面。 | 
| Server | 服务器名字。Servlet一般不设置这个值，而是由Web服务器自己设置。 |
| Set-Cookie | 设置和页面关联的Cookie。Servlet不应使用response.setHeader("Set-Cookie", ...)，而是应使用HttpServletResponse提供的专用方法addCookie。 |
| WWW-Authenticate | 客户应该在Authorization头中提供什么类型的授权信息 |

#### HTTP状态码

| 状态码分类 | 分类说明 |
| --- | --- |
| 1** | 信息，服务器收到请求，需要请求者继续执行操作 |
| 2** | 成功，操作被成功接收并处理 |
| 3** | 重定向，需要进一步的操作以完成请求 |
| 4** | 客户端错误，请求包含语法错误或无法完成请求 |
| 5** | 服务器错误，服务器在处理请求的过程中发生了错误 |

**主要状态码说明**
| 状态码 | 英文描述 | 说明 |
| --- | --- | --- |
| 100 | Continue | 客户端应继续请求 |
| 200 | Ok | 请求成功，一般用于GET和POST请求 |
| 300 | Multiple Choices | 多种选择。请求的资源可包括多个位置，相应可返回一个资源特征与地址的列表用于用户终端（例如：浏览器）选择 |
| 301 | Moved Permanently | 永久移动。请求的资源已被永久的移动到新URI，返回信息会包括新的URI，浏览器会自动定向到新URI。今后任何新的请求都应使用新的URI代替 |
| 302 | Found	| 临时移动。与301类似。但资源只是临时被移动。客户端应继续使用原有URI |
| 304 | Not Modified	| 未修改。所请求的资源未修改，服务器返回此状态码时，不会返回任何资源。客户端通常会缓存访问过的资源，通过提供一个头信息指出客户端希望只返回在指定日期之后修改的资源 |
| 305 | Use Proxy | 使用代理。所请求的资源必须通过代理访问 |
| 400 | Bad Request | 客户端请求的语法错误，服务器无法理解 |
| 401 | Unauthorized | 请求要求用户的身份认证 |
| 403 | Forbidden | 服务器理解请求客户端的请求，但是拒绝执行此请求 |
| 404 | Not Found | 服务器无法根据客户端的请求找到资源（网页）|
| 405 | Method Not Allowed | 客户端请求中的方法被禁止 |
| 408 | Request Time-ou | 服务器等待客户端发送的请求时间过长，超时 |
| 500 | Internal Server Error | 服务器内部错误，无法完成请求 |
| | | 等等 |

#### HTTP Content-Type（内容类型）

| 值 | 介绍 |
| --- | --- |
| application/x-www-form-urlencoded | 这应该是最常见的 POST 提交数据的方式了。浏览器的原生 form 表单，如果不设置 enctype 属性，那么最终就会以 application/x-www-form-urlencoded 方式提交数据。 |
| multipart/form-data | 这又是一个常见的 POST 数据提交的方式。我们使用表单上传文件时，必须让 form 的 enctyped 等于这个值。 |
| application/json | 用来告诉服务端消息主体是序列化后的 JSON 字符串。（除了低版本 IE 之外的各大浏览器都原生支持 JSON.stringify） |
| text/xml | XML格式，它是一种使用 HTTP 作为传输协议，XML 作为编码方式的远程调用规范。 |
| text/html | HTML格式 |
| text/plain | 纯文本格式 |
| image/gif	| |
| image/jpeg | |
| image/png	| |
| application/octet-stream | 二进制流数据（如常见的文件下载） |
| application/pdf | pdf格式 |

#### 对HTTP传输进行压缩

再来说下相关的前端优化的问题。

资深的前端开发人员都知道，在web开发中，对js、css、图片、font等都要进行压缩，尽可能的减小文件的大小，减少前端下载的时间，从而提高网页响应的时间。特别是在移动端，这对用户的流量还有影响。不过本文中所提的压缩并不是指这情况，而是在js，css、图片、font等资源已经压缩了的基础上（当然，这一步非必要条件，压不压缩看你心情，资源文件的压缩跟http传输过程的压缩没关系），在http传输过程中的再次压缩。

**Accept-Encoding**

在HTTP1.1开始，Web客户端可以通过Acceppt-Encoding头来标识对压缩的支持。客户端HTTP请求头声明浏览器支持的压缩方式，服务端配置启用压缩，压缩的文件类型，压缩方式。当客户端请求到达服务器，服务器响应时对请求资源进行压缩，返回给客户端浏览器，浏览器按照相应的方式进行解析。

在HTTP请求中，accept-encoding: gzip, deflate, sdch, br是指客户端浏览器（这里是我的chrome浏览器）支持的压缩方式。在HTTP响应中，content-encoding:gzip 是指服务端使用了gzip这种压缩方式。

如何使用gzip进行压缩？

客户端不用任何配置，在服务端配置即可，服务器不同，配置方法也不尽相同。

#### 概述http的缓存控制


#### 简述三次握手四次挥手

