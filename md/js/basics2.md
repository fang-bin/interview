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


