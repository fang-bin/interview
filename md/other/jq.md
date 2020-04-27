#### jq怎样实现的链式操作，还有是怎么扩展jq插件的
**链式操作实现原理**：

链式操作仅仅是通过对象上的方法最后 return this 把对象再返回回来，对象当然可以继续调用方法啦，所以就可以链式操作了。

**为什么要链式操作**

* 节省代码量，代码看起来更优雅

* 缓存元素，省去了逐步查询DOM元素的性能损耗。

* 为了更好的异步体验，在链条越后位置的方法就越后执行。
  Javascript是非阻塞语言（非阻塞就是一刻不停得做事，一直不停下来，当遇到需要等待的时候，直接跳过，等这边完了，再来执行），所以他不是没阻塞，而是不能阻塞（这是因为js是单线程的），所以他需要通过事件来驱动，异步来完成一些本需要阻塞进程的操作。

**jq是怎么扩展插件的**

为了方便用户创建插件，jquery提供了jQuery.extend()和jQuery.fn.extend()方法。

其中jQuery.extend是扩展工具方法，将方法放在jQuery对象上的，jQuery.fn.extend是扩展实例方法

**jq中的和和.fn（或者说jQuery.extend()和jQuery.fn.extend()）的区别是什么？**

jQuery.extend()扩展jQuery对象本身，例如jQuery内置的 ajax方法都是用jQuery.ajax()这样调用。它是用一个或多个其他对象来扩展一个对象，返回被扩展的对象。如果不指定target，则给jQuery命名空间本身进行扩展。

jQuery.fn.extend(object)扩展 jQuery 元素集来提供新的方法（通常用来制作插件）。

```javascript
jQuery.fn = jQuery.prototype = {
　init: function( selector, context ) {.....};
};
```

jQuery.fn = jQuery.prototype，也就是jQuery对象的原型。那jQuery.fn.extend()方法就是扩展jQuery对象的原型方法。


### jq为什么渐渐没落被取代

jQuery出现的的背景是原生dom操作极为复杂且难以维护，各个浏览器兼容性问题良多。

jQuery的出现解决了以下问题：

* 它解决了dom api兼容的问题，使得dom操作更简便
* 它支持类似css选择器的方式来选择组件
* 支持批量的操作数组中的元素，也叫隐式迭代
* 支持链式操作，可以在一条语句中完成很复杂的逻辑
* 有易于使用的插件扩展机制
* deffered的异步方案比promise更早。

jQuery之所以没落，是因为mvvm的出现

dom操作是业务无关的逻辑，不应该出现在业务的代码中，虽然使用jQuery简化了很多，但是代码依然是难以维护和复用的，直到mvvm的出现，把数据和视图的绑定变成了自动化的操作，进而把dom操作从业务代码中移除。业务代码因此变得更加的纯粹，也更容易复用。

技术的发展趋势就是追求更高的复用性，更简便的业务代码写法，所以最终都会要求跨平台、都会彻底分离非业务逻辑。

业务代码应该是纯粹的，任何业务代码都应该独立出去作为可复用资源而存在。比如dom操作的代码很多时候是业务无关的，所以mvvm实现了自动的绑定之后，逐渐的成为主流，jquery不符合这个趋势，所以也逐渐走向没落。

jQuery在dom操作领域已经做得很好了，但是它不符合技术发展的一般规律。

jQuery战胜了dom操作领域的所有对手，只是输给了时代。

#### jQuery实现

```javascript
;
(function (ROOT) {

  // 构造函数
  var jQuery = function (selector) {

    // 在jQuery中直接返回new过的实例，这里的init是jQuery的真正构造函数
    return new jQuery.fn.init(selector)
  }

  jQuery.fn = jQuery.prototype = {
    constructor: jQuery,

    version: '1.0.0',

    init: function (selector) {
      // 在jquery中这里有一个复杂的判断，但是这里我做了简化
      var elem, selector;
      elem = document.querySelector(selector);
      this[0] = elem;

      // 在jquery中返回一个由所有原型属性方法组成的数组，我们这里简化，直接返回this即可
      // return jQuery.makeArray(selector, this);
      return this;
    },

    // 在原型上添加一堆方法
    toArray: function () { },
    get: function () { },
    each: function () { },
    ready: function () { },
    first: function () { },
    slice: function () { }
    // ... ...
  }

  jQuery.fn.init.prototype = jQuery.fn;

  // 实现jQuery的两种扩展方式
  jQuery.extend = jQuery.fn.extend = function (options) {

    // 在jquery源码中会根据参数不同进行很多判断，我们这里就直接走一种方式，所以就不用判断了
    var target = this;
    var copy;

    for (name in options) {
      copy = options[name];
      target[name] = copy;
    }
    return target;
  }

  // jQuery中利用上面实现的扩展机制，添加了许多方法，其中

  // 直接添加在构造函数上，被称为工具方法
  jQuery.extend({
    isFunction: function () { },
    type: function () { },
    parseHTML: function () { },
    parseJSON: function () { },
    ajax: function () { }
    // ...
  })

  // 添加到原型上
  jQuery.fn.extend({
    queue: function () { },
    promise: function () { },
    attr: function () { },
    prop: function () { },
    addClass: function () { },
    removeClass: function () { },
    val: function () { },
    css: function () { }
    // ...
  })

  // $符号的由来，实际上它就是jQuery，一个简化的写法，在这里我们还可以替换成其他可用字符
  ROOT.jQuery = ROOT.$ = jQuery;
})(window);
```

在上面的实现中，首先在jQuery构造函数里声明了一个fn属性，并将其指向了原型jQuery.prototype。然后在原型中添加了init方法。

```javascript
jQuery.fn = jQuery.prototype = {
  init: {}
}
```

之后又将init的原型，指向了jQuery.prototype。

```javascript
jQuery.fn.init.prototype = jQuery.fn;
```

而在构造函数jQuery中，返回了init的实例对象。

```javascript
var jQuery = function (selector) {

  // 在jQuery中直接返回new过的实例，这里的init是jQuery的真正构造函数
  return new jQuery.fn.init(selector)
}
```

最后对外暴露入口时，将字符$与jQuery对等起来。

```javascript
ROOT.jQuery = ROOT.$ = jQuery;
```

因此当我们直接使用$('#test')创建一个对象时，实际上是创建了一个init的实例，这里的真正构造函数是原型中的init方法。

注意:许多对jQuery内部实现不太了解的盆友，常常会毫无节制使用$(),每当我们执行$()时，就会重新生成一个init的实例对象，因此当我们这样没有节制的使用jQuery是非常不正确的，虽然看上去方便了一些，但是对于内存的消耗非常大。正确的做法是既然是同一个对象，那么就用一个变量保存起来后续使用即可。
