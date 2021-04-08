## AJAX

AJAX的本质是使用XMLHttpRequest对象来请求数据，它是对原生XMLHttpRequest对象的封装。

实例化XMLHttpRequest对象之后，可以通过该实例对象发送请求

### 方法：
##### `XMLHttpRequest.prototype.open(method, url[, async[, user[, password]]])`

这个方法的作用是初始化一个请求，并不会发起真正的请求。

* async 是否异步执行操作  默认为true。如果值为false，send()方法直到收到答复前不会返回。如果true，已完成事务的通知可供事件监听器使用。
* user 可选的用户名用于认证用途；默认为null。
* password 可选的密码用于认证用途，默认为null。

##### `XMLHttpRequest.prototype.send(body)`

发送请求，并接受一个可选参数(在XHR请求中要发送的数据体)

当请求方式为 post 时，可以将请求体的参数传入

当请求方式为 get 时，可以不传或传入 null

不管是 get 还是 post，参数都需要通过 encodeURIComponent 编码后拼接
xhr.send(data)

##### `XMLHttpRequest.prototype.abort()`

如果该请求已被发出，XMLHttpRequest.abort() 方法将终止该请求。

当一个请求被终止，它的  readyState 将被置为 XMLHttpRequest.UNSENT（0）。并且请求的 status 置为 0。

##### `XMLHttpRequest.prototype.setRequestHeader(header, value)`

* header 属性名
* value 属性值

设置HTTP请求头部的方法。此方法必须在  open() 方法和 send()   之间调用。如果多次对同一个请求头赋值，只会生成一个合并了多个值的请求头。

**在通过send方法发送请求后，xhr 对象在收到响应数据时会自动填充到其对应的属性中**

### 属性：

##### `XMLHttpRequest.prototype.timeout`

超时时间(毫秒)，默认值为0，意味着没有超时，而不是真的就是超时时间为0。

在IE中，超时属性可能只能在调用 open() 方法之后且在调用 send() 方法之前设置。

##### `XMLHttpRequest.prototype.onreadystatechange = callback`

只要 readyState 属性发生变化，就会调用相应的处理函数。

当一个 XMLHttpRequest 请求被 abort() 方法取消时，其对应的 readystatechange 事件不会被触发。

其他的一些方法：

**onerror** 当 request 遭遇错误时触发。

**onabort** 当 request 被停止时触发，例如当程序调用 XMLHttpRequest.abort() 时。

**ontimeout** 在预设时间内没有接收到响应时触发。

**onload** XMLHttpRequest请求成功完成时触发。

**loadend** 当请求结束时触发, 无论请求成功 ( load) 还是失败 (abort 或 error)。

**onprogress** 当请求接收到更多数据时，周期性地触发。

##### `XMLHttpRequest.prototype.readyState`

返回一个 XMLHttpRequest  代理当前所处的状态

| 值 | 状态 | 描述 |
| --- | --- | --- |
| 0 | UNSENT | 代理被创建，但尚未调用 open() 方法 |
| 1 | OPENED | open() 方法已经被调用 |
| 2 | HEADERS_RECEIVED | send() 方法已经被调用，并且头部和状态已经可获得 |
| 3 | LOADING | 下载中； responseText 属性已经包含部分数据 |
| 4 | DONE | 下载操作已完成 |

##### `XMLHttpRequest.prototype.status`

返回 XMLHttpRequest 响应中的数字状态码

##### `XMLHttpRequest.prototype.response`

response 属性返回响应的正文。返回的类型为 ArrayBuffer 、 Blob 、 Document 、 JavaScript Object 或 DOMString 中的一个。 这取决于 responseType 属性。

##### `XMLHttpRequest.prototype.responseType`

XMLHttpRequest.responseType 属性是一个枚举类型的属性，返回响应数据的类型。

##### `XMLHttpRequest.prototype.responseText`

只读，返回一个 DOMString，该 DOMString 包含对请求的响应，如果请求未成功或尚未发送，则返回 null。

##### `XMLHttpRequest.prototype.responseXML`

只读，返回一个包含请求检索的HTML或XML的Document，如果请求未成功，尚未发送，或者检索的数据无法正确解析为 XML 或 HTML，则为 null。

