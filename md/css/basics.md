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

