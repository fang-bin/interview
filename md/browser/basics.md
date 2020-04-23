### 跨域问题


### 浏览器渲染机制

#### 常见浏览器内核

| 浏览器 | IE | FireFox | Chrome Opera | Safari |
| --- | --- | --- | --- | --- |
| 内核 |  Trident | Gecko | Blink | Webkit |

浏览器内核分为两个部分: 渲染引擎和JS引擎

### 浏览器缓存策略

##### 浏览器缓存位置和优先级

1. **Service Worker**
    Service Worker 是浏览器在后台独立于网页运行的脚本，其无法直接访问Dom，可以通过 postMessage 接口发送消息来与其控制的页面通信，可以在这个线程中缓存文件（可以通过编辑拦截请求和返回，控制缓存的规则），并且该缓存是持续性的

2. **Memory Cache(内存缓存)**
    内存缓存不是持续性的，缓存会随着进程释放而释放

3. **Disk Cache(硬盘缓存)**
    持续性缓存，并且容量更优，它会根据HTTP header的字段判断哪些资源需要缓存

4. **Push Cache(推送缓存)**
    推送缓存是HTTP/2的内容，目前应用较少

##### 强缓存（不需要向服务器询问的缓存）

###### Expires
过期时间，会在设置的时间之后失效

###### Cache-Control
HTTP/1.1新增字段，`Cache-Control`可以通过`max-age`设置过期时间，如果设置`Cache-Control: no-cache，no-store, must-revalidate`；浏览器则不会做任何缓存。`Cache-Control`会覆盖`Expires`的设置(它会覆盖其他的一切缓存规则)

##### 协商缓存（需要向服务器询问缓存是否已经过期）
###### Last-Modified
最后修改的时间，浏览器第一次请求资源时，服务器会在响应头上加 `Last-Modified`，当浏览器再次加载该资源时，浏览器会在请求头中带上`If-Modified-Since`字段，值就是`Last-Modified`的值，服务器会比对两个时间，若相同则返回304，否则返回新资源，并更新`Last-Modified`。

###### ETag
HTTP/1.1新增字段，表示资源唯一表示，只要资源改动，就会重新计算`ETag`，缓存流程和`Last-Modified`一样，只是它是想服务器发送请求头中带`If-none-Match`字段，如果`ETag`字段不匹配，说明资源已经改变，返回新资源并更新`ETag`，若匹配则返回304。

**ETag 比 Last-Modified 更准确**:打开资源但是并未更改，Last-Modified也会更改，并且Last-Modified的单位时间为秒，如果更改文件的一秒内请求资源，还会命中缓存。
**如果什么缓存策略都没有设置，那么浏览器会取响应头中的Date减去Last-Modified值的10%做为缓存时间**

### 同源策略
浏览器的同源策略限制一个origin的文档或者它加载的脚本如何能与另一个源的资源进行交互。

它为了保证用户信息的安全，防止恶意的网站窃取数据，阻隔恶意文档，减少可能被攻击的媒介。

其中同源指的是:**协议相同、域名相同、端口相同**

限制范围包括:
* Cookie（针对cookie的同源策略只关注域名，忽略协议和端口）、LocalStorage 和 IndexDB 无法读取
* DOM 无法获得
* AJAX 请求不能发送

##### 如何绕过同源策略，共享cookie（强调!同源策略针对Cookie，只关注域名，忽略协议和端口）

Cookie 是服务器写入浏览器的一小段信息，只有同源的网页才能共享。但是，两个网页一级域名相同，只是二级域名不同，浏览器允许通过设置`document.domain`共享 Cookie。

另外，服务器也可以在设置Cookie的时候，指定Cookie的所属域名为一级域名，比如.example.com。 

```html
Set-Cookie: key=value; domain=.example.com; path=/
```

这样的话，二级域名和三级域名不用做任何设置，都可以读取这个Cookie。

##### 如何绕过同源策略，操控iframe
如果两个窗口一级域名相同，只是二级域名不同，那么设置上一节介绍的document.domain属性，就可以规避同源政策，操控iframe，拿到DOM。

但是对于完全不同源的网站，可以通过以下方法实现跨域窗口的通信

* 片段识别符
* window.name
* 跨文档通信API（Cross-document messaging）

##### 片段识别符实现跨域窗口通信
片段标识符（fragment identifier）指的是，URL的#号后面的部分，比如http://example.com/x.html#fragment的#fragment。如果只是改变片段标识符，页面不会重新刷新。

