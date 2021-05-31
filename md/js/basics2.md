### 1. queueMicrotask(function)

功能上更底层（纯粹的队列调度）,以微任务的形式来调度回调。

从微任务本身的概念来说的话，就是当我们期望某段代码，不阻塞当前执行的同步代码，同时又期望它尽可能快地执行时，我们就需要它。

它和node中的process.nextTick类似。

兼容性很一般。

还有其他的一些类似微任务源:

##### MutationObserver

Mutation observer 是用于代替 Mutation events 作为观察DOM树结构发生变化时，做出相应处理的API。

MutationObserver接口提供了监视对DOM树所做更改的能力。创建并返回一个新的 MutationObserver 它会在指定的DOM发生变化时被调用。

和 Mutation events不同，MutationObserver 所有监听操作以及相应处理都是在其他脚本执行完成之后异步执行的，并且是所以变动触发之后，将变得记录在数组中，统一进行回调的，也就是说，当你使用observer监听多个DOM变化时，并且这若干个DOM发生了变化，那么observer会将变化记录到变化数组中，等待一起都结束了，然后一次性的从变化数组中执行其对应的回调函数。

方法：

* `disconnect()`  阻止 MutationObserver 实例继续接收的通知，直到再次调用其observe()方法，该观察者对象包含的回调函数都不会再被调用。
* `observe(target[, options])` 配置MutationObserver在DOM更改匹配给定选项时，通过其回调函数开始接收通知。
* `takeRecords()` 从MutationObserver的通知队列中删除所有待处理的通知，并将它们返回到MutationRecord对象的新Array中。

兼容性还可以

例：

```javascript
// Firefox和Chrome早期版本中带有前缀
var MutationObserver = window.MutationObserver || window.WebKitMutationObserver || window.MozMutationObserver
// 选择目标节点
var target = document.querySelector('#some-id'); 
// 创建观察者对象
var observer = new MutationObserver(function(mutations) {  
  mutations.forEach(function(mutation) { 
    console.log(mutation.type); 
  }); 
}); 
// 配置观察选项:
var config = { attributes: true, childList: true, characterData: true } 
// 传入目标节点和观察选项
observer.observe(target, config); 
// 随后,你还可以停止观察
observer.disconnect();
```

##### requestAnimationFrame(callback);

要求浏览器在下次重绘之前调用指定的回调函数更新动画。该方法需要传入一个回调函数作为参数，该回调函数会在浏览器下一次重绘之前执行。

若想在浏览器下次重绘之前继续更新下一帧动画，那么回调函数自身必须再次调用window.requestAnimationFrame()。

回调函数执行次数通常是每秒60次，但在大多数遵循W3C建议的浏览器中，回调函数执行次数通常与浏览器屏幕刷新次数相匹配。

callback中会传入一个DOMHighResTimeStamp参数（该参数与performance.now()的返回值相同），指示当前被 requestAnimationFrame() 排序的回调函数被触发的时间。在同一个帧中的多个回调函数，它们每一个都会接受到一个相同的时间戳，即使在计算上一个回调函数的工作负载期间已经消耗了一些时间。该时间戳是一个十进制数，单位毫秒，最小精度为1ms(1000μs)。

### 2. JavaScript 数据不变性 (`Object.frezze` `Object.seal`)

相同点:
* 作用的对象变得不可扩展，这意味着不能再添加新属性。
* 作用的对象中的每个元素都变得不可配置，这意味着不能删除属性。（configurable都为false）
* 如果在 ‘use strict’ 模式下使用，这两个方法都可能抛出错误，例如在严格模式下修改会报错。

不同点:
* `Object.freeze` 也会将 writable 改为false，所以`Object.seal`密封的对象能修改值，而`Object.freeze`冻结的对象不能修改值。

限制:
* `Object.freeze` 和 `Object.seal` 在 “实用性” 方面都有 “缺陷”，他们只冻结/封印对象的第一层深度。

验证:
* `Object.isFrozen(target)` 验证是否为冻结对象
* `Object.isSealed(target)` 验证是否为密封对象或者冻结对象（因为冻结对象不仅writable为false, configurable也为false）

深冻结函数:

```javascript
// 这里面没有考虑循环引用的情况
function deepFreeze (target){
  const propNames = Object.getOwnPropertyNames(target);
  propNames.forEach(name => {
    const prop = target[name];
    if (typeof prop === 'object' && prop !== null) {
      deepFreeze(prop);
    }
  });
  return Object.freeze(target);
}
```

冻结对象解冻:

* js中冻结的对象并没有解冻的操作（毕竟`configurable: false`本身就已经绝了可以操作的可能性），一般情况下，我们可以通过克隆一个相同属性的新对象来达到目的。

**补充: `Object.preventExtensions()`**

阻止对象扩展，让一个对象变的不可扩展，也就是永远不能再添加新的属性。（并不会修改第一层对象的 `configurable` 和 `writable` ）

