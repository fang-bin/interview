1. 手写jsonp
2. Promise 带中断实现
3. curry
  封装一个curry函数，在实现一个以下效果的函数
    ```javascript
    add(1)(2)(3);   //6
    add(1,2,3)(4);  // 10
    add(1)(2)(3)(4)(5)
    ```

4. call apply bind new实现
5. 防抖 节流
6. promisify实现
7. co模块简易实现
8. 计算字符串中出现相同字母的最大数量和最大的字母
9. forEach中止
10. 深拷贝
11. 利用Generator函数实现斐波那契数列
12. 利用Generator函数遍历完全二叉树


答案:
##### 1 手写jsonp
```javascript
(function(window, document) {
  const jsonp = function (url, options, callback){
    let jsonpStr = url.indexOf('?') > -1 ? '&' : '?';
    for (const key in options) {
      if (options.hasOwnProperty(key)) {
        jsonpStr += `${key}=${options[key]}&`;
      }
    }
    const callbackName = Math.random().toString(16).replace('.', '');
    jsonpStr += `callback=${callbackName}`;
    const scriptDom = document.createElement('script');
    scriptDom.src = `${url}${jsonpStr}`;
    window[callbackName] = function (data){
      callback(data);
      document.removeChild(scriptDom);
    }
    document.appendChild(scriptDom);
  }
  window.$jsonp = jsonp;
})(window, document);
```

##### 2 Promise带中断实现
```javascript
function abortPromise (promise){
  let _abort = undefined;
  let _abort_promise = new Promise((resolve, reject) => {
    _abort = reject;
  });
  let p = Promise.race([promise, _abort_promise]);
  p.abort = _abort;
  return p;
}
```

##### 3 curry
```javascript
function curry (fn, args = []){
  const len = fn.length;
  return function (..._args) {
    args.push(..._args);
    if (args.length < len) {
      return curry.call(this, fn, args);
    }
    return fn.apply(this, args);
  }
}
```

```javascript
// add(1)(2)(3);   //6
// add(1,2,3)(4);  // 10
// add(1)(2)(3)(4)(5)

function add (...args){
  function _adder (..._args){
    args.push(..._args);
    return _adder;
  }
  _adder.toString = function (){
    return args.reduce((a, b) => {
      return a + b;
    }, 0);
  }
  return _adder
}
```

##### 4 call apply bind new

```javascript
Function.prototype.myCall = function (thisArg, ...args) {
  const fn = Symbol('fn');
  thisArg = thisArg || window;
  thisArg[fn] = this;
  const result = thisArg[fn](...args);
  delete thisArg[fn];
  return result;
}

Function.prototype.myApply = function (thisArg, args) {
  const fn = Symbol('fn');
  thisArg = thisArg || window;
  thisArg[fn] = this;
  const result = thisArg[fn](...args);
  delete thisArg[fn];
  return result;
}

Function.prototype.myBind = function (thisArg, ...args) {
  var self = this
  // new优先级
  var funcBind = function () {
    self.apply(this instanceof self ? this : thisArg, args.concat(Array.prototype.slice.call(arguments)))
  }
  // 继承原型上的属性和方法
  funcBind.prototype = Object.create(self.prototype);
  return funcBind;
}


function mockNew() {
  let Constructor = Array.prototype.shift.call(arguments); // 取出构造函数  这个地方之后，arguments就已经去除了obj
  
  let obj = {}   // new 执行会创建一个新对象
  
  obj.__proto__ = Constructor.prototype 
  
  Constructor.apply(obj, arguments)
  return obj
}
```

##### 5. 防抖、节流

```javascript
// 防抖
function debounce (fn, time){
  let _timer = undefined;
  return function (...args) {
    const context = this;
    if (_timer){
      clearTimeout(_timer);
      _timer = null;
    }
    _timer = setTimeout(() => {
      fn.apply(context, args);
    }, time);
  }
}

// 节流
function throttle (fn, time){
  let _last_time = undefined;
  return function (...args){
    const _now_time = new Date();
    if (!_last_time || _now_time - _last_time > time){
      fn(...args);
      _last_time = _now_time;
    }
  }
}
```

