##  6. link 和 \<script>

##### href 和 src

* href 表示超文本引用，指向网络资源所在位置，`href` 用于在当前文档和引用资源之间确立联系。
* src 目的是要把文件下载到html页面中去，`src` 用于替换当前内容。

#### Link

##### href

此属性指定被链接资源的URL。 URL 可以是绝对的，也可以是相对的。

当浏览器遇到href会并行下载资源并且不会停止对当前文档的处理。(同时也是为什么建议使用 link 方式加载 CSS，而不是使用 @import 方式，用@import添加的样式是在页面载入之后再加载，这可能会导致页面因重新渲染而闪烁。)

主要使用场景
```html
<a href="http://www.baidu.com"></a> 
<link type="text/css" rel="stylesheet" href="common.css"> 
```
重点讲一下**link**标签的使用

##### rel

1. DNS Prefetching（DNS预解析）

    DNS 请求在带宽方面非常小，但延迟非常高，特别是在移动网络上。通过推测性地预取 DNS 结果，可以在某些时候显着降低延迟，例如当用户点击链接时。在某些情况下，延迟可能会缩短一秒。

    现在大多数新浏览器已经针对DNS解析进行了优化，典型的一次DNS解析需要耗费 20-120 毫秒，减少DNS解析时间和次数是个很好的优化方式。

    在一些浏览器中实现域名预解析与实际页面内容的获取并行（而不是与其串行）发生。通过这样做，高延迟域名解析过程在获取内容时不会造成任何延迟。

    ```html
    <link rel="dns-prefetch" href="//imgqn.koudaitong.com/" />  
    ```

    当然，chrome 会自动把当前页面的所有带href的link的dns都prefetch一遍，所以，需要手动添加link标签的场景是：你预计用户在后面的访问中需要用到当前页面的所有链接都不包含的域名。

    所以:

    * 对静态资源域名做手动dns prefetching。
    * 对js里会发起的跳转、请求做手动dns prefetching。
    * 不用对超链接做手动dns prefetching，因为chrome会自动做dns prefetching。
    * 对重定向跳转的新域名做手动dns prefetching，比如：页面上有个A域名的链接，但访问A会重定向到B域名的链接，这么在当前页对B域名做手动dns prefetching是有意义的。

    普遍来说合理的dns prefetching能对页面性能带来50ms ~ 300ms的提升(有人做了这方面的统计)

    **注意** chrome使用8个线程专门做dns prefetching，而且chrome本身不做dns记录的cache，是直接从操作系统读dns —— 也就是说，直接修改系统的dns记录或者host是可以直接影响chrome的。

    手动dns prefetching的代码实际上还是会增加html的代码量的，特别是域名多的情况下。

    所以，最优的方案应该是：通过js初始化一个iframe异步加载一个页面，而这个页面里包含本站所有的需要手动dns prefetching的域名。

    也可以通过iframe来做资源的预加载 [File PreFetching](https://tech.youzan.com/file-frefetching/)

2. Prefecth

    `<link rel="prefetch" as="script" href="example.js">`

    当确定网页在未来（下一页）一定会使用某资源时，可以通过prefetch提前请求资源并且缓存以供后续使用。但具体什么时候请求这个资源由浏览器决定。 页面跳转时prefetch发起的请求不会中断。该方法的加载优先级很低，一般用来提高下一个页面的加载速度。

3. Preload

    `<link rel="preload" as="script" href="example.js">`

    Preload是一项新的web标准，旨在提高性能和为开发人员提供更细粒度的加载控制。Preload可以让开发者自定义资源的加载逻辑，且无需忍受基于脚本的资源加载器带来的性能损失。

    preload是声明式的fetch，可以强制浏览器请求资源，将加载和执行分离开，不阻塞渲染和 document 的 onload 事件。

提前加载指定资源，不再出现依赖的 font 字体隔了一段时间才刷出。

![avator](https://mrgaogang.github.io/images/js/preload-attr.png)

对于当前页面有必要的资源使用preload，对于可能在将来的页面中使用的资源使用prefetch。 preload和prefetch请求的资源都缓存在HTTP缓存中。

Preload支持**onload**事件，可以自定义资源加载完后的回调函数。

`<link rel="proload" href="test.js" as="script" onload="console.log('finish');">`

##### Preload 和 Prefetch 总结

当资源被 preload 或者 prefetch 后，会从网络堆栈传输到 HTTP 缓存并进入渲染器的内存缓存。 如果资源可以被缓存（例如，存在有效的 cache-control 和 max-age），它将存储在 HTTP 缓存中，可用于当前和未来的会话。 如果资源不可缓存，则不会将其存储在 HTTP 缓存中。 相反，它会被缓存到内存缓存中并保持不变直到它被使用。

preload 用 “as” 或者用 “type” 属性来表示他们请求资源的优先级（比如说 preload 使用 as=”style” 属性将获得最高的优先级）。没有 “as” 属性的将被看作异步请求，“Early”意味着在所有未被预加载的图片请求之前被请求（“late”意味着之后）

脚本根据它们在文件中的位置是否异步、延迟或阻塞获得不同的优先级：

* 网络在第一个图片资源之前阻塞的脚本在网络优先级中是中级
* 网络在第一个图片资源之后阻塞的脚本在网络优先级中是低级
* 异步/延迟/插入的脚本（无论在什么位置）在网络优先级中是很低级

图像在可视窗口中比不在视口中的图像(具有更高的优先级，因此在某种程度上， Chrome 将会尽量懒加载这些不在视口中的图片。 较低优先级的图片出现在视口中时，该图片的优先级就会得到提升（但是注意已经在布局完成后的图片优先级不会在更改）。

使用“as”属性预加载的资源将具有与它们请求的资源类型相同的资源优先级。 例如，preload as =“style”将获得最高优先级，而as =“script”将获得低优先级或中优先级。 这些资源也遵循相同的CSP策略（例如脚本受 script-src 约束）。

不带 “as” 属性的 preload 的优先级将会等同于异步请求。

对于不同的资源加载优先级规则：

* html 主要资源，其优先级是最高的
* css 样式资源，其优先级也是最高的
* script 脚本资源，优先级不一
* font 字体资源，优先级不一

**注意**:

* 如果在指定要 preload 的内容（例如脚本）时未提供有效的“as”，则最终将获取两次。
* preload 字体不带 crossorigin 也将会二次获取, 确保在使用 preload 获取字体时添加crossorigin 属性，否则将二次下载。 他这个请求使用匿名的跨域模式。 即使字体与页面位于同个域 下，也建议使用。也适用于其他域名的获取(比如说默认的异步获取)。
* 不要所有的请求资源都加 preload,用 preload 来告诉浏览器一些很被需要的资源，以便让它提早获取它们。

**rel其他内容**

* apple-touch-icon-precomposed 和 apple-touch-icon

  为了苹果移动设备对移动网站（web app）添加到桌面的时候增加的图标定义属性。apple-touch-icon-precomposed属性为“设计原图图标。apple-touch-icon属性为“增加高光光亮的图标”。

##### as

preload的as属性，告诉浏览器加载的是什么资源。常用的as属性值有：`script, style, image, media, document, font`

通过设置as属性可以实现：

  * 浏览器可以设置正确的资源加载优先级
  * 浏览器可以确保请求是符合内容安全策略的
  * 浏览器根据as的值发送正确的accept头部信息
  * 浏览器根据as的值得知资源类型。因此当获取的资源相同时，浏览器能够判断前面获取的资源能否重用。

忽略as或者设置错误的值会使preload等同于XHR异步请求。但浏览器不知道加载的是什么，会赋予此类资源非常低的加载优先级。

##### type

包含该元素所指向资源的MIME类型。在浏览器进行预加载的时候，这个属性值将会非常有用——浏览器将使用type属性来判断它是否支持这一资源，如果浏览器支持这一类型资源的预加载，下载将会开始，否则便对其加以忽略。

##### crossorigin

`<audio>`、`<img>`、`<link>`、`<script>` 和 `<video>` 均有一个跨域属性 (crossorigin)，它允许你配置元素获取数据的 CORS 请求。

| 值 | 描述 |
| :---: | :---: |
| anonymous | 对此元素的 CORS 请求将不设置凭据标志。 |
| use-credentials | 对此元素的CORS请求将设置凭证标志；这意味着请求将提供凭据。 |
| "" | 设置一个空的值，如 crossorigin 或 crossorigin=""，和设置 anonymous 的效果一样。 |

##### disabled

仅对于rel="stylesheet" ，disabled 的Boolean属性指示是否应加载所描述的样式表并将其应用于文档。 如果在加载HTML时在HTML中指定了Disabled，则在页面加载期间不会加载样式表。 相反，如果禁用属性更改为false或删除时，样式表将按需加载。

##### hreflang

此属性指明了被链接资源的语言. 其意义仅供参考。

##### importance

指示资源的相对重要性。 `auto`: 表示没有偏好。 浏览器可以使用其自己的启发式方法来确定资源的优先级。 `high`: 向浏览器指示资源具有高优先级。 `low`: 向浏览器指示资源的优先级较低。

只有存在rel=“preload”或rel=“prefetch”时，importance属性才能用于`<link>`元素。

##### media

这个属性规定了外部资源适用的媒体类型。

##### referrerpolicy 

一个字符串，指示在获取资源时使用哪个引荐来源网址。

##### sizes

这个属性定义了包含相应资源的可视化媒体中的icons的大小。它只有在rel包含icon的link类型值。

#### script

##### src

当浏览器解析到src ，会暂停其他资源的下载和处理，直到将该资源加载或执行完毕。(这也是script标签为什么放在底部而不是头部的原因)

主要使用场景

```html
<img src="img/girl.jpg"></img> 
<iframe src="top.html"> 
<script src="show.js"> 
```

##### async 和 defer

浏览器遇到一个 script 标记时，DOM 构建将暂停，直至脚本完成执行，然后继续构建DOM。每次去执行JavaScript脚本都会严重地阻塞DOM树的构建，如果JavaScript脚本还操作了CSSOM，而正好这个CSSOM还没有下载和构建，浏览器甚至会延迟脚本执行和构建DOM，直至完成其CSSOM的下载和构建。

直接使用script脚本的话，html会按照顺序来加载并执行脚本，在脚本加载&执行的过程中，会阻塞后续的DOM渲染。

`<script src="script.js"></script>`

没有 defer 或 async，浏览器会立即加载并执行指定的脚本，“立即”指的是在渲染该 script 标签之下的文档元素之前，也就是说不等待后续载入的文档元素，读到就加载并执行。

`<script async src="script.js"></script>`

有 async，加载和渲染后续文档元素的过程将和 script.js 的加载与执行并行进行（异步）。

`<script defer src="myscript.js"></script>`

有 defer，加载后续文档元素的过程将和 script.js 的加载并行进行（异步），但是 script.js 的执行要在所有元素解析完成之后，DOMContentLoaded 事件触发之前完成。

总结：
* img

  图片的加载不会阻塞html的解析，但img加载后并不渲染，它需要等待Render Tree生成完后才和Render Tree一起渲染出来。未下载完的图片需等下载完后才渲染。

* css

  css加载不会阻塞Dom树的解析，css加载会阻塞DOM树的渲染（因为，如果没有CSSOM,先对DOM树进行渲染，DOM树后面可能需要重新绘制或回流，造成一些没必要的损耗）

  css加载会阻塞后面js语句的执行（如果样式下载受阻，那么也将阻塞后面js的下载和执行，因为script脚本在执行过程中可能会引用到相关样式）

  如果JavaScript脚本还操作了CSSOM，而正好这个CSSOM还没有下载和构建，浏览器甚至会延迟脚本执行和构建DOM，直至完成其CSSOM的下载和构建。

* js

  html会按照顺序来加载并执行脚本，在脚本加载&执行的过程中，会阻塞后续的DOM渲染。

  当浏览器遇到一个 script 标记时，DOM 构建将暂停，直至脚本完成执行，然后继续构建DOM。每次去执行JavaScript脚本都会严重地阻塞DOM树的构建。

  defer 和 async 在网络读取（下载）这块儿是一样的，都是异步的（相较于 HTML 解析），不会阻塞DOM树渲染，但是async是加载好后直接执行，脚本执行会阻塞DOM树渲染，正因此，async的脚本执行顺序是无序的。而defer会在所有元素解析完成之后，执行脚本，其执行时机在DOMContentLoaded 事件触发之前完成，其会按照加载顺序执行。

##### crossorigin

和\<link> 相同

##### nomodule

这个布尔属性被设置来标明这个脚本在支持 ES2015 modules 的浏览器中不执行。 — 实际上，这可用于在不支持模块化JavaScript的旧浏览器中提供回退脚本。

##### type

该属性定义script元素包含或src引用的脚本语言。属性的值为MIME类型; 支持的MIME类型包括text/javascript, text/ecmascript, application/javascript, 和application/ecmascript。如果没有定义这个属性，脚本会被视作JavaScript。

如果MIME类型不是JavaScript类型（上述支持的类型），则该元素所包含的内容会被当作数据块而不会被浏览器执行。

如果type属性为module，代码会被当作JavaScript模块

## 7. MIME

用来表示文档、文件或字节流的性质和格式

`type/subtype` MIME的组成结构非常简单；由类型与子类型两个字符串中间用'/'分隔而组成。不允许空格存在。type 表示可以被分多个子类的独立类别。subtype 表示细分后的每个类型。

MIME类型对大小写不敏感，但是传统写法都是小写。

浏览器通常使用MIME类型而不是文件扩展名来确定如何处理文档。

| 媒介类型 | 描述 |
| :---: | :---: |
| text | 表示文件是普通文本， text/plain, text/javascript(已被废弃，改用application/javascript), text/css, text/html |
| image | 图像，包括动图， image/gif， image/png, image/x-icon |
| audio | 音频文件， audio/midi, audio/ogg, audio/wav |
| video | 视频文件， video/webm, video/ogg |
| application | 表明是某种二进制文件， application/octet-stream(这是应用程序文件的默认值。意思是 未知的应用程序文件 ，浏览器一般不会自动执行或询问执行), application/xml, application/javascript |
| multipart | 复合文件类型， multipart/form-data(可用于联系 HTML Forms 和 POST 方法), multipart/byteranges |

##### image/x-icon

网站favicon类型

##### text/plain

文本文件默认值。即使它意味着未知的文本文件，但浏览器认为是可以直接展示的。

注: text/plain并不是意味着某种文本数据。如果浏览器想要一个文本文件的明确类型，浏览器并不会考虑他们是否匹配。比如说，如果通过一个表明是下载CSS文件的\<link>链接下载了一个 text/plain 文件。如果提供的信息是text/plain，浏览器并不会认出这是有效的CSS文件。CSS类型需要使用text/css

##### text/css

在网页中要被解析为CSS的任何CSS文件必须指定MIME为text/css。通常，服务器不识别以.css为后缀的文件的MIME类型，而是将其以MIME为text/plain 或 application/octet-stream 来发送给浏览器：在这种情况下，大多数浏览器不识别其为CSS文件，直接忽略掉。特别要注意为CSS文件提供正确的MIME类型。

##### text/html

所有的HTML内容都应该使用这种类型。

##### application/javascript

JS的官方MIME类型

##### multipart/form-data
常见的 POST 数据提交的方式。使用表单上传文件时，必须让 \<form> 表单的 enctype 等于 multipart/form-data。

`multipart/form-data` 可用于HTML表单从浏览器发送信息给服务器。作为多部分文档格式，它由边界线（一个由'--'开始的字符串）划分出的不同部分组成。每一部分有自己的实体，以及自己的 `HTTP` 请求头，`Content-Disposition`和 `Content-Type` 用于文件上传领域，最常用的 (`Content-Length` 因为边界线作为分隔符而被忽略）。

```html
POST / HTTP/1.1
Host: localhost:8000
User-Agent: Mozilla/5.0 (Macintosh; Intel Mac OS X 10.9; rv:50.0) Gecko/20100101 Firefox/50.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate
Connection: keep-alive
Upgrade-Insecure-Requests: 1
Content-Type: multipart/form-data; boundary=---------------------------8721656041911415653955004498
Content-Length: 465

-----------------------------8721656041911415653955004498
Content-Disposition: form-data; name="myTextField"

Test
-----------------------------8721656041911415653955004498
Content-Disposition: form-data; name="myCheckBox"

on
-----------------------------8721656041911415653955004498
Content-Disposition: form-data; name="myFile"; filename="test.txt"
Content-Type: text/plain

Simple file.
-----------------------------8721656041911415653955004498--
```

##### application/x-www-form-urlencoded
最常见的POST提交数据的方式，浏览器原生\<form>表单，如果不设置enctype属性，那么最终会以application/x-wwww-form-urlencoded方式提交数据

```json
POST http://www.example.com HTTP/1.1
Content-Type: application/x-www-form-urlencoded;charset=utf-8

title=test&sub%5B%5D=1&sub%5B%5D=2&sub%5B%5D=3
```
首先，Content-Type 被指定为 application/x-www-form-urlencoded；其次，提交的数据按照 key1=val1&key2=val2 的方式进行编码，key 和 val 都进行了 URL 转码。大部分服务端语言都对这种方式有很好的支持。

很多时候，我们用 Ajax 提交数据时，也是使用这种方式。

##### application/json
application/json 这个 Content-Type 作为响应头，用来告诉服务端消息主体是序列化后的 JSON 字符串。

这种方案，可以方便的提交复杂的结构化数据，特别适合 RESTful 的接口。

##### text/xml

它是一种使用 HTTP 作为传输协议，XML 作为编码方式的远程调用规范,它的使用也很广泛，能很好的支持已有的 XML-RPC 服务。不过，XML 结构还是过于臃肿，一般场景用 JSON 会更灵活方便。

#### 补充:
**AJAX跨域POST发送json时，会先发送一个 OPTIONS 预请求**

在跨域请求中，分为简单请求（get和部分post，post时content-type属于application/x-www-form-urlencoded，multipart/form-data，text/plain中的一种）和复杂请求。而复杂请求发出之前，就会出现一次options请求。

在ajax中出现options请求，也是一种提前探测的情况，ajax跨域请求时，如果请求的是json，就属于复杂请求，因此需要提前发出一次options请求，用以检查请求是否是可靠安全的，如果options获得的回应是拒绝性质的，比如404\403\500等http状态，就会停止post、put等请求的发出。

靠javascript客户端取消options请求是不可能的，只能通过服务端对options请求做出正确的回应，这样才能保证options请求之后，post、put等请求可以被发出。但是，我们不能允许所有的options请求，而应该是有条件的，所以最好是通过一个特殊的机制，去验证客户端发出的options请求数据是否是符合服务端的条件的，如果不满足，返回403，则客户端会取消原有的post计划。

前台跨域post请求，由于CORS（cross origin resource share）规范的存在，浏览器会首先发送一次options嗅探，同时header带上origin，判断是否有跨域请求权限，服务器响应access-control-allow-origin的值，供浏览器与origin匹配，如果匹配则正式发送post请求。

如果有服务器程序权限，设置，比如jsp中，设置header access-control-allow-origin等于*，就可以得到跨域访问的目的。

**注意**
设置正确的MIME类型非常重要，其中只有正确设置了MIME类型的文件才能被 \<video> 或\<audio> 识别和播放

## 8. 语义化

语义化是指根据内容的结构化（内容语义化），选择合适的标签（代码语义化），便于开发者阅读和写出更优雅的代码的同时，让浏览器的爬虫和机器很好的解析。

语义化的好处

* 用正确的标签做正确的事情；
* 去掉或者丢失样式的时候能够让页面呈现出清晰的结构；
* 方便其他设备解析（如屏幕阅读器、盲人阅读器、移动设备）以意义的方式来渲染网页；
* 有利于SEO：和搜索引擎建立良好沟通，有助于爬虫抓取更多的有效信息：爬虫依赖于标签来确定上下文和各个关键字的权重；
* 便于团队开发和维护，语义化更具可读性，遵循W3C标准的团队都遵循这个标准，可以减少差异化。

## 9. iframe的优缺点

优点
* iframe可以实现无刷新文件上传；
* iframe可以跨域通信；
* 解决了加载缓慢的第三方内容如图标和广告等的加载问题。

缺点
* iframe会阻塞主页面的onload事件（不过iframe可以和主页面并行加载，不会阻塞主页面加载）;
* 无法被一些搜索引擎索引到;
* 页面会增加服务器的http请求;
* 会产生很多页面，不容易管理。

### iframe的四种使用方法

##### 1> 普通方法加载iframe

这是一种人尽皆知的普通加载方法，它没有浏览器的兼容性问题。

`<iframe src="/path/to/file" frameborder="0" width="728" height="90" scrolling="auto"> </iframe>`

使用这种加载方法会在各浏览器中有如下表现：

1. iframe会在主页面的onload之前加载
2. iframe会在所有iframe的内容都加载完毕之后触发iframe的onload
3. 主页面的onload会在iframes的onload触发之后触发，所以iframe会阻塞主页面的加载
4. 当iframe在加载的过程中，浏览器的会标识正在加载东西，处于忙碌状态。

**注意onload阻塞**。如果iframe的内容只需要很短的时间来加载和执行，那么也不是个大问题，而且使用这种方法还有个好处是可以和主页面并行加载。但是如果加载这个iframe需要很长时间，用户体验就很差了。

##### 2> 在onload之后加载iframe

如果你想在iframe中加载一些内容，但是这些内容对于页面来说不是那么的重要。或者这些内容不需要马上展现给用户的，需要点击触发之类的。那么可以考虑在主页面载入之后加载iframe。

这种加载方法也是没有浏览器的兼容性问题的：

1. iframe会在主页面onload之后开始加载
2. 主页面的onload事件触发与iframe无关，所以iframe不会阻塞加载
3. 当iframe加载的时候，浏览器会标识正在加载

**但是**当iframe加载的时候，还是会看到浏览器的忙碌状态，相对于普通加载方法，用户看到忙碌状态的时间更长。还有就是用户还没等到页面完全加载完的时候就已经离开了。有些情况下这是个问题，比如广告。

##### 3> setTimeout()来加载iframe

这种方法的目的是不阻塞onload事件。

在除了IE8以外的所有浏览器中会有如下表现：

1. iframe会在主页面onload之前开始加载
2. iframe的onload事件会在iframe的内容都加载完毕之后触发
3. iframe不会阻塞主页面的onload事件(IE8除外)
4. 当iframe加载的时候，浏览器会显示忙碌状态

使用iframe来加载一些插件，并且真正做到了无阻塞加载。

```javascript
<script>

(function(d) {
    var iframe = d.body.appendChild(d.createElement('iframe')),
    doc = iframe.contentWindow.document;
 
    // style the iframe with some CSS
    iframe.style.cssText = "position:absolute;width:200px;height:100px;left:0px;";
 
    doc.open().write('<body onload="' + 'var d = document;d.getElementsByTagName(\'head\')[0].' + 'appendChild(d.createElement(\'script\')).src' + '=\'\/path\/to\/file\'">');
 
    doc.close(); //iframe onload event happens
})(document);
</script>
```

神奇的地方就在<body onload=”">:这个iframe一开始没有内容，所以onload会立即触发。然后你创建一个script元素，用他来加载内容、广告、插件什么的，然后再把这个script添加到HEAD中去，这样iframe内容的加载就不会阻塞主页面的onload！

1. iframe会在主页面onload之前开始加载
2. iframe的onload会立即触发，因为iframe的内容一开始为空
3. 主页面的onload不会被阻塞
4. 如果你不在iframe使用onload监听，那么iframe的加载就会阻塞主页面的onload
5. 当iframe加载的时候，浏览器终于不显示忙碌状态了（非常好）

##### 4> 友好型iframe加载

这是用来加载广告的。虽然这不是一种iframe的加载技术，但是是用iframe来盛放广告的。他的亮点不在于iframe如何加载，而是主页面、iframe、广告如何协同工作的。如下：

1. 先创建一个iframe。设置他的src为一个相同域名下的静态html文件
2. 在这个iframe里面，设置js变量inDapIF=true来告诉广告它已经加载在这个iframe里面了
3. 在这个iframe里面，创建一个script元素加上广告的url作为src，然后像普通广告代码一样加载
4. 当广告加载完成，重置iframe大小来适应广告
5. 这种方法也没有浏览器的兼容性问题。

## 10. table的优缺点

优点：在某些场合，使用Table是100%的适合、恰当和正确。比如，用table做表格是完全正确的

缺点：
* Table要比其它html标记占更多的字节，会导致延迟下载时间，占用服务器更多的流量资源；
* Table会阻挡浏览器渲染引擎的渲染顺序，这会导致延迟页面的生成速度，让用户等待更久的时间；(**由于浏览器的流布局，对渲染树的计算通常只需要遍历一次就可以完成。但 table及其内部元素除外，它可能需要多次计算才能确定好其在渲染树中节点的属性，通常要花3倍于同等元素的时间。这也是为什么我们要避免使用 table做布局的一个原因。**)
* 灵活性差，比如要通多td才能设置tr的border属性；
* 代码臃肿，当在table中套用table的时候，阅读代码会显得异常混乱；
* 混乱的colspan与rowspan，用来布局时，频繁使用他们会造成整个文档顺序混乱；
* 深层的嵌套，导致搜索引擎读取困难，同时还很大程度上增加了代码冗余；
不够语义。

注意: table布局的一个小改动会造成整个table重新布局

## 11. HTML5有哪些新特性

HTML5主要是针对图像、存储、多任务、语义化等功能的增加

1. 图像: 音频(audio)、视频(video)、画布(canvas)
2. 存储: localStorage、sessionStorage、indexDB
3. 语义化: header footer nav aside article section
4. 多任务: webworker websocket Geolocation(定位)

还有一些表单控件

此外还移除了一些标签（自带样式）basefont、font、center、u、big、strike、tt以及可用性产生负面影响的元素frameset、noframes和frame

## 12. 浏览器的怪异模式(Quirks)和严格模式(Standards)

在W3C标准出台之前，不同的浏览器在页面的渲染上没有同一的规范，产生了差异，即Quirks mode(怪异模式或兼容模式)。

当W3C标准出台之后，不同浏览器对页面的渲染有了统一的标准,即Strict mode（标准模式或严格模式）。

1. DTD的作用

  声明位于文档 的最前面，告知浏览器的解析器，用什么文档类型、规范来解析这个文档。

  三种DTD类型：严格、过渡和基于框架的HTML版本。

2. 严格模式、怪异模式、混杂模式

  * 严格模式：严格模式的排版和JS运作模式是以该浏览器支持的最高标准运行。

  * 混杂模式：混杂模式的页面以宽松的向后兼容的方式显示；模拟老的浏览器的行为以防止站点无法工作。

  * 怪异模式：怪异模式则是使用浏览器自己的方式来解析执行代码。

  ```html
  <!DOCTYPE html>  //HTML5文档声明
  <!DOCTYPE html PUBLIC "-//W3C//DTD *XHTML 1.0* Strict//EN" "http://www.w3.org/TR/xhtml1/DTD/xhtml1-strict.dtd"> //XHTML 1.0严格模式的文档类型声明
  ```

  保留文档类型声明主要是历史原因，没有文档声明的话大多数浏览器都将会转换到为怪异模式(quirk mode)，有些地方会称为混杂模式，这种模式下浏览器会以老版本的浏览器使用的规则来渲染页面，并且不同浏览器的混杂模式还是不一样的，我们在平时码代码时应该尽量回避这种错误。

  在添加了文档类型声明之后，浏览器使用的就是严格模式(standard mode)，也称标准模式，这种情况下浏览器会用W3C的标准来渲染网页。

3. 严格模式和怪异模式的区别

  * 盒模型
    在W3C标准中，如果设置一个元素的宽度和高度，指的是元素内容的宽度和高度，而在Quirks 模式下，IE的宽度和高度还包含了padding和border；
  * 设置行内元素的高宽
    在Standards模式下，给等行内元素设置wdith和height都不会生效，而在quirks模式下，则会生效；
  * 行内元素的垂直对齐
    在Standards模式下，其是基线对其，而在quirks模式下，则是底线对齐
  * 设置百分比的高度
    在standards模式下，一个元素的高度是由其包含的内容来决定的，如果父元素没有设置百分比的高度，子元素设置一个百分比的高度是无效的用；
  * 设置水平居中
    使用margin:0 auto在standards模式下可以使元素水平居中，但在quirks模式下却会失效。

## 13. HTML5中的datalist

\<datalist>标签，用来定义选项列表，与input元素配合使用，来定义input可能的值。

datalist及其选项不会被显示出来，他仅仅是合法的输入列表值。

```html
<label for="ice-cream-choice">Choose a flavor:</label>
<input list="ice-cream-flavors" id="ice-cream-choice" name="ice-cream-choice" />

<datalist id="ice-cream-flavors">
    <option value="Chocolate">
    <option value="Coconut">
    <option value="Mint">
    <option value="Strawberry">
    <option value="Vanilla">
</datalist>
```

## 14. 渐进增强和优雅降级之间的区别

* 渐进增强（progressive enhancement）：主要是针对低版本的浏览器进行页面重构，保证基本的功能情况下，再针对高级浏览器进行效果，交互等方面的改进和追加功能，以达到更好的用户体验。
* 优雅降级 graceful degradation： 一开始就构建完整的功能，然后再针对低版本的浏览器进行兼容。

## 15. 为什么利用多个域名来存储网站资源会更有效

* CDN缓存更加方便；
* 突破浏览器并发限制；
* 节约cookie宽带；
* 节约主域名的连接数，优化页面下响应速度；
* 防止不必要的安全问题；

## 16. 自身对网页标准和标准制定机构重要性有何理解
网页标准和标准制定机构是让web更加规范，更加标准，健康的发展所必不可少的东西。

* 于开发者而言： 开发者可以遵循统一的开发标准，大大降低了开发难度，开发成本，从而也提高了代码的可阅读性以及易于后期维护；
* 于SEO而言： 对SEO更加友好，提升了搜索效率。

## 17. 浏览器如何对HTML5的离线储存资源进行管理和加载

1. 有线情况下
    * ~~浏览器发现html头部有manifest属性，它会请求manifest文件，如果是第一次访问app，那么浏览器就会根据 manifest文件的内容下载相应的资源并且进行离线存储。~~
    * Service Worker
    * 如果已经访问过app并且资源已经离线存储了，那么浏览器就会使用离线的资源加载页面，然后 浏览器会对比新的manifest文件与旧的manifest文件，如果文件没有发生改变，就不做任何操作，如果文件改变了，那么就会重新下载文件中的资源并进行离线存储。
2. 在离线情况下
  浏览器直接使用离线缓存的资源；

## 18. 页面可见性（Page Visibility）API 可以有哪些用途

**document.hidden**

它显示页面是否为用户当前观看的页面，值为ture或false

**document.visibilityState**

visibilityState的值要么是visible (表明页面为浏览器当前激活tab，而且窗口不是最小化状态)，要么是hidden (页面不是当前激活tab页面，或者窗口最小化了。)，或者prerender (页面在重新生成，对用户不可见。).

```javascript
// 各种浏览器兼容 var hidden, state, visibilityChange; 
if (typeof document.hidden !== "undefined") { 
    hidden = "hidden";
    visibilityChange = "visibilitychange"; 
    state = "visibilityState"; 
} else if (typeof document.mozHidden !== "undefined") { 
    hidden = "mozHidden"; 
    visibilityChange = "mozvisibilitychange";
    state = "mozVisibilityState"; 
} else if (typeof document.msHidden !== "undefined") { 
    hidden = "msHidden"; 
    visibilityChange = "msvisibilitychange";
    state = "msVisibilityState"; 
} else if (typeof document.webkitHidden !== "undefined") { 
    hidden = "webkitHidden"; 
    visibilityChange = "webkitvisibilitychange";
    state = "webkitVisibilityState";
 } 
// 添加监听器，在title里显示状态变化
document.addEventListener(visibilityChange, function() { 
  document.title = document[state]; 
}, false);
// 初始化 
document.title = document[state];


document.addEventListener("visibilitychange", function() {
  console.log( document.visibilityState );
});
 
document.addEventListener("msvisibilitychange", function() {
  console.log( document.msVisibilityState);
});
 
document.addEventListener("mozvisibilitychange", function() {
  console.log( document.mozVisibilityState);
});
 
document.addEventListener("webkitvisibilitychange", function() {
  console.log( document.webkitVisibilityState);
});
```

## 19. HTML5的form如何关闭自动完成功能

给不想要提示的input是设置autocomplete=off即可


## 补充
[html基础篇](https://juejin.im/post/6844904180943945742)