父窗口可以把信息，写入子窗口的片段标识符。

```javascript
var src = originURL + '#' + data;
document.getElementById('myIFrame').src = src;
//子窗口通过监听hashchange事件得到通知。
window.onhashchange = checkMessage;
function checkMessage() {
  var message = window.location.hash;
  // ...
}
```

同样的，子窗口也可以改变父窗口的片段标识符。

```javascript
parent.location.href= target + "#" + hash;
```

##### window.name实现跨域窗口通信
浏览器窗口有window.name属性。这个属性的最大特点是，无论是否同源，只要在同一个窗口里，前一个网页设置了这个属性，后一个网页可以读取它。

这种方法的优点是，window.name容量很大，可以放置非常长的字符串；缺点是必须监听子窗口window.name属性的变化，影响网页性能。

##### window.postMessage
HTML5引入了window.postMessage，允许跨窗口通信，不论这两个窗口是否同源。

父窗口http://aaa.com向子窗口http://bbb.com发消息，调用postMessage方法

```javascript
var popup = window.open('http://bbb.com', 'title');
popup.postMessage('Hello World!', 'http://bbb.com');
```

postMessage方法的第一个参数是具体的信息内容，第二个参数是接收消息的窗口的源（origin），即"协议 + 域名 + 端口"。也可以设为*，表示不限制域名，向所有窗口发送。

子窗口向父窗口发送消息的写法类似

```javascript
window.opener.postMessage('Nice to see you', 'http://aaa.com');
```

父窗口和子窗口都可以通过message事件，监听对方的消息。

```javascript
window.addEventListener('message', function(e) {
  console.log(e.data);
},false);
```

message事件的事件对象event，提供以下三个属性。

* event.source：发送消息的窗口
* event.origin: 消息发向的网址
* event.data: 消息内容

#### 如何绕过同源策略，和其他页面共享LocalStorage

通过window.postMessage，读写其他窗口的 LocalStorage 也成为了可能。


#### 如何绕过同源策略，向跨源服务器发送AJAX
除了架设服务器代理（浏览器请求同源服务器，再由后者请求外部服务），有三种方法规避这个限制:

* JSONP
* WebSocket
* CROS

###### JSONP
script标签jsonp跨域，原因script标签，img标签等不受同源策略限制

手写JSONP

```javascript
(function (window,document){
  'use strict'
  const jsonp = function(url,data,callback){
    let dataString = url.indexOf('?') ? '&' : '?';
    for (let key in data) {
      dataString += `${key}=${data[key]}&`;
    }
    const jsonp_cb = Math.random.toString.replace('.', '');
    dataString += `callback=${jsonp_cb}`;
    const scriptDom = document.createElement('script');
    scriptDom.src = `${url + dataString}`;
    window[jsonp_cb] = function (data){
      callback(data);
      document.body.removeChild(scriptDom);
    }
    document.body.appendChild(scriptDom);
  }
  window.$jsonp = jsonp;
})(window, document);
```

###### WebSocket
WebSocket是一种通信协议，使用ws://（非加密）和wss://（加密）作为协议前缀。该协议不实行同源政策，只要服务器支持，就可以通过它进行跨源通信。

###### CORS
CORS是跨源资源分享（Cross-Origin Resource Sharing）的缩写。

CORS是跨源AJAX请求的根本解决方法。相比JSONP只能发GET请求，CORS允许任何类型的请求。

## Cookie LocalStorage SessionStorage Session
#### cookie localStorage sessionStorage三者区别

* **容量方面**: localStorage 和 sessionStorage均为5M左右，cookie在不同的浏览器中每个域的数量和大小均不同，不过硬尽量保证数量小于20(ie7+, chrome, firefox最小50个，safari不限制)，大小应尽量小于4k
* **时效性（声明周期）** localStroage和sessionStorage都以文件的形式存储在本地（硬盘），localStorage存储的数据是永久性的，除非用户人为删除否则一直存在，而sessionStorage则是其标签关闭数据也会被清除。cookie一般由服务器生成，可设置失效时间，如果在浏览器端生成cookie，默认关闭浏览器后失效。
* **作用域** localStorage同一浏览器中，同源文档可以共享localStorage数据，而sessionStorage则是只有同一浏览器，同一窗口的同源文档才能共享数据（统一标签窗口不同的iframe也可以共享数据）。

