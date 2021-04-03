## 1. 为什么使用Object.create(null) 而不使用 {}

主要两点考虑:
* 使用create创建的对象，没有任何属性，显示No properties，我们可以把它当作一个非常纯净的map来使用，我们可以自己定义hasOwnProperty、toString方法，不管是有意还是不小心，我们完全不必担心会将原型链上的同名方法覆盖掉。
* 在我们使用for..in循环的时候会遍历对象原型链上的属性，使用create(null)就不必再对属性进行检查了，当然，我们也可以直接使用Object.keys[]

**可枚举性和是否查找原型总结**

* `for in` 会遍历对象包括其原型链上所有可枚举性的属性(不包含 Symbol 属性)
* `hasOwnProperty` 不会去查找原型，只会检查属性是否在当前对象中
* `Object.keys() Object.values() Object.entries()` 不会去查找原型，只会遍历对象本身可枚举（不包含 Symbol）属性（属性对）
* `Object.getOwnPropertyNames()` 不会去查找原型，会遍历对象本身所有的属性（无论是否具有可枚举性）不包含Symbol属性
* `Object.prototype.propertyIsEnumerable` 只会检查属性在该对象上是否可枚举
* `Object.getOwnPropertyDescriptors()` 获取一个对象的所有自身属性的描述符
* `Object.getOwnPropertyDescriptor()` 获取对象自身某个属性的描述符
* `Object.getOwnPropertySymbols()` 获取一个给定对象自身的所有 Symbol 属性的数组
* `Object.assign(target, ...source)`只拷贝自身的可枚举属性，会忽略`enumerable`为`false`的属性(仅限于source对象，target对象的不可枚举属性，也会拷贝) **Symbol属性可以被拷贝**
* `JSON.stringify()` 只串行化对象自身的可枚举的属性
* `Reflect.ownKeys(obj)` Reflect.ownKeys返回一个数组，包含对象自身的（不含继承的）所有键名，不管键名是 Symbol 或字符串，也不管是否可枚举。

`for in` `Object.keys()` `Object.getOwnPropertyNames()` `Object.getOwnPropertySymbols()` `Reflect.ownKeys()`遍历对象的键名，都遵守同样的属性遍历的次序规则:

* 首先遍历所有数值键，按照数值升序排列。
* 其次遍历所有字符串键，按照加入时间升序排列。
* 最后遍历所有 Symbol 键，按照加入时间升序排列。

## 2. Object.defineProperty(obj, prop, descriptor) 和 Object.defineProperties(obj, props)

数据描述符
* configurable 
当且仅当该属性的 configurable 键值为 true 时，该属性的描述符才能够被改变，同时该属性也能从对应的对象上被删除。
* enumerable
当且仅当该属性的 enumerable 键值为 true 时，该属性才会出现在对象的枚举属性中。
* value
可选值，该属性对应的值。
* writable
可选值，当且仅当该属性的 writable 键值为 true 时，属性的值，也就是上面的 value，才能被赋值运算符改变。

存取描述符
* get
执行时不传入任何参数，但是会传入 this 对象（由于继承关系，这里的this并不一定是定义该属性的对象）
* set
当属性值被修改时，会调用此函数。该方法接受一个参数（也就是被赋予的新值），会传入赋值时的 this 对象。

描述符使用场景

|  | configurable | enumerable | value | writable | get | set |
| :---: | :---: | :---: | :---: | :---: | :---: | :---: |
| 数据描述符 | √ | √ | √ | √ | × | × |
| 存取描述符 | √ | √ | × | × | √ | √ |

注意: configurable、enumerable、writable 默认为false, value、set、get 默认为undefined


额外补充:
* Iterator 的作用有三个：一是为各种数据结构，提供一个统一的、简便的访问接口；二是使得数据结构的成员能够按某种次序排列；三是 ES6 创造了一种新的遍历命令for...of循环，Iterator 接口主要供for...of消费。

## 3. Promises

* Promises/A+规范 
* Promise的值传递
* Promise解决什么问题
* Promise有什么缺点

Promise的缺点:

* 无法取消Promise，一旦新建它就会立即执行，无法中途取消。
* 如果不设置回调函数，Promise内部抛出的错误，不会反应到外部。
* 当处于pending状态时，无法得知目前进展到哪一个阶段（刚刚开始还是即将完成）

