## 前端路由

#### 后端路由
路由这个概念最先是后端出现的。在以前用模板引擎开发页面时，经常会看到这样

```html
http://www.xxx.com/login
```

大致流程可以看成这样：

1. 浏览器发出请求
2. 服务器监听到80端口（或443）有请求过来，并解析url路径
3. 根据服务器的路由配置，返回相应信息（可以是 html 字串，也可以是 json 数据，图片等）
4. 浏览器根据数据包的 Content-Type 来决定如何解析数据

简单来说路由就是用来跟后端服务器进行交互的一种方式，通过不同的路径，来请求不同的资源，请求不同的页面是路由的其中一种功能。

#### 前端路由

浏览器中的限制
* 没有提供监听前进后退的事件。
* 不允许开发者读取浏览纪录，也就是 js 读取不了浏览纪录。
* 用户可以手动输入地址，或使用浏览器提供的前进后退来改变 url。

**hash模式**
随着 ajax 的流行，异步数据请求交互运行在不刷新浏览器的情况下进行。而异步交互体验的更高级版本就是 SPA —— 单页应用。单页应用不仅仅是在页面交互是无刷新的，连页面跳转都是无刷新的，为了实现单页应用，所以就有了前端路由。

类似于服务端路由，前端路由实现起来其实也很简单，就是匹配不同的 url 路径，进行解析，然后动态的渲染出区域 html 内容。但是这样存在一个问题，就是 url 每次变化的时候，都会造成页面的刷新。那解决问题的思路便是在改变 url 的情况下，保证页面的不刷新。在 2014 年之前，大家是通过 hash 来实现路由，url hash 就是类似于：

```html
http://www.xxx.com/#/login
```

这种 #。后面 hash 值的变化，并不会导致浏览器向服务器发出请求，浏览器不发出请求，也就不会刷新页面。另外每次 hash 值的变化，还会触发 hashchange 这个事件，通过这个事件我们就可以知道 hash 值发生了哪些变化。然后我们便可以监听hashchange来实现更新页面部分内容的操作：

```javascript
function matchAndUpdate () {
   // todo 匹配 hash 做 dom 更新操作
}
window.addEventListener('hashchange', matchAndUpdate)
```

**history模式**
14年后，因为HTML5标准发布。多了两个 API，pushState 和 replaceState，通过这两个 API 可以改变 url 地址且不会发送请求。同时还有 popstate 事件。通过这些就能用另一种方式来实现前端路由了，但原理都是跟 hash 实现相同的。用了 HTML5 的实现，单页路由的 url 就不会多出一个#，变得更加美观。但因为没有 # 号，所以当用户刷新页面之类的操作时，浏览器还是会给服务器发送请求。为了避免出现这种情况，所以这个实现需要服务器的支持，需要把所有路由都重定向到根页面。

```javascript
function matchAndUpdate () {
   // todo 匹配路径 做 dom 更新操作
}
window.addEventListener('popstate', matchAndUpdate);
```

#### pushState 和 replaceState 介绍

`pushState(state, title, url)`

`replaceState(state, title, url)`

浏览器不会向服务端请求数据，直接改变url地址，可以类似的理解为变相版的hash；但不像hash一样，浏览器会记录pushState的历史记录，可以使用浏览器的前进、后退功能作用，不同于pushState，replaceState仅仅是修改了对应的历史记录。

**注意** `pushState` 和 `replaceState` 更改的url与原有页面跨源的情况下，会报错。这样设计的目的是，防止恶意代码让用户以为他们是在另一个网站上。

当用户在浏览器点击进行后退、前进，或者在js中调用histroy.back()，history.go()，history.forward()等，会触发popstate事件；但pushState、replaceState不会触发这个事件。(可以通过直接更改原生history上的pushState、replaceState方法来监听并触发相应函数。)

#### 实现一个前端路由