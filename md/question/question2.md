### 1. `content-visibility: hidden;` `display:none;` `visibility: hidden`的区别
* `display: none`：隐藏元素并破坏其渲染状态。这意味着取消隐藏元素与渲染具有相同内容的新元素一样昂贵。渲染树中不包括 `display: none` 的元素。

* `visibility: hidden`：隐藏元素并保持其渲染状态。这并不能真正从文档中删除该元素，因为它（及其子树）仍占据页面上的几何空间，并且仍然可以单击。它也可以在需要时随时更新渲染状态，即使隐藏也是如此。其元素仍然存在于渲染树中，该属性为继承属性。

* `content-visibility: hidden`：隐藏元素并保留其渲染状态。这意味着该元素隐藏时行为和`display: none`一样，但再次显示它的成本要低得多。content-visibility可跳过不在屏幕上的内容渲染，包括布局和渲染，直到真正需要布局渲染的时候为止。不过由于其没有其他设置高度为0，可能会造成滚动问题。

### 2. 为什么要对URL进行编码，以及 escape、encodeURI、encodeURIComponent 的区别

URL编码采用ASCII码，为了造成避免歧义（例如&、=），所以需要对URL进行编码。

escape，encodeURI，encodeURIComponent 都是用于将不安全不合法的Url字符转换为合法的Url字符表示。对应的解码方法为 unescape、decodeURI、decodeURIComponent

不同点

  * 安全字符不同
    * escape（69个）：*/@+-._0-9a-zA-Z
    * encodeURI（82个）：!#><'()*+,/:;=?@-._~0-9a-zA-Z
    * encodeURIComponent（71个）：!'()*-._~0-9a-zA-Z
  * 兼容性不同
    escape函数是从Javascript 1.0的时候就存在了，其他两个函数是在Javascript 1.5才引入的。但是由于Javascript 1.5已经非常普及了，所以实际上使用encodeURI和encodeURIComponent并不会有什么兼容性问题。
  * 对Unicode字符的编码方式不同
    这三个函数对于ASCII字符的编码方式相同，均是使用百分号+两位十六进制字符来表示。但是对于Unicode字符，escape的编码方式是%uxxxx，其中的xxxx是用来表示unicode字符的4位十六进制字符。
    这种方式已经被W3C废弃了。但是在ECMA-262标准中仍然保留着escape的这种编码语法。encodeURI和encodeURIComponent则使用UTF-8对非ASCII字符进行编码，然后再进行百分号编码。这是RFC推荐的。因此建议尽可能的使用这两个函数替代escape进行编码。
  * 适用场合不同
    encodeURI被用作对一个完整的URI进行编码，而encodeURIComponent被用作对URI的一个组件进行编码。从上面提到的安全字符范围表格来看，我们会发现，encodeURIComponent编码的字符范围要比encodeURI的大。

URI的保留字符

Url可以划分成若干个组件，协议、主机、路径等。有一些字符（:/?#[]@）是用作分隔不同组件的。例如：冒号用于分隔协议和主机，/用于分隔主机和路径，?用于分隔路径和查询参数，等等。

**注意**： 很多HTTP监视工具或者浏览器地址栏等在显示Url的时候会自动将Url进行一次解码（使用UTF-8字符集），这就是为什么当你在Firefox中访问Google搜索中文的时候，地址栏显示的Url包含中文的缘故。但实际上发送给服务端的原始Url还是经过编码的。

### 3. useEffect 和 useLayoutEffect 的区别，已经它们的执行顺序

useEffect：默认情况下，effect 将在每轮渲染结束后执行。

useLayoutEffect：其函数签名与 useEffect 相同，但它会在所有的 DOM 变更之后同步调用 effect。可以使用它来读取 DOM 布局并同步触发重渲染。在浏览器执行绘制之前，useLayoutEffect 内部的更新计划将被同步刷新。

useLayoutEffect、useEffect 会从最底层的子组件开始执行，一直到最外层组件。 顺序可以理解为从子树的最下层叶子节点向上冒泡，沿着render比对的顺序执行

1. 如果有子节点，优先子节点
2. 没有子节点则执行自身
3. 如果有兄弟节点，则对下一个兄弟节点执行操作 1
4. 如果没有兄弟节点，则返回父节点，执行操作 1

重复以上顺序

### 4. 什么情况下，你会考虑在项目中使用 Redux ?

* 项目中已经使用了它
* 团队对它比较熟悉
* 服务端数据 -> 可能需要被缓存
* 跨组件共享某些数据

如果是一些需要缓存的服务端数据，可能会选择 react-query, relay, apollo 等一些现代的 react 请求状态库。

### 5. 惰性计算

表达式不在它被绑定到变量之后就立即求值，而是在该值被取用的时候求值。

优点是最小化计算机要做的工作

缺点可能造成内存泄漏（计算都是懒惰的，也就是说，所有值都同时在内存中，只有在计算操作执行时，才会调用值去计算）(内存泄露 → 剩余内存不足 → 后续申请不到足够内存 →内存溢出；)

在 js 中，惰性计算主要通过 thunk 函数来实现。