##### 6. promisify实现

```javascript
// const newFn = promisify(fn)
// newFn(a) 会执行Promise参数方法
function promisify(fn) {
  return function(...args) {
    // 返回promise的实例
    return new Promise(function(reslove, reject) {
      // newFn(a) 时会执行到这里向下执行
      // 加入参数cb => newFn(a)
      args.push(function(err, data) {
        if (err) {
          reject(err)
        } else {
          reslove(data)
        }
      })
      // 这里才是函数真正执行的地方执行newFn(a, cb)
      fn.apply(null, args)
    })
  }
}
```

##### 7. co模块简易实现

```javascript

```

##### 8. 计算字符串中出现相同字母的最大数量和最大的字母

```javascript
```

##### 9. forEach中止

```javascript
```

##### 10. 深拷贝

```javascript
```

##### 11. 利用Generator函数实现斐波那契数列

```javascript
function* fibonacci() {
  let [prev, curr] = [0, 1];
  for (;;) {
    yield prev;
    [prev, curr] = [curr, prev + curr];
  }
}
```

##### 12. 利用Generator函数遍历完全二叉树

```javascript
```

## 快记

##### 1. 形成BFC条件  BFC特点
条件:
* 根元素
* 浮动（float的值不为none）；
* 绝对定位元素（position的值为absolute或fixed）；
* 行内块（display为inline-block）
* 表格单元（display为table、table-cell、table-caption等HTML表格相关属性）；
* 弹性盒（display为flex或inline-flex）；
* overflow不为visible；
* 网格元素（display为 grid 或 inline-grid 元素的直接子元素） 等等。

特点:
* 处于同一个BFC中的元素相互影响，可能会发生margin collapse；
* BFC在页面上是一个独立的容器，容器里面的子元素不会影响到外面的元素，反之亦然；
* 计算BFC的高度时，考虑BFC所包含的所有元素，包括浮动元素也参与计算；
* 浮动盒的区域不会叠加到BFC上

##### 2. 形成堆叠上下文条件  堆叠上下文的顺序
* 根元素（即HTML元素）
* 已定位元素（即绝对定位或相对定位）并且z-index不是默认的auto。
* z-index值不为auto的flex项(父元素display:flex|inline-flex).
* 元素的opacity值不是1.
* 元素的transform值不是none.
* 元素mix-blend-mode值不是normal.
* 元素的filter值不是none.
* 元素的isolation值是isolate.
* will-change指定的属性值为上面任意一个。
* 元素的-webkit-overflow-scrolling设为touch.

![avator](https://camo.githubusercontent.com/b6d00d5b77984171393368b4b6988ed9b946abc6/68747470733a2f2f696d6167652e7a68616e6778696e78752e636f6d2f696d6167652f626c6f672f3230313630312f323031362d30312d30395f3231313131362e706e67)

不依赖z-index的层叠上下文，类似于transform不为none,地位与postion: absoulte; z-index:auto;)相当。

##### 3. 伪类选择器顺序
`a:link a:visited a:hover a:active`

##### 4. CSS优化

* 多个css可合并，并尽量减少http请求
* 属性值为0时，不加单位
* 将css文件放在页面最上面
* 避免后代选择符，过度约束和链式选择符
* 使用紧凑的语法
* 避免不必要的重复
* 使用语义化命名，便于维护
* 尽量少的使用!impotrant，可以选择其他选择器
* 精简规则，尽可能合并不同类的重复规则
* 遵守盒子模型规则

##### 5. link 与 @import 的区别
* link 是HTML方式， @import 是CSS方式；
* link最大限度支持并行下载，@import 过多嵌套导致串行下载，出现FOUC；
* link 可以通过 rel="alternate stylesheet" 指定候选样式；
* 浏览器对 link 支持早于@import ，可以使用 @import对老浏览器隐藏样式；
* @import必须在样式规则之前，可以在css文件中引用其他文件；

总的来说： link优于@import。

##### 6. 一个inline-block元素，如果里面没有inline内联元素，或者overflow不是visible，则该元素的基线就是其margin底边缘，否则，其基线就是元素里面最后一行内联元素的基线。

