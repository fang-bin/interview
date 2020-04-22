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

**jq中的和和.fn（或者说jQuery.extend()和jQuery.fn.extend()）的区别是什么？**

jQuery.extend()扩展jQuery对象本身，例如jQuery内置的 ajax方法都是用jQuery.ajax()这样调用。它是用一个或多个其他对象来扩展一个对象，返回被扩展的对象。如果不指定target，则给jQuery命名空间本身进行扩展。

jQuery.fn.extend(object)扩展 jQuery 元素集来提供新的方法（通常用来制作插件）。

```javascript
jQuery.fn = jQuery.prototype = {
　init: function( selector, context ) {.....};
};
```

jQuery.fn = jQuery.prototype，也就是jQuery对象的原型。那jQuery.fn.extend()方法就是扩展jQuery对象的原型方法。