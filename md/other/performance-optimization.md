# 前端性能优化

判断页面性能可以从多个方向考虑，而不仅仅是load和DOMContentLoaded

* **基数度量标准** 主要是用来衡量请求数量，权重和性能的得分。适用于一些提高警报或者监控随着时间的变化而改变的项目，但是对于用户体验不是很友好。
* **里程碑度量标准** 指的是项目在加载过程中，项目在某个生命周期中一个使用状态，例如：到某个生命节点所需要的时间，或者是完成某个交互所需要的时间。虽然这个度量标准不会记录两个不同生命周期之间发生的事情，但是却适合用来监控和描述用户的体验。
* **渲染度量标准** 是用来预估页面内容呈现的速度。适合用于测量和调整渲染性能，但不适合用来测量重要内容在什么时间节点出现并可与之进行交互。
* **自定义度量标准** 是用来衡量项目的特定自定义事件。例如 Twitter 的 Time To First Tweet 和 Pinterest 的 PinnerWaitTime。适合用于精确的描述用户的体验，不太适合根据这个标准来和对手进行比较。

#### 前端优化的方法主要从多个方向下手

**合并请求**

减少浏览器对服务器发起的请求数，从而减少在发起请求过程中花费的时间。
* 合并js和css
  现在大量单页应用的js和css合并之后，会造成js文件或者css文件过大，会增加首屏渲染时间。
  过多的合并之后，后期更新之后，也会让大文件重复更新
* 使用 CSS Sprite
  将背景图片合并成一个文件，通过 background-image 和 background-position 控制显示

**避免空的src和href**
避免使用空的 src 属性确实可以缩减浏览器首屏渲染的时间，因为浏览器在渲染过程中会把 src 属性中的空内容进行加载，直至加载失败，影响 DOMContentLoaded 与 Loaded 事件之间的资源准备过程，拉长了首屏渲染所用的时间；但空的 href 属性对首屏渲染的影响比较小。

**使用CDN(内容分发网络)**
内容分发网络（CDN）是一组分散在不同地理位置的 web 服务器，用来给用户更高效地发送内容。

**内容缓存**

* 为文件头指定 Expires 或 Cache-Control ，使内容具有缓存性。
  对于静态组件：通过设置一个遥远的将来时间作为 Expires 来实现永不失效。
  多余动态组件：用合适的 Cache-Control HTTP 头来让浏览器进行条件性的请求
* 设置 ETags 来控制缓存
  ETag 是服务器和浏览器之间判断浏览器缓存中某个文件是否匹配服务器端原文件的一种机制。实体就是资源文件，如图片，脚本，样式等等。ETag 是比验证 last-modified 日期更高效的机制。

[缓存内容](../network/basics.md)这里面详细解析

**内容压缩**
使用 gzip/br 压缩内容

通过减少 HTTP 请求产生的响应包的大小，从而降低传输时间的方式来提高性能。

通过 HTTP 请求中的 Accept-Encoding 头来标识对压缩的支持。

Web 服务器通过响应中的 Content-Encoding 头来告知 Web 客户端用什么方法来响应压缩。

通常会压缩 HTML 文档，脚本和样式表，图片和pdf通常是已经压缩过的。

不过压缩通常会同时带来服务端和客户端的cpu性能消耗，要根据实际情况来进行压缩，通常需要对大于 1KB 或 2KB 的文件进行压缩。

当浏览器通过代理来发送请求时，有可能出现浏览器期望接受的压缩后内容和实际接收到的不一致的情况。解决这一问题的方法是在 Web 服务器的响应中添加 Vary 头。Web 服务器可以告诉代理根据一个或多个请求头来改变缓存的响应。

有关 Vary 的相关内容，可以移步[网络模块](../network/basics.md)去查看

**减少重构和重绘**
* 把 CSS 放到顶部，把 JS 放到底部。[浏览器渲染机制](../browser/basics.md)


