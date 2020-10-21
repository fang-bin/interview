## 1. 居中

![avator](http://47.98.159.95/my_blog/007/center.jpg)

## 2. 浮动

当元素浮动以后可以向左或向右移动，直到它的外边缘碰到包含它的框或者另外一个浮动元素的边框为止。元素浮动以后会脱离正常的文档流，所以文档的普通流中的框就变现的好像浮动元素不存在一样。

**优点**：这样做的优点就是在图文混排的时候可以很好的使文字环绕在图片周围。另外当元素浮动了起来之后，它有着块级元素的一些性质例如可以设置宽高等，但它与inline-block还是有一些区别的，第一个就是关于横向排序的时候，float可以设置方向而inline-block方向是固定的；还有一个就是inline-block在使用时有时会有空白间隙的问题

**缺点**：浮动元素一旦脱离了文档流，就无法撑起父元素，会造成父级元素的高度塌陷。

清除浮动影响的方法:
1. 后面元素设置 `clear: both;`
2. 父级添加overflow属性，或者设置高度
3. 伪类选择器清除浮动
    ```css
    .parent:after{
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

### 纵向间隙(涉及到浏览器渲染规则)

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

圣杯布局:
* flex布局
* float布局 
  1. 父元素左右padding值，left和right子元素设置margin-left**负值**，middle设置width:100% 实现。
  2. 左右元素设置left\right浮动，利用浮动元素的环绕特性实现
* 绝对定位 中间元素设置left和right值实现
* grid布局
* table布局（年代久远，对seo也不好，不推荐）

双飞翼布局: 也可使用上面的布局方式

## 6. 标准盒模型和怪异盒模型

标准盒模型，一个块的总宽度= width + margin(左右) + padding(左右) + border(左右)

怪异盒模型，一个块的总宽度= width + margin(左右)（即width已经包含了padding和border值）

## 7. CSS3选择器、继承属性
id选择器（#content），类选择器（.content）, 标签选择器（div, p, span等）, 相邻选择器（h1+p）, 子选择器（ul>li）, 后代选择器（li a）， 通配符选择器（*）, 属性选择器（a[rel = "external"]）， 伪类选择器（a:hover, li:nth-child）

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
* sticky只在其父元素内其效果，且保证父元素的高度要高于sticky的高度；
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

rgba() 只作用于元素自身的颜色或其背景色，子元素不会继承透明效果；

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

* 从占据空间角度看：display: none;会让元素完全从渲染树中消失，渲染的时候不占据任何空间；visibility: hidden;不会让元素从渲染树消失，渲染师元素继续占据空间，只是内容不可见；
* 从继承方面角度看：display: none;是非继承属性，子孙节点消失由于元素从渲染树消失造成，通过修改子孙节点属性无法显示；visibility:hidden;是继承属性，子孙节点消失由于继承了hidden，通过设置visibility: visible;可以让子孙节点显式；
* 从重绘和重排角度看：修改常规流中元素的display通常会造成文档重排。修改visibility属性只会造成本元素的重绘
读屏器不会读取display: none;元素内容；会读取visibility: hidden元素内容；

## 18. css hack原理及常用hack有哪些？

原理： 利用不同浏览器对CSS的支持和解析结果不一样编写针对特定浏览器的样式。

常见的hack有： 属性hack、选择器hack、IE条件注释。

## 19. link 与 @import 的区别？

* link 是HTML方式， @import 是CSS方式；
* link最大限度支持并行下载，@import 过多嵌套导致串行下载，出现FOUC；
* link 可以通过 rel="alternate stylesheet" 指定候选样式；
* 浏览器对 link 支持早于@import ，可以使用 @import对老浏览器隐藏样式；
* @import必须在样式规则之前，可以在css文件中引用其他文件；

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

## 22. 经常遇到的浏览器的兼容性有哪些？原因，解决方法是什么？

块级行内元素垂直对齐。

## 23. 伪元素和伪类的区别和作用？
伪类的效果可以通过添加实际的类来实现，而伪元素的效果可以通过添加实际的元素来实现。所以它们的本质区别就是是否抽象创造了新元素。

伪元素： 不存在在DOM文档中，是虚拟的元素，是创建新元素。代表某个元素的子元素，这个子元素虽然在逻辑上存在，但却并不实际存在于文档树中。

伪类：存在DOM文档中，逻辑上存在但在文档树中却无须标识的“幽灵”分类。

## 24. css sprites的优缺点？
优点：
* 利用CSS Sprites能很好地减少网页的http请求，从而大大提高了页面的性能，这也是CSS Sprites最大的优点；
* CSS Sprites能减少图片的字节，曾经多次比较过，把3张图片合并成1张图片的字节总是小于这3张图片的字节总和。

缺点: 开发和维护比较困难

目前网站开发所用的精灵图（如字体库）一般都是直接用云端，而不是采用这种本地的了，如阿里图标库等。

## 25. getComputedStyle

是一个可以获取当前元素所有最终使用的CSS属性值。返回的是一个CSS样式声明对象([object CSSStyleDeclaration])，只读。

jQuery的CSS()方法，其底层运作就应用了getComputedStyle以及getPropertyValue方法。

用法: `window.getComputedStyle("元素", "伪类");`

##### getComputedStyle 和 style 的区别

* getComputedStyle方法是只读的，只能获取样式，不能设置；而element.style能读能写。
* getComputedStyle方法获取的是最终应用在元素上的所有CSS属性对象（即使没有CSS代码，也会把默认的祖宗八代都显示出来）；而element.style只能获取元素style属性中的CSS样式。因此对于一个光秃秃的元素<p>，getComputedStyle方法返回对象中length属性值（如果有）就是190+, 而element.style就是0。

##### getComputedStyle与currentStyle

currentStyle是IE浏览器自娱自乐的一个属性，其与element.style可以说是近亲，至少在使用形式上类似，element.currentStyle，差别在于element.currentStyle返回的是元素当前应用的最终CSS属性值（包括外链CSS文件，页面中嵌入的\<style>属性等）。

因此，从作用上将，getComputedStyle方法与currentStyle属性走的很近，形式上则style与currentStyle走的近。不过，currentStyle属性貌似不支持伪类样式获取，这是与getComputedStyle方法的差异，也是jQuery css()方法无法体现的一点。

#### getPropertyValue和getAttribute

在老的IE浏览器（包括最新的），getAttribute方法提供了与getPropertyValue方法类似的功能，可以访问CSS样式对象的属性。

`window.getComputedStyle("元素").getPropertyValue('属性名')`

## 26. 什么是critical CSS？

Critical CSS是一种提取首屏中 CSS 的技术，以便尽快将内容呈现给用户。这是快速加载网页首屏的好方法。

核心思路：
1. 抽取出首页的CSS；
2. 用行内css样式，加载这部分的css(critical CSS);
3. 等到页面加载完之后，再加载整个css，会有一部分css与critical css重叠；

## 外边距折叠(collapsing margins)

## 格式化上下文

## 层叠上下文