#### 三者相同
* 都受浏览器同源策略限制（不过针对cookie的同源策略，只关注域名，忽略协议和端口）

#### Session
Session是无状态的HTTP协议下，服务端记录用户状态时用于标识具体用户的机制。它是在服务端保存的用来跟踪用户的状态的数据结构，可以保存在文件、数据库或者集群中。

在浏览器关闭后这次的Session就消失了，下次打开就不再拥有这个Session。其实并不是Session消失了，而是Session ID变了，服务器端可能还是存着你上次的Session ID及其Session 信息，只是他们是无主状态，也许一段时间后会被删除。

大多数的应用都是用Cookie来实现Session跟踪的，第一次创建Session的时候，服务端会在HTTP协议中告诉客户端，需要在Cookie里面记录一个SessionID，以后每次请求把这个会话ID发送到服务器。

#### Session 和 Cookie 的关系与区别

* Session是在服务端保存的一个数据结构，用来跟踪用户的状态，这个数据可以保存在集群、数据库、文件中，Cookie是客户端保存用户信息的一种机制，用来记录用户的一些信息，也是实现Session的一种方式。
* Cookie的安全性一般，他人可通过分析存放在本地的Cookie并进行Cookie欺骗。在安全性第一的前提下，选择Session更优。重要交互信息比如权限等就要放在Session中，一般的信息记录放Cookie就好了。
* 单个Cookie保存的数据不能超过4K，很多浏览器都限制一个站点最多保存20个Cookie
* 当访问增多时，Session会较大地占用服务器的性能。考虑到减轻服务器性能方面，应当适时使用Cookie。
* Session的运行依赖Session ID，而Session ID是存在 Cookie 中的。也就是说，如果浏览器禁用了Cookie,Session也会失效（但是可以通过其它方式实现，比如在url中传递Session ID,即sid=xxxx）。

#### 浏览器全面禁用第三方Cookie

**第三方Cookie**
第一方Cookie是由当前网站域所设置的Cookie,第三方Cookie来源于当前网页上嵌入的广告或图片等其他域来源。第一方Cookie和第三方Cookie都是网站在客户端上存放的一小块数据，它们都由某个域存放，只被这个域访问，它们的区别并不是技术上的区别，而是使用方式的区别。

例如你再访问天猫的情况下，会发现它的Cookie不都是.tmall.com这个域下面的，还有很多其他域下的cookie，这些cookie就是第三方cookie

**第三方cookie的用途**
* 前端日志打点
* 用户行为追踪（获取用户信息）

**浏览器禁用第三方Cookie之后**
* 使用第一方Cookie替代第三方Cookie，将设置的Cookie通过参数发给第三方
* 浏览器指纹（是一种通过浏览器对网站可见的配置和设置信息来跟踪Web浏览器的方法）


## 浏览器从发送请求到展示页面的过程

1. 解析域名

    浏览器先查询hosts文件是否有与这个域名对应的ip地址，如果有则直接向这个ip地址发起http请求。如果没有，浏览器向本地DNS服务器发出解析域名的DNS解析报文，本地DNS服务器收到请求后，先查询缓存，判断是否有对应的记录，如果有就返回这条记录，如果没有，本地DNS服务器于是就向DNS根服务器发起查询请求。

    现在的互联网是采用了树型的分层的分布式域名解析服务器集群来完成的。如果这一级的DNS服务器上找不到，则会进一步向上一级的DNS发去查询请求。直到根域名服务器为止。如果中间找到了对应的公网Ip，则再一级一级返回，如果找不到则返回错误信息。域名解析失败。

2. 通过ip查找服务器，找到服务器，建立TCP连接

    分为三步，也叫TCP三次握手：

    * TCP连接请求方，也就是客户端会发送一个请求建立连接的syn包（syn=x）给服务器；
    * 服务器接受到建立连接的请求syn包后，会回复一个确认包。其中有参数ACK=x+1。同时它自己还会再发一个SYN包，这里的SYN=y，并为这次连接分配资源；
    * 客户端接收到服务器端发送来的确认建立连接请求的包后，再次回复一个确认包。这里的ACK=y+1，并分配资源。到了这里，客户端和服务端之间的链接就已经建立起来了。
    在此之后，浏览器开始向服务器发送http请求，请求数据包。请求信息包含一个头部和一个请求体。

    服务器做出响应，返回浏览器需要的相应格式的数据。（即html）

    在响应结果中都有一个HTTP状态码：
    | 类别 | 原因 |
    | --- | --- |
    | 1**	| 信息性状态码 |
    | 2** |	成功状态码 |
    | 3**	| 重定向状态码 |
    | 4** | 客户端错误状态码 |
    | 5**	| 服务器错误状态码 |