针对 Promise 无法取消的问题，我们可以使用 Promise.race 来自己封装中断方法

```javascript
function abortPromise(promise) {
  let _abort = undefined;
  const _abort_promise = new Promise((resolve, reject) => {
    _abort = reject;
  });
  let p = Promise.race([_abort_promise, promise]);
  p.abort = _abort;
  return p;
}

const p1 = new Promise(resolve => {
  setTimeout(() => {
    resolve('结果出现了');
  }, 5000);
});

const p2 = abortPromise(p1);

p2.then(res => {
  console.log(res);
}).catch(err => console.log(err));

setTimeout(() => {
  p2.abort('错误了');
}, 2000);
```

补充说明: 因为Promise 是没有中断方法的，xhr.abort()、ajax 有自己的中断方法，axios 是基于 ajax 实现的；fetch 基于 promise，所以它的请求是无法中断的。

之前对Promise实现比较好的库 q、bluebird、when等

## 4. js中的类型
javascript的7中基本类型: undefined、null、boolean、string、number、symbol、bigint

复杂类型是Object，其中对象类型包括: Array、Function，还有三个特殊的对象 RegExp（正则）、Date(日期)、Error(错误)


## 5. meta标签

meta 标签常用于定义页面的说明、关键字、最后修改日期等元数据，这些元数据服务于浏览器（如何布局或重载页面）、搜索引擎或其他网络服务。

其定义的元数据的类型包括:

  * 设置了 name 属性，meta 元素提供的是文档级别（document-level）的元数据，应用于整个页面。
  * 设置了 http-equiv 属性，meta 元素则是编译指令，提供的信息与类似命名的HTTP头部相同。
  * 设置了 charset 属性，meta 元素是一个字符集声明，告诉文档使用哪种字符编码。
  * 设置了 itemprop 属性，meta 元素提供用户定义的元数据。

**name 属性的一些相关定义**

  * `revisit-after`

      如果页面不是经常更新，为了减轻搜索引擎爬虫对服务器带来的压力，可以设置一个爬虫的重访时间。如果重访时间过短，爬虫将按它们定义的默认时间来访问。 `<meta name="revisit-after" content="7 days" >`

  * `renderer`

      renderer是为双核浏览器准备的，用于指定双核浏览器默认以何种方式渲染页面。比如说360浏览器。`<meta name="renderer" content="webkit">`、`<meta name="renderer" content="ie-comp">`、`<meta name="renderer" content="ie-stand">`

  * `referrer`

      referrer 控制document发起的Request请求中附加的Referer HTTP header，content中的值为:

      * `no-referrer-when-downgrade`（默认值）当请求安全级别下降时不发送 referrer。目前，只有一种情况会发生安全级别下降，即从 HTTPS 到 HTTP。HTTPS 到 HTTP 的资源引用和链接跳转都不会发送 referrer。
      * `no-referrer` 所有请求不发送 referrer
      * `same-origin` 对于同源的链接和引用，会发送referrer，其他的不会。
      * `origin` 在任何情况下仅发送源信息作为引用地址。源信息包括访问协议和域名。
      * `unsafe-url` 无论是否发生协议降级，无论是本站链接还是站外链接，统统都发送 Referrer 信息。正如其名，这是最宽松而最不安全的策略。
      * `origin-when-cross-origin` 同源的链接和引用，会发送完全的 referrer 信息；但非同源链接和引用时，只发送源信息。
      * `strict-origin` 在安全级别下降时不发送 referrer；而在同等安全级别的情况下仅发送源信息。注意：这个是新加的标准，有些浏览器可能还不支持。
      * `strict-origin-when-cross-origin` 同源的链接和引用，会发送 referrer。安全级别下降时不发送 referrer。其它情况下发送源信息。注意：这个是新加的标准，有些浏览器可能还不支持。

  * `format-detection`

      格式检测，用来检测html里的一些格式的，那关于meta的format-detection属性主要是有以下几个设置：`<meta name="format-detection" content="telephone=no">`、`<meta name="format-detection" content="email=no">`、`<meta name="format-detection" content="adress=no" >`

  * `robots`

      robots用来告诉爬虫哪些页面需要索引，哪些页面不需要索引。

      | 值 | 描述 |
      | :---: | :---: |
      | index | 允许robot索引本页面（默认）|
      | noindex | 不允许robot索引本页面 |
      | follow | 允许搜索引擎继续通过此网页的链接索引搜索其它的网页（默认）|
      | nofollow | 搜索引擎不继续通过此网页的链接索引搜索其它的网页 |
      | none | 相当于noindex，nofollow(仅适用于Google) |

  *  `apple-mobile-web-app-status-bar-style`

      针对WebApp全屏模式，隐藏状态栏/设置状态栏颜色，content的值为`default`、`black`、`black-translucent` `<meta name="apple-mobile-web-app-status-bar-style" content="black-translucent" />`

