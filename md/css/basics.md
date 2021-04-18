### z-index堆叠规则（z-index数值大的元素就一定会覆盖数值小的元素么？）

[层叠上下文详解](https://github.com/fang-bin/interview/blob/master/md/day/index-css.md#28-%E5%B1%82%E5%8F%A0%E4%B8%8A%E4%B8%8B%E6%96%87)

### BFC（块级格式化上下文）

[块级格式化上下文详解](https://github.com/fang-bin/interview/blob/master/md/day/index-css.md#27-%E6%A0%BC%E5%BC%8F%E5%8C%96%E4%B8%8A%E4%B8%8B%E6%96%87)

自适应两栏布局

<style>
#layout {
    background: red;
}
#layout .left {
    float: left;
    width: 100px;
    height: 100px;
    background: pink;
}
#layout .right {
    height: 110px;
    background: #ccc;
    overflow: auto;
}
</style>
<div id="layout">
  <!--左边宽度固定，右边自适应-->
  <div class="left">左</div>
  <div class="right">右</div>
</div>

```html
<section id="layout">
  <style>
    #layout {
      background: red;
    }
    #layout .left {
      float: left;
      width: 100px;
      height: 100px;
      background: pink;
    }
    #layout .right {
      height: 110px;
      background: #ccc;
      overflow: auto;
    }
  </style>
  <!--左边宽度固定，右边自适应-->
  <div class="left">左</div>
  <div class="right">右</div>
</section>
```

原理是BFC不会与float元素发生重叠。

**清除浮动的方法**
清除浮动是为了解决子元素浮动而导致父元素高度塌陷的问题（BFC）

1. 最下方添加空元素 `clear: both;`
2. 使用伪元素
    ```javascript
    /* 对父元素添加伪元素 */
    .parent::after{
      content: "";
      display: block;
      height: 0;
      clear:both;
    }
    ```
3. 触发父元素BFC
    ```javascript
    /* 触发父元素BFC */
    .parent {
      overflow: hidden;
      /* float: left; */
      /* position: absolute; */
      /* display: inline-block */
      /* 以上属性均可触发BFC */
    }
    ```

### Flex布局
flex布局很容易理解，我着重记录一下自己经常容易忘记和混淆的

项目属性（使用在容器内子元素上的属性）：

* **flex-grow**
  定义项目的放大比例，默认为0，即使有剩余空间也不放大。如果所有子元素flex-grow为1，那么将等分剩余空间，如果某个子元素flex-grow为2，那么这个子元素将占据2倍的剩余空间
* **flex-shrink**
  定义项目的缩小比例，默认为1，即如果空间不足，子元素将缩小。如果所有子元素flex-shrink都为1，某个子元素flex-shrink为0，那么该子元素将不缩小

### 盒模型
css盒模型主要由外边距（margin）、边框（border）、内边距（padding）、内容（content）组成。

盒模型分为标准盒模型（w3c标准）和怪异盒模型(ie)

* 标准盒模型 width = content
  盒子的实际宽度等于 width + padding + border + margin
  `box-sizing: content-box;`
* 怪异盒模型 width = content + padding + border
  盒子的实际宽度等于 width + margin
  `box-sizing: border-box;`

#### 垂直居中
略

### 常见布局

###### 两边定宽，中间自适应

```html
<style>
  .wrap{
    width: 100%;
    height: 200px;
  }
  .left{
    width: 100px;
    height: 200px;
    background-color: #ccc;
    float: left;
  }
  .right{
    width: 100px;
    height: 200px;
    background-color: #999;
    float: right;
  }
  .mid{
    background-color: #f00;
    width: 100%;
    height: 200px;
  }
  *{
    padding: 0;
    margin: 0;
  }
</style>
<div class="wrap">
  <div class="left">xxx</div>
  <div class="right">ddd</div>
  <div class="mid">yyy</div>
</div>
```

这个就是利用的浮动盒不会叠加到BFC上的特性

等等其他方法（注意这些方法有一些极端情况，比如说中间元素小于两边元素宽度，或者屏幕宽度过小等等，要尽量设置窗口的最小宽度）:
* 同行排列之后，左右固定宽度，中间100%宽度，然后设置中间盒模型为标准盒模型，设置padding值分别为左右的宽度
* 三者皆浮动的情况下，将中间元素放在第一个，然后第二三个元素分别设置`margin-left: -100%`、`margin-left: -(自身宽度)`，这个则是利用**margin/padding取百分比的值时，都是基于父元素的宽度的。**

等其他方法（例如table、grid或者flex）