封装ajax
```javascript
function ajax (options){
  let opts = Object.assign({}, options);
  opts.type = (options.type || 'GET').toUpperCase();
  const formatParams = (obj) => {
    let arr = [];
    for (const name in obj) {
      if (obj.hasOwnProperty(name)) {
        arr.push(`${encodeURIComponent(name)}=${encodeURIComponent(obj[name])}`);
      }
    }
    arr.push(`v=${Math.random().toString(16).replace('.', '')}`);
    return arr.join('&');
  }
  const params = formatParams(opts.data);
  let xhr = null;
  if (window.XMLHttpRequest){
    xhr = new XMLHttpRequest();
  }else {
    xhr = new ActiveXObject('Microsoft.XMLHTTP');
  }
  xhr.onreadystatechange = function (){
    if (xhr.readyState == 4){
      const status = xhr.status;
      if (status >= 200 && status < 300){
        opts.success && opts.success(xhr.responseText, xhr.responseXML);
      }else {
        opts.fail && opts.fail(status);
      }
    }
  }

  xhr.onabort = function (){
    opts.fail('终止');
  }
  xhr.ontimeout = function (){
    opts.fail('超时');
  }
  xhr.onerror = function (err) {
    opts.fail(err);
  }

  if (opts.type === 'GET'){
    xhr.open('GET', opts.url + '?' + params, true);
    opts.timeout && opts.timeout > 0 && (xhr.timeout = opts.timeout);
    xhr.send(null);
  }else if (opts.type === 'POST'){
    xhr.open('POST', opts.url, true);
    opts.timeout && opts.timeout > 0 && (xhr.timeout = opts.timeout);
    xhr.setRequestHeader("Content-Type", "application/x-www-form-urlencoded");
    xhr.send(params);
  }
}
```

## Fetch

fetch 是全局量 window 的一个方法，号称是ajax的替代品，基于Promise设计，使用Promise来处理结果/回调。其特点是：**语法简单，更加语义化**；**基于标准的Promise实现，支持async/await**；

fetch的问题和解决：
1. fetch的浏览器兼容性问题
  因为fetch是基于Promise实现的，如果要兼容fetch，在低版本浏览器中要同时引入Fetch和Promise的polyfill。

2. 默认不带cookie
  可以配置其credentials项:
    * omit: 默认值，忽略cookie的发送
    * same-origin: 表示cookie只能同域发送，不能跨域发送
    * include: cookie既可以同域发送，也可以跨域发送

3. fetch请求对某些错误http状态不会reject
  这主要是由fetch返回promise导致的，因为fetch返回的promise在某些错误的http状态下如400、500等不会reject，相反它会被resolve；只有网络错误会导致请求不能完成时，fetch 才会被 reject；所以一般会对fetch请求做一层封装，例如下面代码所示：
    ```javascript
    function checkStatus(response) {
      if (response.status >= 200 && response.status < 300) {
        return response;
      }
      const error = new Error(response.statusText);
      error.response = response;
      throw error;
    }
    function parseJSON(response) {
      return response.json();
    }
    export default function request(url, options) {
      let opt = options||{};
      return fetch(url, {credentials: 'include', ...opt})
        .then(checkStatus)
        .then(parseJSON)
        .then((data) => ( data ))
        .catch((err) => ( err ));
    }
    ```
4. fetch不支持超时timeout处理
  可以通过timeout+abort方式来实现，起到真正超时丢弃当前的请求。
  在目前的fetch指导规范中，fetch并不是一个具体实例，而只是一个方法；其返回的promise实例根据Promise指导规范标准是不能abort的，也不能手动改变promise实例的状态，只能由内部来根据请求结果来改变promise的状态。
  既然不能手动控制fetch方法执行后返回的promise实例状态，我们可以创建一个可以手动控制状态的新Promise实例。
  **实现fetch的timeout功能，其思想就是新创建一个可以手动控制promise状态的实例，根据不同情况来对新promise实例进行resolve或者reject，从而达到实现timeout的功能。**

    **方法一：单纯setTimeout方式**
    ```javascript
    var oldFetchfn = fetch; //拦截原始的fetch方法
    window.fetch = function(input, opts){//定义新的fetch方法，封装原有的fetch方法
        return new Promise(function(resolve, reject){
            var timeoutId = setTimeout(function(){
                reject(new Error("fetch timeout"))
            }, opts.timeout);
            oldFetchfn(input, opts).then(
                res=>{
                    clearTimeout(timeoutId);
                    resolve(res)
                },
                err=>{
                    clearTimeout(timeoutId);
                    reject(err)
                }
            )
        })
    }

    //也可以模拟abort功能

    var oldFetchfn = fetch; 
    window.fetch = function(input, opts){
        return new Promise(function(resolve, reject){
            var abort_promise = function(){
                reject(new Error("fetch abort"))
            };
            var p = oldFetchfn(input, opts).then(resolve, reject);
            p.abort = abort_promise;
            return p;
        })
    }
    ```

    **方法二：利用Promise.race方法**
    ```javascript
    var oldFetchfn = fetch; //拦截原始的fetch方法
    window.fetch = function(input, opts){//定义新的fetch方法，封装原有的fetch方法
        var fetchPromise = oldFetchfn(input, opts);
        var timeoutPromise = new Promise(function(resolve, reject){
            setTimeout(()=>{
                reject(new Error("fetch timeout"))
            }, opts.timeout)
        });
        retrun Promise.race([fetchPromise, timeoutPromise])
    }
    ```
