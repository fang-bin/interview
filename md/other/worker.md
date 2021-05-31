# Web Worker

Web Worker 的作用，就是为 JavaScript 创造多线程环境，允许主线程创建 Worker 线程，将一些任务分配给后者运行。在主线程运行的同时，Worker 线程在后台运行，两者互不干扰。等到 Worker 线程完成计算任务，再把结果返回给主线程。这样的好处是，一些计算密集型或高延迟的任务，被 Worker 线程负担了，主线程（通常负责 UI 交互）就会很流畅，不会被阻塞或拖慢。

Worker 线程一旦新建成功，就会始终运行，不会被主线程上的活动（比如用户点击按钮、提交表单）打断。这样有利于随时响应主线程的通信。但是，这也造成了 Worker 比较耗费资源，不应该过度使用，而且一旦使用完毕，就应该关闭。

Web Worker 有以下几个使用注意点:

1. 同源策略 (分配给 Worker 线程运行的脚本文件，必须与主线程的脚本文件同源。)

2. DOM 限制(无法操纵主线程所在页面的DOM，无法使用大部分document、window等对象)

3. 主线程和 worker 线程相互之间使用 postMessage() 方法来发送信息, 并且通过 onmessage 这个 event handler来接收信息（传递的信息包含在 Message 这个事件的data属性内) 。数据的交互方式为传递副本，而不是直接共享数据。

4. 脚本限制（Worker 线程不能执行alert()方法和confirm()方法，但可以使用 XMLHttpRequest 对象发出 AJAX 请求。）

