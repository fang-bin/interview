### z-index堆叠规则（z-index数值大的元素就一定会覆盖数值小的元素么？）

z-index用来控制元素重叠时堆叠顺序，已经定位的元素（即position值不为static）;

stacking context: 翻译就是“堆叠上下文”。每个元素仅属于一个堆叠上下文，元素的z-index描述元素在相同堆叠上下文中“z轴”的呈现顺序。(非相同堆叠上下文中的元素，z-index没有可比性)

生成堆叠上下文的规则：

* 根元素（即HTML元素）
* 已定位元素（即绝对定位或相对定位）并且z-index不是默认的auto。
* 元素opacity属性不为1(See the specification for opacity)
* 元素transform不为none
* 元素min-blend-mode不为normal
* 元素filter属性不为none

子元素的z-index值只在父元素范围内有效。子堆叠上下文被看做是父堆叠上下文中一个独立的模块，相邻的堆叠上下文完全没关系。

### BFC（块级格式化上下文）
主要问及的问题是浮动坍塌和margin重叠等

BFC的全称为Block Formatting Context，即块级格式化上下文。一个BFC有如下特性：

* 处于同一个BFC中的元素相互影响，可能会发生margin collapse；
* BFC在页面上是一个独立的容器，容器里面的子元素不会影响到外面的元素，反之亦然；
* 计算BFC的高度时，考虑BFC所包含的所有元素，包括浮动元素也参与计算；
* 浮动盒的区域不会叠加到BFC上；

创建BFC的方法如下
* 浮动（float的值不为none）；
* 绝对定位元素（position的值为absolute或fixed）；
* 行内块（display为inline-block）
* 表格单元（display为table、table-cell、table-caption等HTML表格相关属性）；
* 弹性盒（display为flex或inline-flex）；
* overflow不为visible；

BFC的使用场景：

1. 防止垂直margin重叠，父子元素的边界重叠，在父元素上加上overflow:hidden;使其成为BFC。

2. 清除内部浮动对元素高度的影响。父元素#float的高度为0，解决方案为为父元素#float创建BFC，这样浮动子元素的高度也会参与到父元素的高度计算。

自适应两栏布局
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
* 三者皆浮动的情况下，将中间元素放在第一个，然后第二三个元素分别设置`margin-left: -100%`、`margin-left: -(自身宽度)`，这个则是利用**margin/padding取百分比的值时，无论是 left/right 还是 top/bottom，都是基于父元素的宽度的。**

等其他方法（例如table、grid或者flex）

###### 垂直布局，父元素高度固定，一个子元素高度固定，另一个子元素高度自适应

1. 高度自适应元素可以通过同时设置top,bottom（因为一个元素高度固定，所以top，bottom中一个必定固定的，另一个设置为0）可以强制定义盒模型的区域
2. 定位之后，一个元素固定高度，另一个高度100%后，设置padding
3. 其他方法如flex, grid等


#### 为背景图实现的展位图设置占位高度

可以根据**margin/padding取百分比的值时，无论是 left/right 还是 top/bottom，都是基于父元素的宽度的。**特性，来设置背景图padding值等于（图片高度/图片宽度）基于宽度的百分比来实现。