###### 垂直布局，父元素高度固定，一个子元素高度固定，另一个子元素高度自适应

1. 高度自适应元素可以通过同时设置top,bottom（因为一个元素高度固定，所以top，bottom中一个必定固定的，另一个设置为0）可以强制定义盒模型的区域
2. 定位之后，一个元素固定高度，另一个高度100%后，设置padding
3. 其他方法如flex, grid等


#### 为背景图实现的展位图设置占位高度

可以根据 **margin/padding取百分比的值时，都是基于父元素的宽度** 的特性，来设置背景图padding值等于（图片高度/图片宽度）基于宽度的百分比来实现。

注意: **定位元素 left/right 和 top/bottom 取百分比的值时，则是分别按照父元素的宽高来计算的**

#### 浏览器是怎样解析CSS选择器的？

CSS选择器的解析是从右向左解析的。若从左向右的匹配，发现不符合规则，需要进行回溯，会损失很多性能。若从右向左匹配，先找到所有的最右节点，对于每一个节点，向上寻找其父节点直到找到根元素或满足条件的匹配规则，则结束这个分支的遍历。两种匹配规则的性能差别很大，是因为从右向左的匹配在第一步就筛选掉了大量的不符合条件的最右节点（叶子节点），而从左向右的匹配规则的性能都浪费在了失败的查找上面。

#### 在网页中的应该使用奇数还是偶数的字体？为什么呢？
使用偶数字体。偶数字号相对更容易和 web 设计的其他部分构成比例关系。Windows 自带的点阵宋体（中易宋体）从 Vista 开始只提供 12、14、16 px 这三个大小的点阵，而 13、15、17 px时用的是小一号的点。（即每个字占的空间大了 1 px，但点阵没变），于是略显稀疏。

#### 什么是响应式设计？响应式设计的基本原理是什么？如何兼容低版本的IE？

响应式网站设计(Responsive Web design)是一个网站能够兼容多个终端，而不是为每一个终端做一个特定的版本。
基本原理是通过媒体查询检测不同的设备屏幕尺寸做处理。
页面头部必须有meta声明的viewport。

```html
<meta name="’viewport’" content="”width=device-width," initial-scale="1." maximum-scale="1,user-scalable=no”"/>
```

#### ::before 和 :after中双冒号和单冒号有什么区别？解释一下这2个伪元素的作用

* 单冒号(:)用于CSS3伪类，双冒号(::)用于CSS3伪元素。

* ::before就是以一个子元素的存在，定义在元素主体内容之前的一个伪元素。并不存在于dom之中，只存在在页面之中。

:before 和 :after 这两个伪元素，是在CSS2.1里新出现的。起初，伪元素的前缀使用的是单冒号语法，但随着Web的进化，在CSS3的规范里，伪元素的语法被修改成使用双冒号，成为::before ::after

#### 伪类和伪元素
* 伪类： 用来选择那些不能够被普通选择器选择的文档之外的元素，比如:hover
* 伪元素： 创建通常不存在于文档中的元素，比如::before

#### li与li之间有看不见的空白间隔是什么原因引起的？有什么解决办法？

行框的排列会受到中间空白（回车空格）等的影响，因为空格也属于字符,这些空白也会被应用样式，占据空间，所以会有间隔，把字符大小设为0，就没有空格了。

解决方法：

1. 可以将<li>代码全部写在一排

2. 浮动li中float：left

3. 在ul中用font-size：0（谷歌不支持）；可以使用letter-space：-3px

#### png、jpg、gif 这些图片格式解释一下，分别什么时候用。有没有了解过webp？

1. png是便携式网络图片（Portable Network Graphics）是一种无损数据压缩位图文件格式.优点是：压缩比高，色彩好。 大多数地方都可以用。

2. jpg是一种针对相片使用的一种失真压缩方法，是一种破坏性的压缩，在色调及颜色平滑变化做的不错。在www上，被用来储存和传输照片的格式。

3. gif是一种位图文件格式，以8位色重现真色彩的图像。可以实现动画效果.

4. webp格式是谷歌在2010年推出的图片格式，压缩率只有jpg的2/3，大小比png小了45%。缺点是压缩的时间更久了，兼容性不好，目前谷歌和opera支持。

#### style标签写在body后与body前有什么区别？

页面加载自上而下 当然是先加载样式。
写在body标签后由于浏览器以逐行方式对HTML文档进行解析，当解析到写在尾部的样式表（外联或写在style标签）会导致浏览器停止之前的渲染，等待加载且解析样式表完成之后重新渲染，在windows的IE下可能会出现FOUC现象（即样式失效导致的页面闪烁问题）