5. 文件限制(Worker 线程无法读取本地文件，即不能打开本机的文件系统（file://），它所加载的脚本，必须来自网络。)

### Dedicated Worker

Dedicated Worker 只能为一个页面所使用，而 Shared Worker 则可以被多个页面所共享。

### 同页面 Web Worker

```javascript
function createWorker (f){
  const blob = new Blob(['(' + f.toString() + ')()']);
  const url = window.URL.createObjectURL(blob);
  // 先将嵌入网页的脚本代码，转成一个二进制对象，然后为这个二进制对象生成 URL，再让 Worker 加载这个 URL。这样就做到了，主线程和 Worker 的代码都在同一个网页上面。
  const worker = new Worker(url);
  return worker;
}

const calcWorker = createWorker(function (){
  self.onmessage = function (event){
    calc(event.data);
    function calc (t){
      var timer = setInterval(() => {
        t++;
        if (t >= 20) {
          clearInterval(timer);
          self.postMessage(t);
          self.close();
        }
      }, 1000);
    }
  }
});

calcWorker.onmessage = function (event){
  console.log('结束了', event.data);
}
calcWorker.postMessage(10);
```

###### 扩展 Data URLs

Data URLs，即前缀为 data: 协议的URL，其允许内容创建者向文档中嵌入小文件。

### API

##### 主线程

`new Worker(jsUrl, options);`

Worker()构造函数，可以接受两个参数。第一个参数是脚本的网址（必须遵守同源政策），该参数是必需的，且只能加载 JS 脚本，否则会报错。第二个参数是配置对象，该对象可选。它的一个作用就是指定 Worker 的名称，用来区分多个 Worker 线程。

```javascript
// 主线程
var myWorker = new Worker('worker.js', { name : 'myWorker' });

// Worker 线程
self.name // myWorker
```

* Worker.onerror：指定 error 事件的监听函数。
* Worker.onmessage：指定 message 事件的监听函数，发送过来的数据在Event.data属性中。
* Worker.onmessageerror：指定 messageerror 事件的监听函数。发送的数据无法序列化成字符串时，会触发这个事件。
* Worker.postMessage()：向 Worker 线程发送消息。
* Worker.terminate()：立即终止 Worker 线程。

##### Worker 线程

* self.name： Worker 的名字。该属性只读，由构造函数指定。
* self.onmessage：指定message事件的监听函数。
* self.onmessageerror：指定 messageerror 事件的监听函数。发送的数据无法序列化成字符串时，会触发这个事件。
* self.close()：关闭 Worker 线程。
* self.postMessage()：向产生这个 Worker 线程发送消息。
* self.importScripts()：加载 JS 脚本。

### Shared Worker

一个共享 Worker 是一种特殊类型的 Worker，可以被多个浏览上下文访问，比如多个 windows，iframes 和 workers，但这些浏览上下文必须同源。相比 dedicated workers，它们拥有不同的作用域。

在浏览器中每个相同的JavaScript(同一个sharedWorker文件)只存在一个SharedWorker进程，不管它被创建多少次。

[simple-shared-worker](https://github.com/mdn/simple-shared-worker)

##### 主线程

* SharedWorker.onerror
* SharedWorker.port
    SharedWorker.port.start();
    SharedWorker.port.postMessage();
    SharedWorker.port.onmessage

**注意**: Shared Worker如果用来共享数据，则生成SharedWorker实例时引用地址要完全一致（包括hash）,并且不能设置name，或者name要一致，否则，并不会生成同一个worker。


##### Worker 线程

与常规的 Worker 不同，首先我们需要使用 onconnect 方法等待连接，然后我们获得一个端口，该端口是我们与窗口之间的连接。

```javascript
onconnect = function(e) {
  var port = e.ports[0];

  port.onmessage = function(e) {
    var workerResult = 'Result: ' + (e.data);
    port.postMessage(workerResult);
  }
}
```

#### 调试 Shared Worker 和 Service Worker 

在实际项目开发过程中，若需要调试 Shared Worker / Service Worker 中的脚本，可以通过 chrome://inspect 来进行调试.

# Service Worker

Service workers 本质上充当 Web 应用程序、浏览器与网络（可用时）之间的代理服务器。这个 API 旨在创建有效的离线体验，它会拦截网络请求并根据网络是否可用采取来适当的动作、更新来自服务器的的资源。它还提供入口以推送通知和访问后台同步 API。

Service worker是一个注册在指定源和路径下的事件驱动worker。

它采用JavaScript控制关联的页面或者网站，拦截并修改访问和资源请求，细粒度地缓存资源。你可以完全控制应用在特定情形（最常见的情形是网络不可用）下的表现。

Service worker运行在worker上下文，因此它不能访问DOM。

相对于驱动应用的主JavaScript线程，它运行在其他线程中，所以不会造成阻塞。

它设计为完全异步，同步API（如XHR和localStorage）不能在service worker中使用。

出于安全考量，Service workers只能由HTTPS承载，毕竟修改网络请求的能力暴露给中间人攻击会非常危险。

**Service Worker 也受同源策略的影响**

##### Service Worker也可以用来做这些事情

* 后台数据同步
* 响应来自其它源的资源请求
* 集中接收计算成本高的数据更新，比如地理位置和陀螺仪信息，这样多个页面就可以利用同一组数据
* 在客户端进行CoffeeScript，LESS，CJS/AMD等模块编译和依赖管理（用于开发目的）
* 后台服务钩子
* 自定义模板用于特定URL模式
* 性能增强，比如预取用户可能需要的资源，比如相册中的后面数张图片

### Web Worker 和 Service Worker的异同

两者的共同点是它们无权访问 DOM、使用 postMessage 进行通信。

可以将 Service Worker 看作是具有扩展功能的 Web Worker。

![web-worker](https://github.com/fang-bin/interview/blob/master/image/web-worker.png)

#### Service Worker实现多页面间通信

Service Worker 是一个可以长期运行在后台的 Worker，能够实现与页面的双向通信。多页面共享间的 Service Worker 可以共享，将 Service Worker 作为消息的处理中心（中央站）即可实现广播效果。

```javascript
// 注册
navigator.serviceWorker.register('./service-worker.js').then(function () {
    console.log('Service Worker 注册成功');
})

// A
navigator.serviceWorker.addEventListener('message', function (e) {
    console.log(e.data)
});

// B
navigator.serviceWorker.controller.postMessage('Hello A');


// service-worker.js
self.addEventListener('message', function (e) {
    console.log('service worker receive message', e.data);
    e.waitUntil(
        self.clients.matchAll().then(function (clients) {
            if (!clients || clients.length === 0) {
                return;
            }
            clients.forEach(function (client) {
                client.postMessage(e.data);
            });
        })
    );
});
```

之后有时间会详细研究一下service worker。。。

### App Cahce（已废弃）

废弃的原因：

1. 更新机制
  一旦你采用了manifest之后，你将不能清空这些缓存，只能更新缓存，或者得用户自己去清空这些缓存。这里一旦更新到错误的页面，将被缓存起来，而无法有机会访问到正确的页面，那么就只能杯具了，所以保证更新的页面资源的正确性显得尤为重要。另外电信之类的运营商很喜欢在一些流量大的网站进行劫持广告，这样的话，很可能在更新过程将这些广告给缓存起来了，那就杯具了。
2. manifest本身的编写要求比较严格，要注意换行跟路径文件名之类的问题。不然缓存将无效。
3. 如果更新的资源中有一个资源更新失败了，将导致全部更新失败，将用回上一版本的缓存。
4. 二次更新的问题，即在更新新版本过程中，用户需要第一次时访问的还是旧的资源，需要第二次进去才是新的资源。如果此时后台接口发生变化，访问第一次时的旧数据很可能将出现兼容问题。
5. 限制大小问题，一般是最多缓存5mb，不过一般是够用的。
6. 回滚问题


### postMessage API

##### 发送消息

```javascript
otherWindow.postMessage(message, targetOrigin, [transfer]);
```

###### otherWindow

其他窗口的一个引用，比如iframe的contentWindow属性、执行window.open返回的窗口对象、或者是命名过或数值索引的window.frames。

###### message

将要发送到其他 window的数据。它将会被结构化克隆算法序列化。这意味着你可以不受什么限制的将数据对象安全的传送给目标窗口而无需自己序列化。

###### targetOrigin

通过窗口的origin属性来指定哪些窗口能接收到消息事件，其值可以是字符串`"*"`（表示无限制）或者一个URI。在发送消息的时候，如果目标窗口的协议、主机地址或端口这三者的任意一项不匹配targetOrigin提供的值，那么消息就不会被发送；只有三者完全匹配，消息才会被发送。这个机制用来控制消息可以发送到哪些窗口；例如，当用postMessage传送密码时，这个参数就显得尤为重要，必须保证它的值与这条包含密码的信息的预期接受者的origin属性完全一致，来防止密码被恶意的第三方截获。如果你明确的知道消息应该发送到哪个窗口，那么请始终提供一个有确切值的targetOrigin，而不是*。不提供确切的目标将导致数据泄露到任何对数据感兴趣的恶意站点。

##### 接收消息

```javascript
window.addEventListener('message', function (event: {data, origin, source}){}, false);
```

###### data

从其他 window 中传递过来的对象。

###### origin

调用 postMessage  时消息发送方窗口的 origin . 这个字符串由 协议、`“://“`、域名、`“ : 端口号”`拼接而成。例如 `“https://example.org (隐含端口 443)”`、`“http://example.net (隐含端口 80)”`、`“http://example.com:8080”`。请注意，这个origin不能保证是该窗口的当前或未来origin，因为postMessage被调用后可能被导航到不同的位置。

###### source

对发送消息的窗口对象的引用; 您可以使用此来在具有不同origin的两个窗口之间建立双向通信。

#### 安全问题

**如果不希望从其他网站接收message，不要为message事件添加任何事件侦听器。**

如果确实希望从其他网站接收message，**应该始终使用origin和source属性验证发件人的身份**。 任何窗口（包括例如http://evil.example.com）都可以向任何其他窗口发送消息，并且不能保证未知发件人不会发送恶意消息。 但是，验证身份后，仍然应该始**终验证接收到的消息的语法**。 否则，信任只发送受信任邮件的网站中的安全漏洞可能会在网站中打开跨网站脚本漏洞。

**当使用postMessage将数据发送到其他窗口时，始终指定精确的目标origin，而不是`*`。**


### Broadcast Channel

Broadcast Channel API 可以实现同 源 下浏览器不同窗口，Tab页，frame或者 iframe 下的 浏览器上下文 (通常是同一个网站下不同的页面)之间的简单通讯。

###### 创建或加入某个频道

```javascript
// 连接到广播频道
var bc = new BroadcastChannel('test_channel');
```

###### 发送消息

```javascript
// 发送简单消息的示例
bc.postMessage('This is a test message.');
```

###### 接收消息

```javascript
// 简单示例，用于将事件打印到控制台
bc.onmessage = function (ev) { console.log(ev); }
```

###### 与频道断开连接

```javascript
// 断开频道连接
bc.close()
```