3. 关闭TCP连接
为了避免服务器与客户端双方的资源占用和损耗，当双方没有请求或响应传递时，任意一方都可以发起关闭请求。与创建TCP连接的3次握手类似，关闭TCP连接，需要4次握手。

4. 浏览器渲染
    准确地说，浏览器需要加载解析的不仅仅是HTML，还包括CSS、JS。以及还要加载图片、视频等其他媒体资源。

    其中，img的加载不会阻塞html的解析，但img加载后并不渲染，它需要等待Render Tree生成完后才和Render Tree一起渲染出来。未下载完的图片需等下载完后才渲染。

    (1) 浏览器会将HTML解析成一个DOM树，DOM 树的构建过程是一个深度遍历过程：当前节点的所有子节点都构建好后才会去构建当前节点的下一个兄弟节点。 构建DOM树

    (2) 将CSS解析成 CSS Rule Tree。 构建CSS规则树

    (3) 根据DOM树和CSSOM来构造 Rendering Tree。注意：Rendering Tree 渲染树并不等同于 DOM 树，因为一些像Header或display:none的东西就没必要放在渲染树中了。 构建渲染树

    (4) 有了Render Tree，浏览器已经能知道网页中有哪些节点、各个节点的CSS定义以及他们的从属关系。下一步操作称之为layout，顾名思义就是计算出每个节点在屏幕中的位置。 渲染树计算

    (5) 再下一步就是绘制，即遍历render树，并使用UI后端层绘制每个节点。 渲染树绘制

    注意：上述这个过程是逐步完成的，为了更好的用户体验，渲染引擎将会尽可能早的将内容呈现到屏幕上，并不会等到所有的html都解析完成之后再去构建和布局render树。它是解析完一部分内容就显示一部分内容，同时，可能还在通过网络下载其余内容。
    ![渲染树](https://github.com/fang-bin/interview/blob/master/image/render.jpeg)
    **简单来说**：浏览器通过解析HTML，生成DOM树，解析CSS，生成CSS规则树，然后通过DOM树和CSS规则树生成渲染树。渲染树与DOM树不同，渲染树中并没有head、display为none等不必显示的节点。根据渲染树计算每个节点在屏幕中的位置。最后进行绘制

    要注意的是，浏览器的解析过程并非是串连进行的，比如在解析CSS的同时，可以继续加载解析HTML，但在解析执行JS脚本时，会停止解析后续HTML，这就会出现阻塞问题

    **渲染阻塞**

    当浏览器遇到一个 script 标记时，DOM 构建将暂停，直至脚本完成执行，然后继续构建DOM。每次去执行JavaScript脚本都会严重地阻塞DOM树的构建，如果JavaScript脚本还操作了CSSOM，而正好这个CSSOM还没有下载和构建，浏览器甚至会延迟脚本执行和构建DOM，直至完成其CSSOM的下载和构建。

### script和css阻塞和解决方案

#### script加载解析
直接使用script脚本的话，html会按照顺序来加载并执行脚本，在脚本加载&执行的过程中，会阻塞后续的DOM渲染。defer 和 async 都只适用于外部脚本文件，对与内联的 script 标签是不起作用。

* `<script src="script.js"></script>`
  没有 defer 或 async，浏览器会立即加载并执行指定的脚本，“立即”指的是在渲染该 script 标签之下的文档元素之前，也就是说不等待后续载入的文档元素，读到就加载并执行。

* `<script async src="script.js"></script>`
  有 async，加载和渲染后续文档元素的过程将和 script.js 的加载与执行并行进行（异步）。

* `<script defer src="script.js"></script>`
  有 defer，加载后续文档元素的过程将和 script.js 的加载并行进行（异步），但是 script.js 的执行要在所有元素解析完成之后，DOMContentLoaded 事件触发之前完成。

* `<script type="module" src="script.js"></script>`
  type 为 module 的 script 标签将被当作一个 JavaScript 模块对待，被称为 module script，且不受 charset 和 defer 属性影响。

  支持 module script 的浏览器，不会执行拥有 nomodule 属性的 script。

  不支持 module script 的浏览器，会忽略未知的 type="module" 的 script，同时也会忽略传统 script 中不认识的 nomodule 属性，进而执行传统的 bundle.js 代码。

  module script 以及其依赖所有文件（源文件中通过 import 声明导入的文件）都会被下载，一旦整个依赖的模块树都被导入，页面文档也完成解析，app.js 将会被执行。

  但是如果 module script 里有 async 属性，比如 <script type="module" src="util.js" async></script> ，module script 及其所有依赖都会异步下载，待整个依赖的模块树都被导入时会立即执行，而此时页面有可能还没有完成解析渲染。

![script加载](https://github.com/fang-bin/interview/blob/master/image/script-load.jpeg)

蓝色线代表网络读取，红色线代表执行时间，这俩都是针对脚本的；绿色线代表 HTML 解析。

此图告诉我们以下几个要点：

* defer 和 async 在网络读取（下载）这块儿是一样的，都是异步的（相较于 HTML 解析）
* 它俩的差别在于脚本下载完之后何时执行，显然 defer 是最接近我们对于应用脚本加载和执行的要求的
* 关于 defer，此图未尽之处在于它是按照加载顺序执行脚本的，这一点要善加利用
* async 则是一个乱序执行的主，反正对它来说脚本的加载和执行是紧紧挨着的，所以不管你声明的顺序如何，只要它加载完了就会立刻执行
* 仔细想想，async 对于应用脚本的用处不大，因为它完全不考虑依赖（哪怕是最低级的顺序执行），不过它对于那些可以不依赖任何脚本或不被任何脚本依赖的脚本来说却是非常合适的，最典型的例子：Google Analytics

#### script 标签的 integrity 属性

integrity 属性是资源完整性规范的一部分，它允许你为 script 提供一个 hash，用来进行验签，检验加载的JavaScript 文件是否完整。

```html
<script crossorigin="anonymous" integrity="sha256-PJJrxrJLzT6CCz1jDfQXTRWOO9zmemDQbmLtSlFQluc=" src="https://assets-cdn.github.com/assets/frameworks-3c926bc6b24bcd3e820b3d630df4174d158e3bdce67a60d06e62ed4a515096e7.js"></script>
```

上面的代码来自github源码，integrity="sha256-PJJrxrJLzT6CCz1jDfQXTRWOO9zmemDQbmLtSlFQluc=" 告诉浏览器，使用sha256签名算法对下载的js文件进行计算，并与intergrity提供的摘要签名对比，如果二者不一致，就不会执行这个资源。

intergrity 的作用有：
* 减少由【托管在CDN的资源被篡改】而引入的XSS 风险
* 减少通信过程资源被篡改而引入的XSS风险（同时使用https会更保险）
* 可以通过一些技术手段，不执行有脏数据的CDN资源，同时去源站下载对应资源

#### script 标签的 crossorigin 属性
crossorigin 的属性值可以是 anonymous、use-credentials，如果没有属性值或者非法属性值，会被浏览器默认做anonymous。

crossorigin的作用有三个：

* crossorigin 会让浏览器启用CORS访问检查，检查http相应头的 Access-Control-Allow-Origin
* 对于传统 script 需要跨域获取的js资源，控制暴露出其报错的详细信息
* 对于 module script ，控制用于跨域请求的凭据模式

通过使用 crossorigin 属性可以使跨域js暴露出跟同域js同样的报错信息。但是，资源服务器必须返回一个 Access-Control-Allow-Origin 的header，否则资源无法访问。

#### js动态添加 script 标签

可以用js将script动态的append到文档当中，其会异步执行（可以理解为默认拥有async属性），如果需要加载的js按顺序执行，需要设置async为false。

注意：通过 innerHTML 动态添加到页面上的 script 标签则不会被执行。

#### css加载解析
* css加载不会阻塞Dom树的解析
* css加载会阻塞DOM树的渲染（因为，如果没有CSSOM,先对DOM树进行渲染，DOM树后面可能需要重新绘制或回流，造成一些没必要的损耗）
* css加载会阻塞后面js语句的执行（如果样式下载受阻，那么也将阻塞后面js的下载和执行，因为script脚本在执行过程中可能会引用到相关样式）

#### css和javascript加载总结

CSS 优先：引入顺序上，CSS 资源先于 JavaScript 资源。JS置后：我们通常把JS代码放到页面底部，且JavaScript 应尽量少影响 DOM 的构建。

当解析html的时候，会把新来的元素插入dom树里面，同时去查找css，然后把对应的样式规则应用到元素上，查找样式表是按照从右到左的顺序去匹配的。

例如： div p {font-size: 16px}，会先寻找所有p标签并判断它的父标签是否为div之后才会决定要不要采用这个样式进行渲染）。所以，我们平时写CSS时，尽量用id和class，千万不要过渡层叠。

根据渲染树布局，计算CSS样式，即每个节点在页面中的大小和位置等几何信息。HTML默认是流式布局的，CSS和js会打破这种布局，改变DOM的外观样式以及大小和位置。这时就要提到两个重要概念：replaint和reflow。

**replaint（重绘）**：屏幕的一部分重画，不影响整体布局，比如某个CSS的背景色变了，但元素的几何尺寸和位置不变。

**reflow（回流/重排）**：意味着元件的几何尺寸变了，我们需要重新验证并计算渲染树。是渲染树的一部分或全部发生了变化。这就是Reflow，或是Layout。

reflow 会从 这个 root frame 开始递归往下，依次计算所有的结点几何尺寸和位置。reflow 几乎是无法避免的。现在界面上流行的一些效果，比如树状目录的折叠、展开（实质上是元素的显 示与隐藏）等，都将引起浏览器的 reflow。鼠标滑过、点击……只要这些行为引起了页面上某些元素的占位面积、定位方式、边距等属性的变化，都会引起它内部、周围甚至整个页面的重新渲 染。通常我们都无法预估浏览器到底会 reflow 哪一部分的代码，它们都彼此相互影响着。

注意：

* display:none 的节点不会被加入Render Tree，而visibility: hidden 则会，所以，如果某个节点最开始是不显示的，设为display:none是更优的。
* display:none 会触发 reflow，而 visibility:hidden 只会触发 repaint，因为没有发现位置变化。
* 某些情况下，比如修改了元素的样式，浏览器并不会立刻reflow 或 repaint 一次，而是会把这样的操作积攒一批，然后做一次 reflow，这又叫异步 reflow 或增量异步 reflow。但是在有些情况下，比如resize 窗口，改变了页面默认的字体等。对于这些操作，浏览器会马上进行 reflow。

**重排必定会引发重绘，但重绘不一定会引发重排。**

触发重排的条件：任何页面布局和几何属性的改变都会触发重排，比如：

* 页面渲染初始化；(无法避免)
* 添加或删除可见的DOM元素；
* 元素位置的改变，或者使用动画；
* 元素尺寸的改变——大小，外边距，边框；
* 浏览器窗口尺寸的变化（resize事件发生时）；
* 填充内容的改变，比如文本的改变或图片大小改变而引起的计算值宽度和高度的改变；
* 读取某些元素属性：（offsetLeft/Top/Height/Width,　clientTop/Left/Width/Height,　scrollTop/Left/Width/Height,　width/height,　getComputedStyle(),　currentStyle(IE)　)

**重绘重排的代价：耗时，导致浏览器卡慢。**

**优化**

浏览器有自己的优化，浏览器会维护1个队列，把所有会引起回流、重绘的操作放入这个队列，等队列中的操作到了一定的数量或者到了一定的时间间隔，浏览器就会flush队列，进行一个批处理。这样就会让多次的回流、重绘变成一次回流重绘。

我们要减少重绘和重排就是要减少对渲染树的操作，则我们可以合并多次的DOM和样式的修改。并减少对style样式的请求。

* 直接改变元素的className
* display：none；先设置元素为display：none；然后进行页面布局等操作；设置完成后将元素设置为display：block；这样的话就只引发两次重绘和重排；
* 不要经常访问浏览器的flush队列属性；如果一定要访问，可以利用缓存。将访问的值存储起来，接下来使用就不会再引发回流；
* 使用cloneNode(true or false) 和 replaceChild 技术，引发一次回流和重绘；
* 将需要多次重排的元素，position属性设为absolute或fixed，元素脱离了文档流，它的变化不会影响到其他元素；
* 如果需要创建多个DOM节点，可以使用DocumentFragment创建完后一次性的加入document；
* 尽量不要使用table布局。