<style>
.box{
  margin: 16px 0;
  word-break: break-all;
  font-stretch: expanded;
}
.dib-baseline{
  display: inline-block;
  width: 100px;
  height: 100px;
  border: 1px solid #cad5eb;
  background-color: #f0f3f9;
}
</style>
<p class="box"><span class="dib-baseline"></span> <span class="dib-baseline" onclick="this.innerHTML=this.innerHTML? '': 'x-baseline'">x-baseline</span></p>

##### 7. 针对一些字体在安卓上渲染靠上的问题

这个问题通过css是无法解决的，即使解决了也是一种通过微调来实现的hack方法，因为文字在content-area内部渲染的时候已经偏移了，而css的居中方案都是控制的整个content-area的居中。

导致这个问题的本质原因可能是Android在排版计算的时候参考了primyfont字体的相关属性（即HHead Ascent、HHead Descent等），而primyfont的查找是看`font-family`里哪个字体在fonts.xml里第一个匹配上，而原生Android下中文字体是没有family name的，导致匹配上的始终不是中文字体，所以解决这个问题就要在`font-family`里显式申明中文，或者通过什么方法保证所有字符都fallback到中文字体。

针对Android 7.0+设备: \<html>上设置 lang 属性：\<html lang="zh-cmn-Hans">，同时font-family不指定英文，如 font-family: sans-serif 。这个方法是利用了浏览器的字体fallback机制，让英文也使用中文字体来展示，blink早期的内核在fallback机制上存在问题，Android 7.0+才能ok，早期的内核下会导致英文fallback到Noto Sans Myanmar，这个字体非常丑。

针对MIUI 8.0+设备：设置 font-family: miui;

##### 8. viewport
`<meta name="viewport" content="width=device-width, initial-scale=1, maximum-scale=1, minimum-scale=1, user-scale=no">`

##### 9. 语义化

语义化是指根据内容的结构化（内容语义化），选择合适的标签（代码语义化），便于开发者阅读和写出更优雅的代码的同时，让浏览器的爬虫和机器很好的解析。

语义化的好处

* 用正确的标签做正确的事情；
* 去掉或者丢失样式的时候能够让页面呈现出清晰的结构；
* 方便其他设备解析（如屏幕阅读器、盲人阅读器、移动设备）以意义的方式来渲染网页；
* 有利于SEO：和搜索引擎建立良好沟通，有助于爬虫抓取更多的有效信息：爬虫依赖于标签来确定上下文和各个关键字的权重；
* 便于团队开发和维护，语义化更具可读性，遵循W3C标准的团队都遵循这个标准，可以减少差异化。

##### 10. 浏览器的怪异模式（Quirks）和严格模式（Standards）的区别

* 盒模型 在W3C标准中，如果设置一个元素的宽度和高度，指的是元素内容的宽度和高度，而在Quirks 模式下，IE的宽度和高度还包含了padding和border；
* 设置行内元素的高宽 在Standards模式下，给等行内元素设置wdith和height都不会生效，而在quirks模式下，则会生效；
* 行内元素的垂直对齐 在Standards模式下，其是基线对其，而在quirks模式下，则是底线对齐
* 设置百分比的高度 在standards模式下，一个元素的高度是由其包含的内容来决定的，如果父元素没有设置百分比的高度，子元素设置一个百分比的高度是无效的用；
* 设置水平居中 使用margin:0 auto在standards模式下可以使元素水平居中，但在quirks模式下却会失效。

##### 11. Promise的实现就是一个发布订阅模式，这种收集依赖 -> 触发通知 -> 取出依赖执行的方式，被广泛运用于发布订阅模式的实现。

##### 12. forEach map filter这些js内置的函数，第二个参数都是可以绑定this的

##### 13 序列化与反序列化

* 序列化：把变量从内存中变成可存储或传输的过程称之为序列化 
* 反序列化：把变量内容从序列化的对象重新读到内存里称之为反序列化

**序列化过程中，不安全的值(undefined, Symbol, function, Map, Set, RegExp)不能识别，undefined、Symbol、function会变成null，Map、Set、RegExp则会变成{}**
