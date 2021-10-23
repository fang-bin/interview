## 快记

#### 1. 形成复合图层
* 最常用的方式：translate3d、translateZ、scale3d、scaleZ(仅仅是transform: translate、scale、skew并不会变成复合图层)
* 过渡动画(opacity, transform等动画，不包括absolute和margin等动画)（需要动画执行的过程中才会创建合成层，动画没有开始或结束后元素还会回到之前的状态）
* opacity和transform的动画也不会引起重绘，所以我猜测它们也会形成复合图层，作为纹理传入GPU中
* will-chang属性（这个比较偏僻），一般配合opacity与translate使用（而且经测试，除了上述可以引发硬件加速的属性外，其它属性并不会变成复合层），作用是提前告诉浏览器要变化，这样浏览器会开始做一些优化工作（这个最好用完后就释放）
* \<video>\<iframe>\<canvas>\<webgl>等元素
* 其它，譬如以前的flash插件

#### 2. 形成BFC条件  BFC特点

它是页面中的一块儿渲染区域，有一套渲染规则，决定了子元素如何定位以及与其他元素的关系和相互作用。

条件:
* 根元素
* 浮动（float的值不为none）；
* 绝对\固定定位元素（position的值为absolute或fixed）；
* 行内块（display为inline-block）
* 表格单元（display为table、table-cell、table-caption等HTML表格相关属性）；
* overflow不为visible；

以下是分别形成FFC、GFC
* 弹性盒（display为flex或inline-flex）；
* 网格元素（display为 grid 或 inline-grid 元素的直接子元素） 等等。

特点:
* 处于同一个BFC中的元素相互影响，可能会发生margin collapse；（BFC垂直方向边距重叠，FFC和GFC并不会）
* BFC在页面上是一个独立的容器，容器里面的子元素不会影响到外面的元素，反之亦然；
* 计算BFC的高度时，考虑BFC所包含的浮动元素，其也参与计算；
* 浮动盒的区域不会叠加到BFC、IFC上；

#### 3. 形成堆叠上下文条件  堆叠上下文的顺序
* 根元素（即HTML元素）
* 已定位元素（即绝对定位或相对定位）并且z-index不是默认的auto。
* z-index值不为auto的**flex子项**(父元素display:flex|inline-flex).
* 元素的opacity值不是1.
* 元素的transform值不是none.
* 元素mix-blend-mode值不是normal.
* 元素的filter值不是none.
* 元素的isolation值是isolate.
* will-change指定的属性值为上面任意一个。
* 元素的-webkit-overflow-scrolling设为touch.

![avator](https://camo.githubusercontent.com/b6d00d5b77984171393368b4b6988ed9b946abc6/68747470733a2f2f696d6167652e7a68616e6778696e78752e636f6d2f696d6167652f626c6f672f3230313630312f323031362d30312d30395f3231313131362e706e67)

不依赖z-index的层叠上下文，类似于transform不为none,地位与postion: absoulte; z-index:auto;)相当。

#### 4. 伪类选择器顺序
`a:link a:visited a:hover a:active`

#### 5. CSS优化

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

#### 6. link 与 @import 的区别
* link 是HTML方式， @import 是CSS方式；
* link最大限度支持并行下载，@import 过多嵌套导致串行下载，出现FOUC；
* link 可以通过 rel="alternate stylesheet" 指定候选样式；
* 浏览器对 link 支持早于@import ，可以使用 @import对老浏览器隐藏样式；
* @import必须在样式规则之前，可以在css文件中引用其他文件；

总的来说： link优于@import。

#### 7. 一个inline-block元素，如果里面没有inline内联元素，或者overflow不是visible，则该元素的基线就是其margin底边缘，否则，其基线就是元素里面最后一行内联元素的基线。

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

#### 8. 针对一些字体在安卓上渲染靠上的问题

这个问题通过css是无法解决的，即使解决了也是一种通过微调来实现的hack方法，因为文字在content-area内部渲染的时候已经偏移了，而css的居中方案都是控制的整个content-area的居中。

导致这个问题的本质原因可能是Android在排版计算的时候参考了primyfont字体的相关属性（即HHead Ascent、HHead Descent等），而primyfont的查找是看`font-family`里哪个字体在fonts.xml里第一个匹配上，而原生Android下中文字体是没有family name的，导致匹配上的始终不是中文字体，所以解决这个问题就要在`font-family`里显式申明中文，或者通过什么方法保证所有字符都fallback到中文字体。

针对Android 7.0+设备: \<html>上设置 lang 属性：\<html lang="zh-cmn-Hans">，同时font-family不指定英文，如 font-family: sans-serif 。这个方法是利用了浏览器的字体fallback机制，让英文也使用中文字体来展示，blink早期的内核在fallback机制上存在问题，Android 7.0+才能ok，早期的内核下会导致英文fallback到Noto Sans Myanmar，这个字体非常丑。