可以使用`Object.isExtensible()`来判断一个对象是否可以扩展。

### 3. 跳转一个已经打开的页面

```javascript
window.open(url[, target]);
```

* 第一个参数（url）: 新页面的地址；
* 第二个参数（target）: 页面的名称。如果当前打开的页面中没有该名称的页面。则打开新页面，并给该页面名称标注为target。否则跳转到该页面。

**补充**页面的名称即是页面的`window.name`属性。

### 4. IntersectionObserver

IntersectionObserver接口提供了一种异步观察目标元素与其祖先元素或顶级文档视窗(viewport)交叉状态的方法。祖先元素与视窗(viewport)被称为根(root)。

```javascript
var observer = new IntersectionObserver(callback[, options]);
```

###### callback

当元素可见比例超过指定阈值后，会调用一个回调函数，此回调函数接受两个参数：

* entries
  一个IntersectionObserverEntry对象的数组，每个被触发的阈值，都或多或少与指定阈值有偏差。
  IntersectionObserverEntry对象有很多属性，常用的：
  * target: 被观察的目标元素，是一个 DOM 节点对象
  * isIntersecting: 是否进入可视区域
  * intersectionRatio: 相交区域和目标元素的比例值，进入可视区域，值大于0，否则等于0
* observer
  被调用的IntersectionObserver实例。

###### options
一个可以用来配置observer实例的对象。如果options未指定，observer实例默认使用文档视口作为root，并且没有margin，阈值为0%（意味着即使一像素的改变都会触发回调函数）。有以下可配置项

* root
  监听元素的祖先元素Element对象，其边界盒将被视作视口。目标在根的可见区域的的任何不可见部分都会被视为不可见。
* rootMargin
  一个在计算交叉值时添加至根的边界盒(bounding_box (en-US))中的一组偏移量，类型为字符串(string) ，可以有效的缩小或扩大根的判定范围从而满足计算需要。语法大致和CSS 中的margin 属性等同; 
* threshold
  规定了一个监听目标与边界盒交叉区域的比例值，可以是一个具体的数值或是一组0.0到1.0之间的数组。若指定值为0.0，则意味着监听元素即使与根有1像素交叉，此元素也会被视为可见. 若指定值为1.0，则意味着整个元素都是可见的

###### 返回值

一个可以使用规定阈值监听目标元素可见部分与root交叉状况的新的IntersectionObserver 实例。调用自身的observe() 方法开始使用规定的阈值监听指定目标。

##### 属性（都是只读）

* IntersectionObserver.root
* IntersectionObserver.rootMargin
* IntersectionObserver.threshold

##### 方法

* `IntersectionObserver.disconnect()` 使IntersectionObserver对象停止监听工作

* `IntersectionObserver.observe()` 使IntersectionObserver开始监听一个目标元素。

* `IntersectionObserver.unobserve()` 使IntersectionObserver停止监听特定目标元素。

* `IntersectionObserver.takeRecords()` 返回所有观察目标的IntersectionObserverEntry对象数组。

例子:

```javascript
const box = document.querySelector('.box');
const imgs = document.querySelectorAll('.img');

const observer = new IntersectionObserver(entries => {
    // 发生交叉目标元素集合
    entries.forEach(item => {
        // 判断是否发生交叉
        if (item.isIntersecting) {
            // 替换目标元素Url
            item.target.src = item.target.dataset.src;
            // 取消监听此目标元素
            observer.unobserve(item.target);
        }
    });
}, {
    root: box, // 父级元素
    rootMargin: '20px 0px 100px 0px' // 设置偏移 我们可以设置在目标元素距离底部100px的时候发送请求
});

imgs.forEach(item => {
    // 监听目标元素
    observer.observe(item);
});
```


**IntersectionObserver API是异步的，不随着目标元素的滚动同步触发，性能消耗极低。**

**IntersectionObserver的实现，应该采用requestIdleCallback()，即只有线程空闲下来，才会执行观察器。这意味着，这个观察器的优先级非常低，只在其他任务执行完，浏览器有了空闲才会执行。**

### 5. Chrome 89 更新事件触发顺序

在我之前了解到的事件触发按照触发阶段大致可以划分为 **捕获阶段->目标阶段->冒泡阶段**

而在**冒泡阶段**，以addEventListener方式添加事件来说，先添加的先执行，后添加的后执行。

不过在Chrome 89及之后版本更改为**目标元素的触发事件顺序不再按照注册顺序触发！而是按照先捕获再冒泡的形式依次执行！**

之所以这么做，是因为在 webkit 中原先的事件模型，会导致含有 Shadow DOM 的情况下，子元素的捕获事件会优先于父元素的捕获事件触发。

为了兼容这些变化，在项目开发过程中，所有目标元素代码的顺序都应按照先写捕获事件代码，再写冒泡事件代码。

### 6. Shadow DOM