5. fetch跨域问题
XHR2是支持跨域请求的，只不过要满足浏览器端支持CORS，服务器通过Access-Control-Allow-Origin来允许指定的源进行跨域，仅此一种方式。

fetch也是支持跨域请求的，只不过其跨域请求做法与XHR2一样，需要客户端与服务端支持；另外，fetch还支持一种跨域，不需要服务器支持的形式，具体可以通过其mode的配置项来说明。

fetch的mode配置项有3个值，如下：

* same-origin：该模式是不允许跨域的，它需要遵守同源策略，否则浏览器会返回一个error告知不能跨域；其对应的response type为basic。

* cors: 该模式支持跨域请求，顾名思义它是以CORS的形式跨域；当然该模式也可以同域请求不需要后端额外的CORS支持；其对应的response type为cors。

* no-cors: 该模式用于跨域请求但是服务器不带CORS响应头，也就是服务端不支持CORS；这也是fetch的特殊跨域请求方式；其对应的response type为opaque。

针对跨域请求，cors模式是常见跨域请求实现，但是fetch自带的no-cors跨域请求模式则较为陌生，该模式有一个比较明显的特点：**该模式允许浏览器发送本次跨域请求，但是不能访问响应返回的内容，这也是其response type为opaque透明的原因。**

这与<img/>发送的请求类似，只是该模式不能访问响应的内容信息；但是它可以被其他APIs进行处理，例如ServiceWorker。另外，该模式返回的repsonse可以在Cache API中被存储起来以便后续的对它的使用，这点对script、css和图片的CDN资源是非常合适的，因为这些资源响应头中都没有CORS头。

总的来说，fetch的跨域请求是使用CORS方式，需要浏览器和服务端的支持。

6. fetch不支持progress事件

XHR是原生支持progress事件的，例如下面代码这样：

```javascript
var xhr = new XMLHttpRequest()
xhr.open('POST', '/uploads')
xhr.onload = function() {}
xhr.onerror = function() {}
function updateProgress (event) {
  if (event.lengthComputable) {
    var percent = Math.round((event.loaded / event.total) * 100)
    console.log(percent)
  }
xhr.upload.onprogress =updateProgress; //上传的progress事件
xhr.onprogress = updateProgress; //下载的progress事件
}
xhr.send();
```

但是fetch是不支持有关progress事件的，不过其内部设计实现了Request和Response类；其中Response封装一些方法和属性，通过Response实例可以访问这些方法和属性，例如response.json()、response.body等等。

值得关注的地方是，response.body是一个可读字节流对象，其实现了一个getRender()方法，其具体作用是：getRender()方法用于读取响应的原始字节流，该字节流是可以循环读取的，直至body内容传输完成。

因此，利用到这点可以模拟出fetch的progress：

fetch(url).then(response => {
  var reader = response.body.getReader();
  var bytesReceived = 0;
  reader.read().then(function processResult(result) {
    if (result.done) {
      console.log("Fetch complete");
      return;
    }
    bytesReceived += result.value.length;
    return reader.read().then(processResult);
  });
});

## Axios