**使用外链js和css**
JavaScript 和 CSS 有机会被浏览器缓存起来。对于内联的情况，由于 HTML 文档通常不会被配置为可以进行缓存的，所以每次请求 HTML 文档都要下载 JavaScript 和 CSS。所以，如果 JavaScript 和 CSS 在外部文件中，浏览器可以缓存它们，HTML 文档的大小会被减少而不必增加 HTTP 请求数量。

**减少 DNS 查询**
用户输入 URL 以后，浏览器首先要查询域名（example.com）对应服务器的 IP 地址，这个操作一般需要耗费 20-120 毫秒时间。DNS 查询完成之前，浏览器无法从服务器下载任何数据。

[DNS解析域名](../browser/basics.md)

基于性能考虑，ISP、局域网、操作系统、浏览器都会有相应的 DNS 缓存机制：IE 缓存 30 分钟，Firefox和Chrome 缓存 1 分钟。


减少DNS查找次数，最理想的方法就是将所有的内容资源都放在同一个域(Domain)下面，这样访问整个网站就只需要进行一次DNS查找，这样可以提高性能。

但理想总归是理想，上面的理想做法会带来另外一个问题，就是由于这些资源都在同一个域，而HTTP /1.1 中推荐客户端针对每个域只有一定数量的并行度（它的建议是2），那么就会出现下载资源时的排队现象，这样就会降低性能。

所以，折衷的做法是：建议在一个网站里面使用至少2个域，但不多于4个域来提供资源。我认为这条建议是很合理的，也值得我们在项目实践中去应用。(根据自己网站的实际情况来确定)

使用不同的域名可以最大化下载线程，但注意保持在 2~4 个域名内，以避免 DNS 查询损耗。

**压缩js/css**
Gulp、Webpack 等流行构建工具有UglifyJS等工具来进行压缩

**避免 301/302 重定向**
HTTP 重定向通过 301/302 状态码实现。下面是一个 301 状态码的 HTTP 头：
```javascript
 HTTP/1.1 301 Moved Permanently 
 Location: http://example.com/newuri
 Content-Type: text/html
```
客户端收到服务器的重定向响应后，会根据响应头中 Location 的地址再次发送请求。重定向会影响用户体验，尤其是多次重定向时，用户在一段时间内看不到任何内容，只看到浏览器进度条一直在刷新。

有一种常见的极其浪费资源的重定向，而且 Web 开发人员一般都意识不到这一点：URL 末尾应该添加 / 但未添加。比如，访问 `http://astrology.yahoo.com/astrology` 将被 301 重定向到 `http://astrology.yahoo.com/astrology/`（注意末尾的 /）。如果使用 Apache，可以通过 Alias 或 mod_rewrite 或 DirectorySlash 解决这个问题。

**移除重复的 JavaScript 脚本**
重复脚本会创建不必要的 HTTP 请求，执行无用的 JavaScript 代码，而影响页面性能。

**缓存Ajax请求**
Ajax在发送的数据成功后，会把请求的URL和返回的响应结果保存在缓存内，当下一次调用Ajax发送相同的请求时，它会直接从缓存中把数据取出来，这是为了提高页面的响应速度和用户体验。当前这要求两次请求URL完全相同，包括参数。这个时候，浏览器就不会与服务器交互。

ajax缓存只对GET方式的请求有效，因为浏览器认为POST请求提交的内容必定有变化，所以不走缓存。

解决ajax缓存的方法:
1. 在 Ajax 的 URL 参数后加上 “?fresh=” + Math.random(); //当然这里参数 fresh 可以任意取了
2. 在 Ajax 的 URL 参数后加上 “?timestamp=” + new Date().getTime();
3. 设置参数cache：false;
4. `<meta http-equiv="cache-control" content="no-cache" />`

**Ajax尽量使用GET方法（在可以使用get和post的情况下）**
使用 XMLHttpRequest 时，浏览器的 POST 请求是通过一个两步的过程来实现的：先发送 HTTP 头再发送数据。所以最好用 GET 请求，它只需要发送一个 TCP 报文（除非 Cookie 特别多）。

