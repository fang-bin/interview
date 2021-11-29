## 1. 居中

![居中方式](https://github.com/fang-bin/interview/blob/master/image/set-center.jpeg)

## 2. 浮动

当元素浮动以后可以向左或向右移动，直到它的外边缘碰到包含它的框或者另外一个浮动元素的边框为止。元素浮动以后会脱离正常的文档流，所以文档的普通流中的框就变现的好像浮动元素不存在一样。

**优点**：这样做的优点就是在图文混排的时候可以很好的使文字环绕在图片周围。另外当元素浮动了起来之后，它有着块级元素的一些性质例如可以设置宽高等，但它与inline-block还是有一些区别的，第一个就是关于横向排序的时候，float可以设置方向而inline-block方向是固定的；还有一个就是inline-block在使用时有时会有空白间隙的问题

**缺点**：浮动元素一旦脱离了文档流，就无法撑起父元素，会造成父级元素的高度塌陷。

清除浮动影响的方法:
1. 后面兄弟元素设置 `clear: both;`
2. 父级添加overflow属性，或者设置高度
3. 伪元素选择器清除浮动
    ```css
    .parent::after{
      /* 设置添加子元素的内容是空 */
      content: '';  
      /* 设置添加子元素为块级元素 */
      display: block;
      /* 设置添加的子元素的高度0 */
      height: 0;
      /* 设置添加子元素看不见 */
      visibility: hidden;
      /* 设置clear：both */
      clear: both;
    }
    ```

## 3. 由 inline-block 引入间隙问题(间隙在水平和竖直方向都有)

[别人的demo](https://xxholic.github.io/lab/lab-css/inline-block.html)

元素被当成行内元素排版的时候，元素之间的空白符（空格、回车换行等）都会被浏览器处理，根据CSS中white-space属性的处理方式（默认是normal，合并多余空白），原来HTML代码中的回车换行被转成一个空白符，在字体不为0的情况下，空白符占据一定宽度，所以inline-block的元素之间就出现了空隙。

##### 去掉横向间隙的方法:

1. 避免使用空格、换行、tab、换页(虽然有效，但不方便识别和维护)
2. 使用letter-spacing
3. 使用word-spacing
4. font-size:0
5. 使用margin负值
6. 不适用inline-block，转而float

### 纵向间隙

**一个inline-block元素，如果里面没有inline内联元素，或者overflow不是visible，则该元素的基线就是其margin底边缘，否则，其基线就是元素里面最后一行内联元素的基线。**

[CSS深入理解vertical-align和line-height的基友关系](https://www.zhangxinxu.com/wordpress/2015/08/css-deep-understand-vertical-align-and-line-height/)

## 4. margin padding

margin padding的百分比都会根据父元素的宽度来计算

##### 为什么margin-top、margin-bottom、padding-top、padding-bottom 不根据父元素的高度计算？

因为这会导致一个无限循环，父元素的height会增加，以适应后代元素上下外边距的增加，而相应的，上下外边距因为父元素height的增加也会增加，形成无限循环。

还有一种说法是根本原因并不是因为死循环。例如张鑫旭认为相对于 height 计算，大多数情况下计算值都是 0，跟摆设没什么 区别，还不如相对宽度计算，因为 CSS 默认的是水平流，计算值一直会有效，而且我们还可以 利用这一特性实现一些有意思的布局效果。也就是面向场景和需求设计，这种设计可以让我们轻松实现自适应的等比例矩形效果。

可以用这种特性来实现诸如：自适应的正方形（固定比例的矩形）的效果，例如高度自适应的占位，可以避免闪烁。

其他宽高自适应的方案: vw长度单位，以屏幕宽度为参照物等。

## 5. 固定定位

（父元素相对定位）子元素绝对定位，定位值（top, left, right, bottom的具体值或者百分比）相对于父元素的内容区域(content+padding)，不包括父元素的border。

上述情况，子元素的width和height为百分比值时，也是如此。

相对定位的定位置(top, left, right, bottom)为百分比，也是相对于父元素的内容区域(content+padding)换算，并且偏移前的位置保留不动。

**注**: transform中的translate中的百分比值是相对于自身的，它也保留偏移前的位置。

## 圣杯布局(左右宽度固定，中间自适应)、双飞翼布局(淘宝UED团队在圣杯布局上完成)

##### 圣杯布局:

```html
<style>
.wrap{
  padding: 0 300px 0 200px;
  height: 300px;
  background-color: #ccc;
}
.mid,.left,.right{
  height: 200px;
}
.mid{
  float: left;
  width: 100%;
  background-color: #f00;
}
.left{
  float: left;
  width: 200px;
  margin-left: -100%;
  background-color: #0f0;
  position: relative;
  left: -200px;
}
.right{
  float: left;
  width: 300px;
  margin-left: -300px;
  position: relative;
  right: -300px;
  background-color: #00f;
}
</style>
<div class="wrap">
  <div class="mid">mid</div>
  <div class="left">left</div>
  <div class="right">right</div>
</div>
```
优点：不需要添加dom节点

缺点：正常情况下是没有问题的，当middle部分的宽小于left部分时就会发生布局混乱。（middle < left即会变形）

##### 双飞翼布局

```html
<style>
#center{
  float:left;
  width:100%;/*左栏上去到第一行*/     
  height:100px;
  background:blue;
}
#left{
  float:left;
  width:180px;
  height:100px;
  margin-left:-100%;
  background:#0c9;
}
#right{
  float:left;
  width:200px;
  height:100px;
  margin-left:-200px;
  background:#0c9;
}
/*给内部div添加margin，把内容放到中间栏，其实整个背景还是100%*/ 
#inside{
  margin:0 200px 0 180px;
  height:100px;
}
</style>
<div id="center">
    <div id="inside">middle</div>
</div>
<div id="left">left</div>
<div id="right">right</div>
```

优点：不会像圣杯布局那样变形

缺点是：多加了一层dom节点

其他实现类似效果的方案：

* grid布局
* flex布局
* table布局（年代久远，对seo也不好，不推荐）

[圣杯布局和双飞翼布局](https://www.cnblogs.com/jiguiyan/p/11425276.html)

## 6. 标准盒模型和怪异盒模型

标准盒模型，一个块的总宽度= width + margin(左右) + padding(左右) + border(左右)

怪异盒模型，一个块的总宽度= width + margin(左右)（即width已经包含了padding和border值）

## 7. CSS3选择器、继承属性
id选择器（#content），类选择器（.content）, 标签选择器（div, p, span等）, 相邻选择器（h1+p）, 子选择器（ul>li）, 后代选择器（li a）， 通配符选择器（*）, 属性选择器（a[rel = "external"]）， 伪类选择器（a:hover, li:nth-child）

选择器优先级：
!important > 行内样式 > id选择器 > 类选择器 === 伪类选择器 === 属性选择器 > 标签选择器 === 伪元素选择器 > 通配符选择器

可继承的样式属性： `font-size, font-family, color, ul, li, dl, dd, dt`;

不可继承的样式属性： `border, padding, margin, width, height`；

##### CSS3新增伪类

| 伪类 | 含义 |
| --- | --- |
| :root | 选择文档的根元素，等同于html元素 |
| :empty | 选择没有子元素的元素 |
| :target | 选取当前活动的目标元素 |
| :not(selector) | 选择除 selector 元素意外的元素 |
| :enabled | 选择可用的表单元素 |
| :disabled | 选择禁用的表单元素 |
| :checked | 选择被选中的表单元素 |
| :nth-child(n) | 匹配父元素下指定子元素，在所有子元素中排序第n |
| nth-last-child(n) | 匹配父元素下指定子元素，在所有子元素中排序第n，从后向前数|
| :nth-child(odd) | |
| :nth-child(even) | |
| :nth-child(3n+1) | |
| :first-child | |
| :last-child | |
| :only-child | |
| :nth-of-type(n) | 匹配父元素下指定子元素，在同类子元素中排序第n |
| :nth-last-of-type(n) | 匹配父元素下指定子元素，在同类子元素中排序第n，从后向前数 |
| :nth-of-type(odd) | |
| :nth-of-type(even) | |
| :nth-of-type(3n+1) | |
| :first-of-type | |
| :last-of-type | |
| :only-of-type | |
| ::selection | 选择被用户选取的元素部分（伪元素） |
| :first-line | 选择元素中的第一行（伪元素） |
| :first-letter | 选择元素中的第一个字符（伪元素） |
| :after | 在元素在该元素之后添加内容（伪元素） |
| :before | 在元素在该元素之前添加内容（伪元素） |

## 8. display有哪些值？他们的作用是什么？

<table spellcheck="false">
  <thead>
    <tr><th>值</th><th>作用</th></tr>
  </thead>
  <tbody><tr><td>none</td><td>使用后元素将不会显示</td></tr><tr><td>grid</td><td>定义一个容器属性为网格布局</td></tr><tr><td>flex</td><td>定义一个弹性布局</td></tr><tr><td>block</td><td>使用后元素将变为块级元素显示，元素前后带有换行符</td></tr><tr><td>inline</td><td>display默认值。使用后原色变为行内元素显示，前后无换行符</td></tr><tr><td>list-item</td><td>使用后元素作为列表显示</td></tr><tr><td>run-in</td><td>使用后元素会根据上下文作为块级元素或行内元素显示</td></tr><tr><td>table</td><td>使用后将作为块级表格来显示（类似<code>&lt;table&gt;</code>），前后带有换行符</td></tr><tr><td>inline-table</td><td>使用后元素将作为内联表格显示（类似<code>&lt;table&gt;</code>），前后没有换行符</td></tr><tr><td>table-row-group</td><td>元素将作为一个或多个行的分组来显示（类似<code>&lt;tbody&gt;</code>）</td></tr><tr><td>table-hewder-group</td><td>元素将作为一个或多个行的分组来表示（类似<code>&lt;thead&gt;</code>）</td></tr><tr><td>table-footer-group</td><td>元素将作为一个或多个行分组显示（类似<code>&lt;tfoot&gt;</code>）</td></tr><tr><td>table-row</td><td>元素将作为一个表格行显示（类似<code>&lt;tr&gt;</code>）</td></tr><tr><td>table-column-group</td><td>元素将作为一个或多个列的分组显示（类似<code>&lt;colgroup&gt;</code>）</td></tr><tr><td>table-column</td><td>元素将作为一个单元格列显示（类似<code>&lt;col&gt;</code>）</td></tr><tr><td>table-cell</td><td>元素将作为一个表格单元格显示（类似<code>&lt;td&gt;和&lt;th&gt;</code>）</td></tr><tr><td>table-caption</td><td>元素将作为一个表格标题显示（类似<code>&lt;caption&gt;</code>）</td></tr><tr><td>inherit</td><td>规定应该从父元素集成display属性的值</td></tr>
  </tbody>
</table>

## 9. position: sticky;

sticky: 可以设置 position:sticky 同时给一个 (top,bottom,right,left) 之一即可。一般用来做吸顶。

注意：

* 使用sticky时，必须指定top、bottom、left、right4个值之一，不然只会处于相对定位；
* sticky只在其父元素内起效果，且保证父元素的高度要高于sticky的高度；
* **父元素不能overflow:hidden或者overflow:auto等属性。**

## 10. css绘制三角形

隐藏三条边，颜色设定为（transparent）
```css
* {margin: 0; padding: 0;}
.content {
    width: 0;
    height: 0;
    margin: 0 auto;
    border-width: 20px;
    border-style: solid;
    border-color: transparent transparent pink transparent;  // 对应上右下左，此处为 下 粉色
}
```
```html
<div class="content"></div>
```

## 11. CSS优化、提高性能的方法有哪些

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

## 12. CSS预处理器/后处理器是什么？为什么要使用它们？

预处理器：如：less，sass，stylus,用来预编译sass或者less，增加了css代码的复用性，还有层级，mixin， 变量，循环， 函数等，对编写以及开发UI组件都极为方便。

后处理器： 如： postCss,通常被视为在完成的样式表中根据css规范处理css，让其更加有效。目前最常做的是给css属性添加浏览器私有前缀，实现跨浏览器兼容性的问题。

css预处理器为css增加一些编程特性，无需考虑浏览器的兼容问题，我们可以在CSS中使用变量，简单的逻辑程序，函数等在编程语言中的一些基本的性能，可以让我们的css更加的简洁，增加适应性以及可读性，可维护性等。

使用原因：

* 结构清晰， 便于扩展
* 可以很方便的屏蔽浏览器私有语法的差异
* 可以轻松实现多重继承
* 完美的兼容了CSS代码，可以应用到老项目中

## 13. 如果需要手动写动画，你认为最小时间间隔是多久，为什么？

多数显示器默认频率是60Hz，即1秒刷新60次，所以理论上最小间隔为1/60＊1000ms ＝ 16.7ms。

## 14. rgba() 和 opacity 的透明效果有什么不同？

opacity 作用于元素以及元素内的所有内容（包括文字）的透明度；

rgba() 只作用于元素自身的颜色或其背景色，并不会作用于子元素；

## 15. png、jpg、 jpeg、 bmp、gif 这些图片格式解释一下，分别什么时候用。有没有了解过webp？

1. png-便携式网络图片（Portable Network Graphics）,是一种无损数据压缩位图文件格式。优点是：压缩比高，色彩好。 大多数地方都可以用。
2. jpg是一种针对相片使用的一种失真压缩方法，是一种破坏性的压缩，在色调及颜色平滑变化做的不错。在www上，被用来储存和传输照片的格式。
3. gif是一种位图文件格式，以8位色重现真色彩的图像。可以实现动画效果。
4. bmp的优点： 高质量图片；缺点： 体积太大； 适用场景： windows桌面壁纸
5. webp格式是谷歌在2010年推出的图片格式，压缩率只有jpg的2/3，大小比png小了45%。缺点是压缩的时间更久了，兼容性不好，目前谷歌和opera支持。
WebP 的优势体现在它具有更优的图像数据压缩算法，能带来更小的图片体积，而且拥有肉眼识别无差异的图像质量；同时具备了无损和有损的压缩模式、Alpha 透明以及动画的特性，在 JPEG 和 PNG 上的转化效果都相当优秀、稳定和统一。

## 16. 在网页中的应该使用奇数还是偶数的字体？
在网页中的应该使用“偶数”字体：

偶数字号相对更容易和 web 设计的其他部分构成比例关系，使用奇数号字体时文本段落无法对齐，宋体的中文网页排布中使用最多的就是 12 和 14。

## 17. display: none; 与 visibility: hidden; 有什么区别？

联系： 这两个属性的值都可以让元素变得不可见；

区别： 

* 从占据空间角度看：display: none;会让元素完全从渲染树中消失，渲染的时候不占据任何空间；visibility: hidden;不会让元素从渲染树消失，渲染后元素继续占据空间，只是内容不可见；
* 从继承方面角度看：display: none;是非继承属性，子孙节点消失由于父元素从渲染树消失造成，通过修改子孙节点属性无法显示；visibility:hidden;是继承属性，子孙节点消失由于继承了hidden，通过设置visibility: visible;可以让子孙节点显式；
* 从重绘和重排角度看：修改常规流中元素的display通常会造成文档重排。修改visibility属性只会造成本元素的重绘
读屏器不会读取display: none;元素内容；会读取visibility: hidden元素内容；

## 18. css hack原理及常用hack有哪些？

原理： 利用不同浏览器对CSS的支持和解析结果不一样编写针对特定浏览器的样式。

常见的hack有： 属性hack、选择器hack、IE条件注释。

## 19. link 与 @import 的区别？

* @import 是CSS方式，link 是HTML提供的标签，不仅可以加载CSS，还可以定义RSS,rel等连接属性。
* 加载页面时，link标签引入的 CSS 被同时加载；@import引入的 CSS 将在页面加载完毕后被加载（相当于和页面的加载是串行）。
* 可以通过 JS 操作 DOM ，插入link标签来改变样式；由于DOM方法是基于文档的，无法使用@import的方式插入样式。
* link最大限度支持并行下载，@import 过多嵌套导致串行下载，出现FOUC；
* 浏览器对 link 支持早于@import ，可以使用 @import对老浏览器隐藏样式；

总的来说： link优于@import。

## 20. 什么是 FOUC(Flash of Unstyled Content)？ 如何来避免 FOUC？

当使用@import导入CSS时，会导致某些页面在IE出现奇怪的现象： 没有样式的页面内容显示瞬间闪烁，这种现象被称为“文档样式暂时失效”，简称FOUC。

产生原因： 当样式表晚于结构性html加载时，加载到此样式表时，页面将会停止之前的渲染。等待此样式表被下载和解析后，再重新渲染页面，期间导致短暂的花屏现象。

解决办法： 只要在\<head>之间加入一个\<link>或者\<script>元素即可。

## 21. 制作一个访问量很大的网站，如何管理所有的css文件，js和图片？

从人手，分工和同步方面回答：

* 前期团队必须确认好全局样式，编码模式；
* 代码风格，编写习惯保持一致；
* 标注样式编写人，各模块都要及时标注（标注关键样式调用的地方）；
* 对自己负责的页面进行标注；
* CSS和JS分文件夹存并行存放，命名都要统一；
* JS分文件夹存放，明明以该JS功能为准的英文翻译；
* 图片采用整合的.png格式存放，尽量整合在一起，方便将来管理；

## 22. 经常遇到的浏览器的兼容性或者奇葩bug有哪些？原因，解决方法是什么？

* 浏览器都有附带的默认样式，它们可能不一样。
  
  解决：可以通过引入reset.css来进行统一。

* transform会让字体模糊。
  
  原因：可能出现带小数的样式，transform在渲染非整数的 px 时就会出现字体模糊。
  
  解决：可以将数值设置为偶数；用其他方式替换，例如margin；

* 默认情况下，设备会自动识别任何可能是电话号码、邮箱等字符串。
  
  解决：设置`<meta name="format-detection"content="telephone=no">`

* 滚动卡顿

  解决：`-webkit-overflow-scrolling: touch;`

  `auto` 使用普通滚动, 当手指从触摸屏上移开，滚动会立即停止。

  `touch` 使用具有回弹效果的滚动, 当手指从触摸屏上移开，内容会继续保持一段时间的滚动效果。继续滚动的速度和持续的时间和滚动手势的强烈程度成正比。同时也会创建一个新的堆栈上下文。(现在好像不行了)

* border设置0.5px

  解决：可以通过`transform: scale()`来实现

* 移动端点击300ms延迟

  原因：为了让移动端小屏幕浏览桌面端网页，设置了双击缩放，，这也是会有上述 300 毫秒延迟的主要原因。

  解决： 
    1. 完全禁止缩放（没试过） 
    2. 引用第三方库，例如fastclick。其原理是监听 touchend 事件，通过DOM自定义事件立即出发模拟一个click事件，并把浏览器在300ms之后的click事件阻止掉。
    3. 使用 touchend 事件替代。（不能使用 touchstart 事件，主要是因为可能用户只是想滚动屏幕，却会触发事件，同时某些情景也会出现点击穿透）

* 移动端点击穿透

  现象：

  在发生触摸动作约300ms之后，移动端会模拟产生click动作，它底下的具有点击特性的元素也会被触发，这种现象称为点击穿透。

  发生的条件：

  上层元素监听了触摸事件，触摸之后该层元素消失，下层元素具有点击特性（监听了click事件或默认的特性（a标签、input、button标签））

  解决：
    1. 最简单，最彻底的方法就是将页面内所有的 click 事件替换为 touch 事件
    2. 全部用 click 事件，不过这样在移动端会有300ms延迟
    3. 隐藏本身元素的操作设置一个超过300ms的延迟。
    4. touch元素隐藏后，给下面元素添上`pointer-events: none;`样式，让click穿过去，350ms后去掉这个样式，恢复响应。
    pointer-events CSS 属性指定在什么情况下 (如果有) 某个特定的图形元素可以成为鼠标事件的 target。
    默认为auto，设置为none，元素永远不会成为鼠标事件的target。
    5. 用fastclick这类第三方库

###### fastclick实现原理

```javascript
// 业务代码
var $test = document.getElementById('test')
$test.addEventListener('click', function () {
  console.log('1 click')
})

// FastClick简单实现
var targetElement = null
document.body.addEventListener('touchstart', function () {
  // 记录点击的元素
  targetElement = event.target
})
document.body.addEventListener('touchend', function (event) {
  // 阻止默认事件（屏蔽之后的click事件）
  event.preventDefault()
  var touch = event.changedTouches[0]
  // 合成click事件，并添加可跟踪属性forwardedTouchEvent
  var clickEvent = document.createEvent('MouseEvents')
  clickEvent.initMouseEvent('click', true, true, window, 1, touch.screenX, touch.screenY, touch.clientX, touch.clientY, false, false, false, false, 0, null)
  clickEvent.forwardedTouchEvent = true // 自定义的
  targetElement.dispatchEvent(clickEvent)
})
```

* 块级行内元素垂直对齐。

## 23. 伪元素和伪类的区别和作用？
伪类的效果可以通过添加实际的类来实现，而伪元素的效果可以通过添加实际的元素来实现。所以它们的本质区别就是是否抽象创造了新元素。

伪元素： 不存在在DOM文档中，是虚拟的元素，是创建新元素。代表某个元素的子元素，这个子元素虽然在逻辑上存在，但却并不实际存在于文档树中。

伪类：存在DOM文档中，逻辑上存在但在文档树中却无须标识的“幽灵”分类。

## 24. css sprites的优缺点？
优点：
* 利用CSS Sprites能很好地减少网页的http请求，从而大大提高了页面的性能，这也是CSS Sprites最大的优点；
* CSS Sprites能减少图片的字节，曾经多次比较过，把3张图片合并成1张图片的字节总是小于这3张图片的字节总和。

缺点: 开发和维护比较困难

**目前网站开发所用的精灵图（如字体库）一般都是直接用云端，而不是采用这种本地的了，如阿里图标库等。**

## 25. getComputedStyle

是一个可以获取当前元素所有最终使用的CSS属性值。返回的是一个CSS样式声明对象([object CSSStyleDeclaration])，只读。

jQuery的CSS()方法，其底层运作就应用了getComputedStyle以及getPropertyValue方法。

用法: `window.getComputedStyle("元素", "伪类");` 其中`window.getComputedStyle`等价于`document.defaultView.getComputedStyle`

**getComputedStyle会引起回流，因为它需要获取祖先节点的一些信息进行计算（譬如宽高等），所以用的时候慎用，回流会引起性能问题。**

##### getComputedStyle 和 style 的区别

* getComputedStyle 方法是只读的，只能获取样式，不能设置；而element.style能读能写。
* getComputedStyle 方法获取的是最终应用在元素上的所有CSS属性对象（即使没有CSS代码，也会把默认的祖宗八代都显示出来）；而 element.style 只能获取**元素style属性**中的CSS样式。因此对于一个光秃秃的元素 \<p> ，getComputedStyle 方法返回对象中 length 属性值（如果有）就是190+, 而 element.style 就是0。

##### getComputedStyle与currentStyle

currentStyle 是IE浏览器自娱自乐的一个属性，其与 element.style 可以说是近亲，至少在使用形式上类似，差别在于 element.currentStyle 返回的是元素当前应用的最终 CSS 属性值（包括外链 CSS 文件，页面中嵌入的 \<style> 属性等）。

因此，从作用上将，getComputedStyle 方法与 currentStyle 属性走的很近，形式上则 style 与 currentStyle 走的近。不过，currentStyle 属性貌似不支持伪类样式获取，这是与 getComputedStyle 方法的差异，也是 jQuery css() 方法无法体现的一点。

#### getPropertyValue和getAttribute

在老的IE浏览器（包括最新的），getAttribute 方法提供了与 getPropertyValue 方法类似的功能，可以访问CSS样式对象的属性。

`window.getComputedStyle("元素").getPropertyValue('属性名')`

## 26. 什么是Critical CSS？

Critical CSS是一种提取首屏中 CSS 的技术，以便尽快将内容呈现给用户。这是快速加载网页首屏的好方法。

核心思路：
1. 抽取出首页的CSS；
2. 用行内css样式，加载这部分的css(critical CSS);
3. 等到页面加载完之后，再加载整个css，会有一部分css与critical css重叠；

## 27. 格式化上下文

BFC最初被定义在css2.1规范的Visual formatting model中。

##### 视觉格式化模型(Visual formatting model)

视觉格式化模型是用来处理文档并将它显示在视觉媒体上的机制，它让视觉媒体知道如何处理文档。（视觉媒体——user agent通常指的浏览器。）

在视觉格式化模型中，文档树的每个元素根据盒模型生成零个或多个盒子。

这些盒子的布局受以下因素控制：

* 盒子的尺寸和类型。
* 定位方案（普通文档流，浮动流和绝对定位流）。 
* 文档树中元素间的关系。
* 外部因素（如视口大小，图像本身的尺寸等）。

**视觉格式化模型会根据CSS盒子模型将文档中的元素转换为一个个盒子。**

##### 定位方案

css布局宏观上来说是受定位方案影响，包括以下几种：

* 普通文档流

  元素按照其在 HTML 中的位置顺序决定排布的过程。并且这种过程遵循标准的描述。

  只要不是float和绝对定位方式布局的，都在普通流里面。

* 浮动文档流

  浮动框不在文档的普通流中，浮动的流会漂浮在普通的流上面。

  浮动的框可以向左或向右移动，直到它的外边缘碰到包含框或另一个浮动框的边框为止。

* 定位文档流

  1. 相对定位在普通流之中，是相对于它在普通流中的位置中进行移动，元素占据原来位置

  2. 绝对定位脱离普通流，不占据空间相对于距离它最近的那个已定位的祖先(相对/绝对)元素决定的。

  3. 固定定位，相对于浏览器窗口定位，脱离普通流，不占据空间


##### 格式化上下文

Formatting Context，既格式化上下文。用于决定如何渲染文档的一个区域。

可以简单理解为格式化上下文就是为盒子准备的一套渲染规则。

常见的有：

* BFC（Block Formatting Context）
* IFC（Inline Formatting Context）
* FFC（Flex Formatting Context）
* GFC（Grid Formatting Context）
### BFC

BFC的全称为Block Formatting Context，即块级格式化上下文。规定了块级盒子的渲染布局方式。一个BFC有如下特性：
* 处于同一个BFC中的元素相互影响，可能会发生margin collapse；（BFC垂直方向边距重叠，FFC和GFC并不会）
* BFC在页面上是一个独立的容器，容器里面的子元素不会影响到外面的元素，反之亦然；
* 计算BFC的高度时，考虑BFC所包含的浮动元素，其也参与计算；
* 浮动盒的区域不会叠加到BFC、IFC上；

创建BFC的方法如下
* 根元素
* 浮动（float的值不为none）；
* 绝对定位元素（position的值为absolute或fixed）；
* 行内块（display为inline-block）
* 表格单元（display为table、table-cell、table-caption等HTML表格相关属性）；
* overflow不为visible；

在最新的CSS3规范中，以下分别会创建FFC和GFC

* 弹性盒（display为flex或inline-flex）；
* 网格元素（display为 grid 或 inline-grid 元素的直接子元素） 等等。

BFC的使用场景

* 防止垂直margin重叠，父子元素的边界重叠(当一个元素包含在另一个元素之中时，子元素与父元素之间也会产生重叠现象，重叠后的外边距，等于其中最大者)，在父元素上加上overflow:hidden;使其成为BFC。

* 防止浮动导致父元素高度塌陷，清除内部浮动对元素高度的影响。父元素#float的高度为0，解决方案为为父元素#float创建BFC，这样浮动子元素的高度也会参与到父元素的高度计算。

* 自适应两栏布局(原理是BFC不会与float元素发生重叠。)

## 28. 层叠上下文

层叠上下文，英文称作”stacking context”. 是HTML中的一个三维的概念。如果一个元素含有层叠上下文，我们可以理解为这个元素在z轴上就“高人一等”。

“层叠水平”英文称作”stacking level”，决定了同一个层叠上下文中元素在z轴上的显示顺序。

普通元素的层叠水平优先由层叠上下文决定，因此，层叠水平的比较只有在当前层叠上下文元素中才有意义。

（注: 不要把层叠水平和CSS的z-index属性混为一谈。没错，某些情况下z-index确实可以影响层叠水平，但是，只限于定位元素以及flex盒子的孩子元素；而层叠水平所有的元素都存在。）

层叠顺序，表示元素发生层叠时候有着特定的垂直显示顺序，注意，这里跟上面两个不一样，上面的层叠上下文和层叠水平是概念，而这里的层叠顺序是规则。

在CSS2.1的年代，在CSS3还没有出现的时候（注意这里的前提），层叠顺序规则遵循下面这张图：

![CSS2.1层叠规则](https://github.com/fang-bin/interview/blob/master/image/z-stack-context.png)

诸如border/background一般为装饰属性，而浮动和块状元素一般用作布局，而内联元素都是内容。

网页中最重要的内容的层叠顺序相当高，当发生层叠是很好，重要的文字啊图片内容可以优先暴露在屏幕上。

这些层叠顺序规则还是老时代的，CSS3就不一样了。

##### 务必牢记的层叠准则
1. 谁大谁上：当具有明显的层叠水平标示的时候，如识别的z-indx值，在同一个层叠上下文领域，层叠水平值大的那一个覆盖小的那一个。
2. 后来居上：当元素的层叠水平一致、层叠顺序相同的时候，在DOM流中处于后面的元素会覆盖前面的元素。

#### 层叠上下文的特性

* 层叠上下文的层叠水平要比普通元素高
* 层叠上下文可以阻断元素的混合模式
* 层叠上下文可以嵌套，内部层叠上下文及其所有子元素均受制于外部的层叠上下文。
* 每个层叠上下文和兄弟元素独立，也就是当进行层叠变化或渲染的时候，只需要考虑后代元素。
* 每个层叠上下文是自成体系的，当元素发生层叠的时候，整个元素被认为是在父层叠上下文的层叠顺序中。

#### 层叠上下文的创建
* 根层叠上下文
  指的是页面根元素，也就是滚动条的默认的始作俑者\<html>元素。这就是为什么，绝对定位元素在left/top等值定位的时候，如果没有其他定位元素限制，会相对浏览器窗口定位的原因。
* 定位元素与传统层叠上下文
  对于包含有position:relative/position:absolute的定位元素，以及FireFox/IE浏览器（不包括Chrome等webkit内核浏览器，它们position:fixed元素天然层叠上下文元素，无需z-index为数值。）下含有position:fixed声明的定位元素，当其z-index值不是auto的时候，会创建层叠上下文。
* CSS3与新时代的层叠上下文

    * z-index值不为auto的**flex子项**(父元素display:flex|inline-flex).
    * 元素的opacity值不是1.
    * 元素的transform值不是none.
    * 元素mix-blend-mode值不是normal.
    * 元素的filter值不是none.
    * 元素的isolation值是isolate.
    * will-change指定的属性值为上面任意一个。
    * 元素的-webkit-overflow-scrolling设为touch.

##### 详解CSS3与新时代的层叠上下文

[深入理解层叠上下文-张鑫旭](https://www.zhangxinxu.com/wordpress/2016/01/understand-css-stacking-context-order-z-index/)

![完善的7阶层叠顺序图](https://github.com/fang-bin/interview/blob/master/image/stacking-context-order.png)

这个地方挺重要。


##### 这里稍微补充一下transform的一些特性

[transform对N多元素渲染影响](https://www.zhangxinxu.com/wordpress/2015/05/css3-transform-affect/)

1. 提升元素的垂直地位(就是上面说的生成不依赖z-index的层叠上下文，地位与`postion: absoulte; z-index:auto;`)相当

2. 在Chrome、Firefox中，祖先元素设置了transform会限制子孙元素position:fixed的效果，降级为postion:absolute的表现。

3. transform改变overflow对absolute元素的限制
  absolute绝对定位元素，如果含有overflow不为visible的父级元素，同时，该父级元素以及到该绝对定位元素之间任何嵌套元素都没有position为非static属性的声明，则overflow对该absolute元素不起作用。(这玩意我原先都不知道)
  而不管overflow容器还是嵌套子元素，只要有transform属性，就会hidden溢出的absolute元素。

## 29. mix-blend-mode / background-blend-mode (混合模式)

现在Chrome 和 Firefox都支持良好

#### mix-blend-mode 和 isolation: isolate;

mix-blend-mode默认情况下是会混合所有比起层叠顺序低的元素的，混合模式只到某一个元素，或者只是某一组元素怎么办呢？isolation: isolate就是为了解决这个问题产生的。

isolation:isolate之所以可以阻断混合模式的进行，本质上是因为isolation:isolate创建一个新的层叠上下文(stacking context)。

其实任何可以创建层叠上下文的属性都可以阻断mix-blend-mode的生效。

从这个角度来看，isolation:isolate除了创建层叠上下文，其他没有任何用。

```css
mix-blend-mode: normal;          //正常
mix-blend-mode: multiply;        //正片叠底
mix-blend-mode: screen;          //滤色
mix-blend-mode: overlay;         //叠加
mix-blend-mode: darken;          //变暗
mix-blend-mode: lighten;         //变亮
mix-blend-mode: color-dodge;     //颜色减淡
mix-blend-mode: color-burn;      //颜色加深
mix-blend-mode: hard-light;      //强光
mix-blend-mode: soft-light;      //柔光
mix-blend-mode: difference;      //差值
mix-blend-mode: exclusion;       //排除
mix-blend-mode: hue;             //色相
mix-blend-mode: saturation;      //饱和度
mix-blend-mode: color;           //颜色
mix-blend-mode: luminosity;      //亮度

mix-blend-mode: initial;         //初始
mix-blend-mode: inherit;         //继承
mix-blend-mode: unset;           //复原
```

#### background-blend-mode

background-blend-mode这个要更好理解一点，背景的混合模式。可以是背景图片见的混合，也可以是背景图片和背景色的混合。

## 30. will-change

用来增强页面渲染性能

3D transform会启用GPU加速，例如translate3D, scaleZ之类，但是呢，这些属性业界往往称之为hack加速法。我们实际上不需要z轴的变化。

（注： GPU即图形处理器，是与处理和绘制图形相关的硬件。GPU是专为执行复杂的数学和几何计算而设计的，可以让CPU从图形处理的任务中解放出来，从而执行其他更多的系统任务，例如，页面的计算与重绘。）

当我们通过某些行为（点击、移动或滚动）触发页面进行大面积绘制的时候，浏览器往往是没有准备的，只能被动使用CPU去计算与重绘，由于没有事先准备，应付渲染够呛，于是掉帧，于是卡顿。而will-change则是真正的行为触发之前告诉浏览器，什么可能要触发大量绘制，浏览器会调用GPU，从容应对即将到来的变形。

```css
/* 关键字值 */
will-change: auto;    /*默认，没luan用*/
will-change: scroll-position;  /*告诉浏览器，我要开始翻滚了*/
will-change: contents;        /*告诉浏览器，内容要动画或变化了。*/
will-change: transform;        /* <custom-ident>示例 */
will-change: opacity;          /* <custom-ident>示例 */
will-change: left, top;        /* 两个<animateable-feature>示例 */

/* 全局值 */
will-change: inherit;
will-change: initial;
will-change: unset;
```

will-change虽然可以加速，但是，一定一定要适度使用。那种全局都开启will-change等待模式的做法，无疑是死路一条。

平常的渲染处理，手机都是可以比较流畅的。完全没有必要以牺牲其他东西(手机电量等)来实现。

如果使用JS添加will-change, 事件或动画完毕，一定要及时remove.（css也是如此，可以分开写，不用一直挂在常态样式上面）

## 31. 超链接访问过后hover样式就不出现了

因为被点击访问过的超链接样式不再具有hover和active了。解决方法是改变CSS属性的排列顺序:L-V-H-A ;

```javascript
ele:link{}
ele:visited{}
ele:hover{}
ele:active{}
```

## 32. CSS的伪类选择器

* ele:link
* ele:visited
* ele:hover
* ele:active
* f ele:first-child
* f ele:last-child
* f ele:nth-child(n) f元素内所有ele元素的第n个元素(如果第n个元素不是ele类型，则不会选中)
* f ele:nth-of-type(n) f元素下第n个ele类型的元素

## 33. word-break word-wrap white-space 的区别

1. white-sapce
  用来控制空白字符的显示，同时还能控制是否自动换行。
    * normal 默认，空白会被浏览器忽略。(连续空白字符合并)
    * nowrap 永不换行（自动换行、换行符都不能换行，连续空白字符也会合并，但是\<br />标签可以换行）
    * pre 空格和换行符都保留，但是不会自动换行（preserve的缩写：保留）
    * pre-wrap 保留空格和换行符，并且可以自动换行（preserve+wrap的意思）
    * pre-line 合并空格，但是保留换行符和自动换行

2. word-break
  控制单词如何被拆分换行
    * normal 默认，浏览器默认规则
    * keep-all  所有单词一律不能拆分换行（只能在半角空白字符或者连字符处换行）(这里的单词包括连续的中文字符等)（中文字符中，碰到空格可以出发自动换行）
    * break-all 所有单词碰到边界一律拆分换行(不论单词长短)
    * break-word 和word-wrap的break-word效果一样，不过只有Chrome、Safari等支持
3. word-wrap (CSS3中叫 overflow-wrap)
  控制单词如何被拆分换行，实际上是作为word-break的互补。
    * normal 默认，浏览器默认规则
    * break-word 只有当一个单词一整行都显示不下时，才会拆分换行该单词。

## 34. 深入理解vertical-align 

工作中，我经常遇到明明设置了line-height和行高一致，在安卓上就会偏上，如果设置了`overflow:hideen;`还会截断字体一部分。

而且字体和一些行内元素/inline-block元素，设置了行高和`vertical-align:middle`也会有一些偏差。

当然这些情况大多出现在安卓系统上。

### CSS的排版布局

**line-height行高的定义就是两基线的间距**

**vertical-align的默认值就是基线；**

#### 基线

字母x的下边缘(线)就是基线。

![基线](https://github.com/fang-bin/interview/blob/master/image/baseline.png)

CSS中有一个概念叫做"x-height", 指的是字母'x'的高度。

![x-height](https://github.com/fang-bin/interview/blob/master/image/x-height.png)

需要了解下"x-height"的含义，通俗讲，"x-height"就是指的小写字母'x'的高度；术语描述就是基线和等分线 mean line (也称作中线[midline])之间的距离。

* ascender height: 上下线高度
* cap height: 大写字母高度
* median: 中线
* descender height: 下行线高度

##### vertical: middle;

规范中对垂直对齐的middle，middle指的是基线往上1/2 "x-height"高度。我们可以近似脑补成字母x交叉点那个位置。

有此可见，vertical-align: middle并不是绝对的垂直居中对齐，我们平常看到的middle效果只是一种近似的效果。原因很简单，因为不同的字体，其在行内盒子中的位置是不一样的，比方说’微软雅黑’就是一个字符下沉比较明显的字体，所有字符的位置相比其他字体要偏下一点。要是vertical-align: middle是相对容器中分线对齐，呵呵，你会发现图标和文字不在一条线上，而相对于字符x的中心位置对齐，我们肉眼看上去就好像和文字居中对齐了。

在HTML5文档声明下，块状元素内部的内联元素的行为表现，就好像块状元素内部还有一个（更有可能两个-前后）看不见摸不着没有宽度没有实体的空白节点，这个假想又似乎存在的空白节点，张鑫旭称之为“幽灵空白节点”。

**一个inline-block元素，如果里面没有inline内联元素，或者overflow不是visible，则该元素的基线就是其margin底边缘，否则，其基线就是元素里面最后一行内联元素的基线。**

以上相关内容，重点可以参考:

[字母’x’在CSS世界中的角色和故事](https://www.zhangxinxu.com/wordpress/2015/06/about-letter-x-of-css/)

[CSS深入理解vertical-align和line-height的基友关系](https://www.zhangxinxu.com/wordpress/2015/08/css-deep-understand-vertical-align-and-line-height/)

[彻底搞定vertical-align垂直居中不起作用疑难杂症](https://juejin.im/post/6844903561780789255)

[关于display: inline-block产生的间隙](https://github.com/XXHolic/blog/issues/13)

[Android浏览器下line-height垂直居中为什么会偏离？](https://www.zhihu.com/question/39516424)

[解决 Android 浏览器下 line-height 垂直居中偏离问题](https://github.com/o2team/H5Skills/issues/4)

[关于 Android 下 line-height 文字垂直居中偏移的思考](https://rprns.me/2018/07/27/%E5%85%B3%E4%BA%8E%20Android%20%E4%B8%8B%20line-height%20%E6%96%87%E5%AD%97%E5%9E%82%E7%9B%B4%E5%B1%85%E4%B8%AD%E5%81%8F%E7%A7%BB%E7%9A%84%E6%80%9D%E8%80%83/)