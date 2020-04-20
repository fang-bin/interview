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