**延迟加载**
非首屏使用的数据、样式、脚本、图片等或用户交互时才会显示的内容延迟加载提高速度。等首屏加载完成或者用户操作时，再去渲染剩余的页面内容。

script的延迟加载可以通过动态创建DOM的方式。

还有就是图片的延迟加载（懒加载），具体逻辑是当用户滚动页面看到内容时，再加载所需图片。这种操作在较多图片的网页中，可以节省大量带宽，页面渲染速度也会变快。

**预加载**
预加载可能看起来和延迟加载是相反的，但它其实有不同的目标。通过预加载组件可以充分利用浏览器空闲的时间来请求将来会用到的组件（图片，样式和脚本）。用户访问下一页的时候，大部分组件都已经在缓存里了，所以在用户看来页面会加载得更快。

实际应用中有以下几种预加载的类型：

* 无条件预加载 —— 尽快开始加载，获取一些额外的组件。google.com 就是一个 sprite 图片预加载的好例子，这个 sprite 图片并不是 google.com 主页需要的，而是搜索结果页面上的内容。
* 条件性预加载 —— 根据用户操作猜测用户将要跳转到哪里并据此预加载。在 search.yahoo.com 的输入框里键入内容后，可以看到那些额外组件是怎样请求加载的。
* 提前预加载 —— 在推出新设计之前预加载。经常在重新设计之后会听到：“这个新网站不错，但比以前更慢了”，一部分原因是用户访问先前的页面都是有旧缓存的，但新的却是一种空缓存状态下的体验。可以通过在将要推出新设计之前预加载一些组件来减轻这种负面影响，老站可以利用浏览器空闲的时间来请求那些新站需要的图片和脚本。

###### HTML5中有原生的预加载属性，名为prefetch和preload

**prefetch**: prefetch 提示浏览器这个资源将来可能需要，但是把决定是否和什么时间加载这个资源的决定权交给浏览器。
**preload**: preload 是声明式的 fetch，可以强制浏览器请求资源，同时不阻塞文档 onload 事件。

**对于当前页面很有必要的资源使用 preload，对于可能在将来的页面中使用的资源使用 prefetch。**

**Chrome 有四种缓存: 硬盘缓存，内存缓存，Service Worker 缓存和 Push 缓存。preload 和 prefetch 都被存储在 HTTP 缓存中（内存缓存）中。**

用 preload 和 prefetch 情况下，如果资源不能被缓存，那么都有可能浪费一部分带宽。

如何让 preload 的 CSS 样式表立即生效？
preload 支持基于异步加载的标记，使用 <link rel=”preload”> 的样式表使用 onload 事件立即应用到文档：
```javascript
<link rel="preload" href="style.css" onload="this.rel=stylesheet">
```

在Chrome中，Prefetch的优先级为Lowest。而Preload的优先级则是根据as（值为style时优先级最高）属性值所对应的资源类型来决定，总体上，Preload的优先级比Prefetch高。不过两者都不应该延迟页面的load事件。

`<link rel="prefetch" href="(url)">`如果有很大概率会访问href指向的资源，则可以加入上面的代码，浏览器会预加载一些资源，访问就会更迅速！