针对MIUI 8.0+设备：设置 font-family: miui;

#### 9. viewport
`<meta name="viewport" content="width=device-width, initial-scale=1, maximum-scale=1, minimum-scale=1, user-scale=no">`

#### 10. 语义化

语义化是指根据内容的结构化（内容语义化），选择合适的标签（代码语义化），便于开发者阅读和写出更优雅的代码的同时，让浏览器的爬虫和机器很好的解析。

语义化的好处

* 用正确的标签做正确的事情；
* 去掉或者丢失样式的时候能够让页面呈现出清晰的结构；
* 方便其他设备解析（如屏幕阅读器、盲人阅读器、移动设备）以意义的方式来渲染网页；
* 有利于SEO：和搜索引擎建立良好沟通，有助于爬虫抓取更多的有效信息：爬虫依赖于标签来确定上下文和各个关键字的权重；
* 便于团队开发和维护，语义化更具可读性，遵循W3C标准的团队都遵循这个标准，可以减少差异化。

#### 11. 浏览器的怪异模式（Quirks）和严格模式（Standards）的区别

* 盒模型 在W3C标准中，如果设置一个元素的宽度和高度，指的是元素内容的宽度和高度，而在Quirks 模式下，IE的宽度和高度还包含了padding和border；
* 设置行内元素的高宽 在Standards模式下，给等行内元素设置wdith和height都不会生效，而在quirks模式下，则会生效；
* 行内元素的垂直对齐 在Standards模式下，其是基线对其，而在quirks模式下，则是底线对齐
* 设置百分比的高度 在standards模式下，一个元素的高度是由其包含的内容来决定的，如果父元素没有设置百分比的高度，子元素设置一个百分比的高度是无效的用；
* 设置水平居中 使用margin:0 auto在standards模式下可以使元素水平居中，但在quirks模式下却会失效。

#### 12. Promise的实现就是一个发布订阅模式，这种收集依赖 -> 触发通知 -> 取出依赖执行的方式，被广泛运用于发布订阅模式的实现。

#### 13. forEach map filter这些js内置的函数，第二个参数都是可以绑定this的

#### 14 序列化与反序列化

* 序列化：把变量从内存中变成可存储或传输的过程称之为序列化 
* 反序列化：把变量内容从序列化的对象重新读到内存里称之为反序列化

**序列化过程中，不安全的值(undefined, Symbol, function, Map, Set, RegExp, BigInt)不能识别，undefined、Symbol、function会被直接过滤掉，Map、Set、RegExp则会变成{}，BigInt数据则不能进行序列化（会直接报错）**

#### 15. clearfix

```css
.clearfix {
  content: "";
  display: block;
  height: 0;
  clear: both;
  visibility: hidden;
}
```

#### 16. 超出显示省略号

```css
// 单行省略
{
  overflow: hidden;
  text-overflow: ellipsis;
  white-space: nowrap;
}

//多行省略
{
  overflow: hidden;
  text-overflow: ellipsis;
  display: -webkit-box;  //将对象作为弹性伸缩盒子模型展示）
  -webkit-line-clamp: 2;  //用来限制在一个块元素显示的文本行数
  -webkit-box-orient: vertical; //设置或检索伸缩盒对象的子元素的排列方式
}
```

#### 17. 跳出循环
* forEach 遍历数组的每一项，并对每一项执行一个 callback 函数。没有返回值(不论什么情况返回值都可以视为undefined)。forEach 方法没办法使用 break 语句跳出循环，或者使用 continue 跳过这次循环进入下次循环，但是可以通过 return 来实现跳过这次循环进入下次循环。
* for...of ES6提出的语句，在可迭代对象（Array，Map，Set，String，TypedArray，arguments）上创建一个迭代循环。（所有遍历器对象都可以遍历，即 [Symbol.iterator]=collection 都可以），不同于 forEach 可以使用 break, continue，但是不可以使用return;
* for...in for...in 语句以任意顺序遍历一个对象的可枚举属性的属性名。但是 for...in 会遍历对象本身的所有可枚举属性和从它原型继承而来的可枚举属性，因此如果想要仅迭代对象本身的属性，要结合hasOwnProperty() 来使用。 for...in遍历数组的情况下，可能会随机顺序遍历。(不可使用 continue break return)

#### 18. 有关原型内容

##### 判断