[axios文档，推荐直接看此文档](https://www.kancloud.cn/yunye/axios/234845)

Axios 是一个基于 promise 的 HTTP 库，可以用在浏览器和 node.js 中，它也是对原生XMLHttpRequest对象的封装。

特性:

* 从浏览器中创建 XMLHttpRequests
* 从 node.js 创建 http 请求
* 支持 Promise API
* 拦截请求和响应
* 转换请求数据和响应数据
* 取消请求
* 自动转换 JSON 数据
* 客户端支持防御 XSRF

#### axios的配置

```javascript
{
  // `url` 是用于请求的服务器 URL
  url: '/user',

  // `method` 是创建请求时使用的方法
  method: 'get', // 默认是 get

  // `baseURL` 将自动加在 `url` 前面，除非 `url` 是一个绝对 URL。
  // 它可以通过设置一个 `baseURL` 便于为 axios 实例的方法传递相对 URL
  baseURL: 'https://some-domain.com/api/',

  // `transformRequest` 允许在向服务器发送前，修改请求数据
  // 只能用在 'PUT', 'POST' 和 'PATCH' 这几个请求方法
  // 后面数组中的函数必须返回一个字符串，或 ArrayBuffer，或 Stream
  transformRequest: [function (data) {
    // 对 data 进行任意转换处理

    return data;
  }],

  // `transformResponse` 在传递给 then/catch 前，允许修改响应数据
  transformResponse: [function (data) {
    // 对 data 进行任意转换处理

    return data;
  }],

  // `headers` 是即将被发送的自定义请求头
  headers: {'X-Requested-With': 'XMLHttpRequest'},

  // `params` 是即将与请求一起发送的 URL 参数
  // 必须是一个无格式对象(plain object)或 URLSearchParams 对象
  params: {
    ID: 12345
  },

  // `paramsSerializer` 是一个负责 `params` 序列化的函数
  // (e.g. https://www.npmjs.com/package/qs, http://api.jquery.com/jquery.param/)
  paramsSerializer: function(params) {
    return Qs.stringify(params, {arrayFormat: 'brackets'})
  },

  // `data` 是作为请求主体被发送的数据
  // 只适用于这些请求方法 'PUT', 'POST', 和 'PATCH'
  // 在没有设置 `transformRequest` 时，必须是以下类型之一：
  // - string, plain object, ArrayBuffer, ArrayBufferView, URLSearchParams
  // - 浏览器专属：FormData, File, Blob
  // - Node 专属： Stream
  data: {
    firstName: 'Fred'
  },

  // `timeout` 指定请求超时的毫秒数(0 表示无超时时间)
  // 如果请求话费了超过 `timeout` 的时间，请求将被中断
  timeout: 1000,

  // `withCredentials` 表示跨域请求时是否需要使用凭证
  withCredentials: false, // 默认的

  // `adapter` 允许自定义处理请求，以使测试更轻松
  // 返回一个 promise 并应用一个有效的响应 (查阅 [response docs](#response-api)).
  adapter: function (config) {
    /* ... */
  },

  // `auth` 表示应该使用 HTTP 基础验证，并提供凭据
  // 这将设置一个 `Authorization` 头，覆写掉现有的任意使用 `headers` 设置的自定义 `Authorization`头
  auth: {
    username: 'janedoe',
    password: 's00pers3cret'
  },

  // `responseType` 表示服务器响应的数据类型，可以是 'arraybuffer', 'blob', 'document', 'json', 'text', 'stream'
  responseType: 'json', // 默认的

  // `xsrfCookieName` 是用作 xsrf token 的值的cookie的名称
  xsrfCookieName: 'XSRF-TOKEN', // default

  // `xsrfHeaderName` 是承载 xsrf token 的值的 HTTP 头的名称
  xsrfHeaderName: 'X-XSRF-TOKEN', // 默认的

  // `onUploadProgress` 允许为上传处理进度事件
  onUploadProgress: function (progressEvent) {
    // 对原生进度事件的处理
  },

  // `onDownloadProgress` 允许为下载处理进度事件
  onDownloadProgress: function (progressEvent) {
    // 对原生进度事件的处理
  },

  // `maxContentLength` 定义允许的响应内容的最大尺寸
  maxContentLength: 2000,

  // `validateStatus` 定义对于给定的HTTP 响应状态码是 resolve 或 reject  promise 。如果 `validateStatus` 返回 `true` (或者设置为 `null` 或 `undefined`)，promise 将被 resolve; 否则，promise 将被 rejecte
  validateStatus: function (status) {
    return status >= 200 && status < 300; // 默认的
  },

  // `maxRedirects` 定义在 node.js 中 follow 的最大重定向数目
  // 如果设置为0，将不会 follow 任何重定向
  maxRedirects: 5, // 默认的

  // `httpAgent` 和 `httpsAgent` 分别在 node.js 中用于定义在执行 http 和 https 时使用的自定义代理。允许像这样配置选项：
  // `keepAlive` 默认没有启用
  httpAgent: new http.Agent({ keepAlive: true }),
  httpsAgent: new https.Agent({ keepAlive: true }),

  // 'proxy' 定义代理服务器的主机名称和端口
  // `auth` 表示 HTTP 基础验证应当用于连接代理，并提供凭据
  // 这将会设置一个 `Proxy-Authorization` 头，覆写掉已有的通过使用 `header` 设置的自定义 `Proxy-Authorization` 头。
  proxy: {
    host: '127.0.0.1',
    port: 9000,
    auth: : {
      username: 'mikeymike',
      password: 'rapunz3l'
    }
  },

  // `cancelToken` 指定用于取消请求的 cancel token
  // （查看后面的 Cancellation 这节了解更多）
  cancelToken: new CancelToken(function (cancel) {
  })
}
```

#### axios返回响应结构
```javascript
{
  // `data` 由服务器提供的响应
  data: {},

  // `status` 来自服务器响应的 HTTP 状态码
  status: 200,

  // `statusText` 来自服务器响应的 HTTP 状态信息
  statusText: 'OK',

  // `headers` 服务器响应的头
  headers: {},

  // `config` 是为请求提供的配置信息
  config: {}
}
```

#### axios的请求方法别名都有哪些

* axios.request(config)
* axios.get(url[, config])
* axios.delete(url[, config])
* axios.head(url[, config])
* axios.post(url[, data[, config]])
* axios.put(url[, data[, config]])
* axios.patch(url[, data[, config]])

在使用别名方法时， url、method、data 这些属性都不必在配置中指定。

#### 配置设置
**全局的 axios 默认值**

```javascript
axios.defaults.baseURL = 'https://api.example.com';
axios.defaults.headers.common['Authorization'] = AUTH_TOKEN;
axios.defaults.headers.post['Content-Type'] = 'application/x-www-form-urlencoded';
```

**自定义实例默认值**

```javascript
// 创建实例时设置配置的默认值
var instance = axios.create({
  baseURL: 'https://api.example.com'
});

// 在实例已创建后修改默认值
instance.defaults.headers.common['Authorization'] = AUTH_TOKEN;
```

**配置的优先顺序**
配置会以一个优先顺序进行合并。这个顺序是：在 lib/defaults.js 找到的库的默认值，然后是实例的 defaults 属性，最后是请求的 config 参数。后者将优先于前者。这里是一个例子：

```javascript
// 使用由库提供的配置的默认值来创建实例
// 此时超时配置的默认值是 `0`
var instance = axios.create();

// 覆写库的超时默认值
// 现在，在超时前，所有请求都会等待 2.5 秒
instance.defaults.timeout = 2500;

// 为已知需要花费很长时间的请求覆写超时设置
instance.get('/longRequest', {
  timeout: 5000
});
```

#### 拦截器
在请求或响应被 then 或 catch 处理前拦截它们。

```javascript
// 添加请求拦截器
axios.interceptors.request.use(function (config) {
    // 在发送请求之前做些什么
    return config;
  }, function (error) {
    // 对请求错误做些什么
    return Promise.reject(error);
  });

// 添加响应拦截器
axios.interceptors.response.use(function (response) {
    // 对响应数据做点什么
    return response;
  }, function (error) {
    // 对响应错误做点什么
    return Promise.reject(error);
  });
```
如果你想在稍后移除拦截器，可以这样：

```javascript
var myInterceptor = axios.interceptors.request.use(function () {/*...*/});
axios.interceptors.request.eject(myInterceptor);
```

可以为自定义 axios 实例添加拦截器

```javascript
var instance = axios.create();
instance.interceptors.request.use(function () {/*...*/});
```

#### 错误处理
```javascript
axios.get('/user/12345')
  .catch(function (error) {
    if (error.response) {
      // 请求已发出，但服务器响应的状态码不在 2xx 范围内
      console.log(error.response.data);
      console.log(error.response.status);
      console.log(error.response.headers);
    } else {
      // Something happened in setting up the request that triggered an Error
      console.log('Error', error.message);
    }
    console.log(error.config);
  });
```

可以使用 validateStatus 配置选项定义一个自定义 HTTP 状态码的错误范围。

```javascript
axios.get('/user/12345', {
  validateStatus: function (status) {
    return status < 500; // 状态码在大于或等于500时才会 reject
  }
})
```

#### 取消
使用 cancel token 取消请求， 可以使用 CancelToken.source 工厂方法创建 cancel token，像这样：

```javascript
var CancelToken = axios.CancelToken;
var source = CancelToken.source();

axios.get('/user/12345', {
  cancelToken: source.token
}).catch(function(thrown) {
  if (axios.isCancel(thrown)) {
    console.log('Request canceled', thrown.message);
  } else {
    // 处理错误
  }
});

// 取消请求（message 参数是可选的）
source.cancel('Operation canceled by the user.');
```

还可以通过传递一个 executor 函数到 CancelToken 的构造函数来创建 cancel token：

```javascript
var CancelToken = axios.CancelToken;
var cancel;

axios.get('/user/12345', {
  cancelToken: new CancelToken(function executor(c) {
    // executor 函数接收一个 cancel 函数作为参数
    cancel = c;
  })
});

// 取消请求
cancel();
```

注意：可以使用同一个 cancel token 取消多个请求。