[preload和prefetch详解](https://juejin.im/post/58e8acf10ce46300585a7a42)

###### dns-prefetch
当浏览器从第三方服务跨域请求资源的时候，在浏览器发起请求之前，这个第三方的跨域域名需要被解析为一个IP地址，这个过程就是DNS解析，DNS缓存可以用来减少这个过程的耗时，DNS解析可能会增加请求的延迟，对于那些需要请求许多第三方的资源的网站而言，DNS解析的耗时延迟可能会大大降低网页加载性能。

```javascript
<link rel="dns-prefetch" href="https://fonts.googleapis.com/">
```

**减少DOM元素数量**
复杂的页面不仅下载的字节更多，JavaScript DOM 操作也更慢。

```javascript
document.getElementsByTagName('*').length;  //计算出页面中有多少 DOM 元素
```

**避免使用iframe**

ifame的优点：
1. 可以用来加载速度较慢的第三方资源，如广告、徽章
2. 可用作安全沙箱
3. 可以并行下载脚本

iframe的缺点:
1. 加载代价昂贵，即使是空的页面
2. 阻塞页面 load 事件触发
3. iframe 完全加载以后，父页面才会触发 load 事件。 Safari、Chrome 中通过 JavaScript 动态设置 iframe src 可以避免这个问题。
4. 缺乏语义

**杜绝404**

HTTP 请求是昂贵的，所以发出 HTTP 请求但获得没用的响应（如 404）是完全不必要的，并且会降低用户体验。

一些网站会有特别的 404 页面提高用户体验，但这仍然会浪费服务器资源。更糟糕的是当链接指向外部 js 但却得到 404 结果。这样首先会降低（占用）并行下载数，其次浏览器可能会把 404 响应体当作 js 来解析，试图从里面找出可用的东西。

**减少Cookie**

Cookie 被用于身份认证、个性化设置等诸多用途。Cookie 通过 HTTP 头在服务器和浏览器间来回传送，减少 Cookie 大小可以降低其对响应速度的影响。

* 去除不必要的 Cookie；
* 尽量压缩 Cookie 大小；
* 注意设置 Cookie 的 domain 级别，如无必要，不要影响到 sub-domain；
* 设置合适的过期时间。

**使用不带Cookie的域名**
当浏览器请求静态图片并把 Cookie 一起发送到服务器时，Cookie 此时对服务器没什么用处。这些 Cookie 只是增加了网络流量。所以你应该保证静态文件的请求是没有 Cookie 的。可以创建一个子域名来托管所有静态组件。


**减少DOM操作**

JavaScript 操作 DOM 很慢，尤其是当 DOM 节点很多时。

使用时应该注意：
* 缓存已经访问过的元素；
* 使用 DocumentFragment 暂存 DOM，整理好以后再插入 DOM 树；
* 使用 className 来操纵元素的样式；
* 避免使用 JavaScript 修复布局。

**使用高效事件处理**

有时候感觉页面反映不够灵敏，是因为有太多频繁执行的事件处理器被添加到了 DOM 树的不同元素上，这就是推荐使用事件委托的原因

另外，不必等到 onload 事件来开始处理 DOM 树，DOMContentLoaded 更快。大多时候需要的只是想访问的元素已在 DOM 树中，所以不必等到所有图片被下载。

**优化图片**

压缩图片

**不要在 HTML 中缩放图片（主要针对pc）**

不要使用 \<img> 的 width、height 缩放图片，如果用到小图片，就使用相应大小的图片。如果需要
```javascript
\<img width="100" height="100" src="mycat.jpg" alt="My Cat" />
```

那么图片本身（mycat.jpg）应该是 100x100px 的，而不是去缩小 500x500px 的图片。

设置图片的宽和高，以免浏览器按照「猜」的宽高给图片保留的区域和实际宽高差异，产生重绘。

**使用体积小、可缓存的 favicon.ico**

Favicon.ico 一般存放在网站根目录下，无论是否在页面中设置，浏览器都会尝试请求这个文件。

所以确保这个图标：

* 存在（避免 404）；
* 尽量小，最好小于 1K；
* 设置较长的过期时间。

对于较新的浏览器，可以使用 PNG 格式的 favicon。

**其他**

* 避免使用css表达式  
  css表达式在页面呈现和调整大小时进行重新计算，而且在页面滚动时甚至在用户将鼠标移动到页面上时进行计算。

* 文件尽量不要大于 25K
  这个限制是因为 iPhone 不能缓存大于 25K 的组件，注意这里指的是未压缩的大小。这就是为什么缩减内容本身也很重要，因为单纯的 gzip 可能不够。



#### 页面加载白屏的原因有哪些，以及如何监控白屏时间，如何优化？首屏时间？
白屏时间是指浏览器从响应用户输入网址地址，到浏览器开始显示内容的时间。
首屏时间是指浏览器从响应用户输入网络地址，到首屏内容渲染完成的时间。

白屏时间 = 地址栏输入网址后回车 - 浏览器出现第一个元素
首屏时间 = 地址栏输入网址后回车 - 浏览器第一屏渲染完成

影响白屏时间的因素：网络，服务端性能，前端页面结构设计。
影响首屏时间的因素：白屏时间，资源下载执行时间。

**白屏时间监控**

通常认为浏览器开始渲染 \<body> 或者解析完 \<head> 的时间是白屏结束的时间点。

**首屏时间监控**

首屏时间为首屏结束渲染，首屏（首屏可见内容）中的图片加载完成，即是首屏完成，不在首屏中的图片可以不考虑。

监听方法:
1. 首屏模块标签标记法 
  由于浏览器解析 HTML 是按照顺序解析的，当解析到某个元素的时候，你觉得首屏完成了，就在此元素后面加入 script 计算首屏完成时间。
2. 统计首屏内加载最慢的图片/iframe
  通常首屏内容中加载最慢的就是图片或者 iframe 资源，因此可以理解为当图片或者 iframe 都加载出来了，首屏肯定已经完成了。
    ```javascript
    <!DOCTYPE html>
    <html>
    <head>
        <meta charset="utf-8">
        <title>首屏</title>
        <script>
            // 不兼容 performance.timing 的浏览器
            window.pageStartTime = Date.now()
        </script>
    </head>
    <body>
        <img src="https://lz5z.com/assets/img/google_atf.png" alt="img" onload="load()">
        <img src="https://lz5z.com/assets/img/css3_gpu_speedup.png" alt="img" onload="load()">
        <script>
            function load () {
                window.firstScreen = Date.now()
            }
            window.onload = function () {
                // 首屏时间
                console.log(window.firstScreen - performance.timing.navigationStart)
            }
        </script>
    </body>
    </html>
    ```
3. 自定义模块内容计算法
  忽略图片等资源加载情况，只考虑页面主要 DOM，只考虑首屏的主要模块，而不是严格意义首屏线以上的所有内容

**白屏时间优化**
1. DNS解析优化
    * DNS缓存优化
    * DNS预加载策略
    * 稳定可靠的DNS服务器
2. 服务端优化
3. 浏览器下载、解析、渲染页面优化
    * 尽可能的精简HTML的代码和结构,同时压缩HTML
    * 尽可能的优化CSS文件和结构
    * 一定要合理的放置JS代码，尽量不要使用内联的JS代码

**首屏时间优化**
之前说的很多前端性能优化就可以用于首屏时间优化

#### script 标签的属性有哪些

**type 与 language**
type 和 language 属性都可用来指定 \<script> 标签中的脚本的类型。

language 属性在 HTML 和 XHTML 标准中受到了非议，这两个标准提倡使用 type 属性。遗憾的是，这两个属性的值是不一样的。

如果您在使用 JavaScript，可以使用下面两种属性： `language = "JavaScript"` 或 `type = "text/javascript"`

**src 和 charset**

src 的值是包含这个 JavaScript 程序的文件的 URL。保存的文件的 MIME 类型应是 application/x-javascript，但如果文件名的后缀为 .js，也能够被正确配置了的服务器进行恰当的处理。

charset 属性与 src 属性一起使用，告诉浏览器用来编码这个 javascript 程序的字符集。它的值是任何一个 ISO 标准字符集编码的名称。

**defer 与 async**
下面有讲到

#### script 标签的 defer 和 async 标签的作用与区别
[参考浏览器渲染流程中的讲解](../browser/basics.md)

#### load 与 DOMContentLoaded

**load**

load 应该仅用于检测一个完全加载的页面 当一个资源及其依赖资源已完成加载时，将触发load事件。

意思是页面的html、css、js、图片等资源都已经加载完之后才会触发 load 事件。

**DOMContentLoaded**

当初始的 HTML 文档被完全加载和解析完成之后，DOMContentLoaded 事件被触发，而无需等待样式表、图像和子框架的完成加载。

意思是HTML下载、解析完毕之后就触发。


###### 参考
[35条前端性能优化军规](https://learnku.com/docs/f2e-performance-rules/using-gzip-compression/6374)