* `typeof` 在ES5中是一个安全操作，但是在ES6中，由于let/const的暂时性死区的问题，typeof也不再安全。typeof 只能大致区分类型，尤其面对复杂类型的时候，更是无法区分。
* `instanceof` 运算符返回一个布尔值，表示对象是否为某个构造函数的实例。会检查右边构造函数的原型对象（prototype），是否在左边对象的原型链上。有一种特殊情况，就是左边对象的原型链上，只有null对象。这时，instanceof判断会失真。
* `isPrototypeOf` 用于测试一个对象是否存在于另一个对象的原型链上。
* `Object.prototype.toString.call()` 可以准确判断某个对象值属于哪种内置类型。在JavaScript中所有类型(如：对象，数组，函数)都包含一个内部属性\[[calss]]，此属性可以看作是一个内部分类。它并不是传统面向对象上的类,由于是内部属性，所以我们无法直接访问,不过，可以通过该方法转换为字符串来查看。除了null和undefined，其他都有各自的构造类，都是javascript的内置构造函数。
* `Object.getPrototypeOf()` 获取对象的原型（__proto__是厂商实现的私有属性，对环境有依赖，开发中应使用 Object.getPrototypeOf()来获取实例对象的原型）

##### 生成或设置

* `Object.create()` 方法创建一个新对象，使用现有的对象来提供新创建对象的__proto__。相当于直接将新生成对象的__proto__指向现有对象。Object.create里面的参数是一个原型对象。\[Object.create参数].isPrototypeOf(\[Object.create生成对象]) 为true。
* `Object.setPrototypeOf(obj, prototype)` 设置 prototype 为 obj 的新原型对象。

##### Object.create 和 Object.setPrototypeOf的区别

两者都能达到设置对象原型的功能，但是具体表现上有一些区别。

`Object.create(prototype[, propertiesObject])` 中，先创建一个 __proto__ 指向 prototype 的空对象，然后把这个空对象返回赋值，凸显了重新赋值。

`Object.setPrototypeOf(Ctor1.prototype, Ctor2.prototype)` 中， Ctor1.prototype 仍会指向原先 Ctor1 的 prototype，然后这个 Ctor1.prototype 的 __proto__ 则会指向 Ctor2.prototype，保留了原先 Ctor1.prototype 上的属性。

在进行俩个原型之间的委托时使用setPrototype更好，Object.create更适和直接对一个无原生原型的对象快速进行委托。

但是，由于现代js引擎优化属性访问所带来的特性的关系，更改对象的原型在各个浏览器和js引擎上都是一个很慢的操作。其在更改继承的性能上的影响是微妙而又广泛的，这不仅仅限于obj.__proto__ = ...语句上的时间花费，而且可能会延伸到任何代码，那些可以访问任何原型已被更改的对象的代码。如果关心性能，应该避免设置一个对象的原型，相反，应该使用Object.create()来创建带有想要原型的新对象。

#### 19. 属性的可枚举性和遍历

**ES6 规定，所有 Class 的原型的方法都是不可枚举的。**

* `for in` 会遍历对象包括其原型链上所有可枚举性的属性(不包含 Symbol 属性)
* `hasOwnProperty` 不会去查找原型，只会检查属性是否在当前对象中
* `Object.keys() Object.values() Object.entries()` 不会去查找原型，只会遍历对象本身可枚举（不包含 Symbol）属性（属性对）
* `Object.getOwnPropertyNames()` 不会去查找原型，会遍历对象本身所有（无论是否具有可枚举性，但是不包括 Symbol）的属性
* `Object.prototype.propertyIsEnumerable` 只会检查属性在该对象上是否可枚举
* `Object.getOwnPropertyDescriptors()` 获取一个对象的所有自身属性的描述符
* `Object.getOwnPropertyDescriptor()` 获取对象自身某个属性的描述符
* `Object.getOwnPropertySymbols()` 获取一个给定对象自身的所有 Symbol 属性的数组
* `Object.assign(target, ...source)`只拷贝自身的可枚举属性，会忽略`enumerable`为`false`的属性(仅限于source对象，target对象的不可枚举属性，也会拷贝)
* `JSON.stringify()` 只串行化对象自身的可枚举的属性(其第二个参数是个类似map的遍历函数，不过这个函数的入参是index、element),不包括 Symbol 属性。
* `Reflect.ownKeys(obj)` Reflect.ownKeys返回一个数组，包含对象自身的（不含继承的）所有键名，不管键名是 Symbol 或字符串，也不管是否可枚举。

`for in` `Object.keys()` `Object.getOwnPropertyNames()` `Object.getOwnPropertySymbols()` `Reflect.ownKeys()`遍历对象的键名，都遵守同样的属性遍历的次序规则:

* 首先遍历所有数值键，按照数值升序排列。
* 其次遍历所有字符串键，按照加入时间升序排列。
* 最后遍历所有 Symbol 键，按照加入时间升序排列。


#### 20. css值取百分比

* margin/padding取百分比的值时，都是基于父元素的宽度的
* 定位元素 left/right 和 top/bottom 取百分比的值时，则是分别按照父元素的宽高来计算的
* transform去百分比值时，都是基于自身的宽高