**重点讲一下 http-equiv 属性**，其所有允许的值都是特定HTTP头部的名称，顾名思义，相当于http的文件头作用，它可以向浏览器传回一些有用的信息，以帮助正确和精确地显示网页内容，与之对应的属性值为content，content中的内容其实就是各个参数的变量值。

  * `content-security-policy`

      它允许页面作者定义当前页的内容策略。 内容策略主要指定允许的服务器源和脚本端点，这有助于防止跨站点脚本攻击。

      CSP 的实质就是白名单制度，开发者明确告诉客户端，哪些外部资源可以加载和执行，等同于提供白名单。它的实现和执行全部由浏览器完成，开发者只需提供配置。

      CSP 大大增强了网页的安全性。攻击者即使发现了漏洞，也没法注入脚本，除非还控制了一台列入了白名单的可信主机。

      两种方法可以启用 CSP。一种是通过 HTTP 头信息的Content-Security-Policy的字段。`Content-Security-Policy: script-src 'self'; object-src 'none';style-src cdn.example.org third-party.org; child-src https:`

      另一种是通过网页的<meta>标签。`<meta http-equiv="Content-Security-Policy" content="script-src 'self'; object-src 'none'; style-src cdn.example.org third-party.org; child-src https:">`

      上面代码中，CSP 做了如下配置: 脚本：只信任当前域名、<object>标签：不信任任何URL，即不加载任何资源、样式表：只信任cdn.example.org和third-party.org、框架（frame）：必须使用HTTPS协议加载、其他资源：没有限制。

      启用后，不符合 CSP 的外部资源就会被阻止加载。

      | CPS指令 | 描述 |
      | :---: | :---: |
      | default-src | 默认加载策略 |
      | script-src | 对于JavaScript脚本的加载策略 |
      | style-src | 对于样式的加载策略 |
      | img-src | 图片的加载策略 |
      | object-src | 显示插件来源，如flash |
      | frame-src | iframe来源 |
      | media-src | 媒体 |
      | report-uri | 请求资源不被允许时，向该地址提交日志 |

      | CSP指令值 | 描述 |
      | :---: | :---: |
      | * | 允许任何内容 |
      | 'none' | 不允许加载任何内容 |
      | 'self' | 允许来自相同源的资源 |
      | data | 允许data:协议，如base64编码的图片 |
      | www.host.com | 允许加载指定域名的资源 |
      | unsafe-inline | 允许加载inline资源，如style属性，inline css, inline js |
      | unsafe-eval | 允许加载动态js代码 |

  * `content-type`(已过时，直接使用 `<meta charset="utf-8">`)

      如果使用这个属性，其值必须是"text/html; charset=utf-8"。注意：该属性只能用于 MIME type 为 text/html 的文档，不能用于MIME类型为XML的文档。

  * `default-style`

      设置默认 CSS 样式表组的名称。

  * `X-UA-Compatible`

      用于告知浏览器以何种版本来渲染页面。`<meta http-equiv="X-UA-Compatible" content="IE=edge,chrome=1"/>`指定IE和Chrome使用最新版本渲染当前页面

  * `refresh`

      这个属性指定:

      * 如果 content 只包含一个正整数，则为重新载入页面的时间间隔(秒)；
      * 如果 content 包含一个正整数，并且后面跟着字符串 ';url=' 和一个合法的 URL，则是重定向到指定链接的时间间隔(秒)

      `＜meta http-equiv="Refresh" content="2；URL=http://www.net.cn/"＞`其中的2是指停留2秒钟后自动刷新到URL网址。

  * `Expires`

      可以用于设定网页的到期时间。一旦网页过期，必须到服务器上重新传输。`＜meta http-equiv="expires" content="Wed, 20 Jun 2007 22:33:00 GMT"＞` 必须使用GMT的时间格式

  * `Pragma`

      禁止浏览器从本地计算机的缓存中访问页面内容，这样设定，访问者将无法脱机浏览

  * `Set-Cookie`（已过时, 使用HTTP头的Set-Cookie替代）

      如果网页过期，那么存盘的cookie将被删除。`＜meta http-equiv="Set-Cookie" content="cookievalue=xxx;expires=Wednesday, 20-Jun-2007 22:33:00 GMT； path=/"＞` 必须使用GMT的时间格式。

  * `Window-target`

      强制页面在当前窗口以独立页面显示。`＜meta http-equiv="Window-target" content="_top"＞` 用来防止别人在框架里调用自己的页面。

      注: `_blank` 在新窗口显示 `_top` 当前整个窗口显示 `_parent` 父容器显示，比如框架嵌套 `_self` 当前容器显示，比如框架嵌套

  * `content-Type`

      设定页面使用的字符集。`＜meta http-equiv="content-Type" content="text/html; charset=gb2312"＞`

  * `Page_Enter`、`Page_Exit`

      设定进入(退出)页面时的特殊效果 `<meta http-equiv="Page-Enter" contect="revealTrans(duration=1.0,transtion=12)">`、`<meta http-equiv="Page-Exit" contect="revealTrans(duration=1.0,transtion=12)">`

      duration的值为网页动态过渡的时间，单位为秒。transition是过渡方式，它的值为0到23，分别对应24种过渡方式。如下：

      | 数值 | 效果 | 数值 | 效果 | 数值 | 效果 | 数值 | 效果 |
      | --- | :---: | :---: | :---: | :---: | :---: | :---: | :---: |
      | 0 | 盒状收缩 | 1 | 盒状放射 | 2 | 圆形收缩 | 3 | 圆形放射 |
      | 4 | 由下往上 | 5 | 由上往下 | 6 | 从左至右 | 7 | 从右至左 | 
      | 8 | 垂直百叶窗 | 9 | 水平百叶窗 | 10 | 水平格状百叶窗 | 11 | 垂直格状百叶窗 |
      | 12 | 随意溶解 | 13 | 从左右两端向中间展开 | 14 | 从中间向左右两端展开 | 15 | 从上下两端向中间展开 |
      | 16 | 从中间向上下两端展开 | 17 | 从右上角向左下角展开 | 18 | 从右下角向左上角展开 | 19 | 从左上角向右下角展开 |
      | 20 | 从左下角向右上角展开 | 21 | 水平线状展开 | 22 | 垂直线状展开 | 23 | 随机产生一种过渡方式 |

  * `cache-control`

      被用于在http请求和响应中，通过指定指令来实现缓存机制。缓存指令是单向的，这意味着在请求中设置的指令，不一定被包含在响应中。

      **可缓存性**：

      * `public` 表明响应可以被任何对象（包括：发送请求的客户端，代理服务器，等等）缓存，即使是通常不可缓存的内容。（例如：1.该响应没有max-age指令或Expires消息头；2. 该响应对应的请求方法是 POST 。）
      * `private` 表明响应只能被单个用户缓存，不能作为共享缓存（即代理服务器不能缓存它）。私有缓存可以缓存响应内容，比如：对应用户的本地浏览器。
      * `no-cache` 在发布缓存副本之前，强制要求缓存把请求提交给原始服务器进行验证(协商缓存验证)。
      * `no-store` 缓存不应存储有关客户端请求或服务器响应的任何内容，即不使用任何缓存。

      **到期**
      * `max-age=<seconds>` 设置缓存存储的最大周期，超过这个时间缓存被认为过期(单位秒)。与Expires相反，时间是相对于请求的时间。

  * `x-dns-prefetch-control`
      是否允许DNS预解析  `on` `off`
