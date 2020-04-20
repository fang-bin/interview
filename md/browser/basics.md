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
* Cookie、LocalStorage 和 IndexDB 无法读取
* DOM 无法获得
* AJAX 请求不能发送

##### 如何绕过同源策略，共享cookie

Cookie 是服务器写入浏览器的一小段信息，只有同源的网页才能共享。但是，两个网页一级域名相同，只是二级域名不同，浏览器允许通过设置`document.domain`共享 Cookie。

另外，服务器也可以在设置Cookie的时候，指定Cookie的所属域名为一级域名，比如.example.com。 

    Set-Cookie: key=value; domain=.example.com; path=/

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

    var src = originURL + '#' + data;
    document.getElementById('myIFrame').src = src;

子窗口通过监听hashchange事件得到通知。

    window.onhashchange = checkMessage;
    function checkMessage() {
      var message = window.location.hash;
      // ...
    }

同样的，子窗口也可以改变父窗口的片段标识符。

    parent.location.href= target + "#" + hash;

##### window.name实现跨域窗口通信
浏览器窗口有window.name属性。这个属性的最大特点是，无论是否同源，只要在同一个窗口里，前一个网页设置了这个属性，后一个网页可以读取它。

这种方法的优点是，window.name容量很大，可以放置非常长的字符串；缺点是必须监听子窗口window.name属性的变化，影响网页性能。

##### window.postMessage
HTML5引入了window.postMessage，允许跨窗口通信，不论这两个窗口是否同源。

父窗口http://aaa.com向子窗口http://bbb.com发消息，调用postMessage方法

    var popup = window.open('http://bbb.com', 'title');
    popup.postMessage('Hello World!', 'http://bbb.com');

postMessage方法的第一个参数是具体的信息内容，第二个参数是接收消息的窗口的源（origin），即"协议 + 域名 + 端口"。也可以设为*，表示不限制域名，向所有窗口发送。

子窗口向父窗口发送消息的写法类似

    window.opener.postMessage('Nice to see you', 'http://aaa.com');

父窗口和子窗口都可以通过message事件，监听对方的消息。

    window.addEventListener('message', function(e) {
      console.log(e.data);
    },false);

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

###### WebSocket
WebSocket是一种通信协议，使用ws://（非加密）和wss://（加密）作为协议前缀。该协议不实行同源政策，只要服务器支持，就可以通过它进行跨源通信。

###### CORS
CORS是跨源资源分享（Cross-Origin Resource Sharing）的缩写。

CORS是跨源AJAX请求的根本解决方法。相比JSONP只能发GET请求，CORS允许任何类型的请求。








