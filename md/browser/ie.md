### IE的haslayout

hasLayout可以简单看作是IE5.5/6/7中的BFC(Block Formatting Context)。也就是一个元素要么自己对自身内容进行组织和尺寸计算(即可通过width/height来设置自身的宽高)，要么由其containing block来组织和尺寸计算。而IFC（即没有拥有布局）而言，则是元素无法对自身内容进行组织和尺寸计算，而是由自身内容来决定其尺寸（即仅能通过line-height设置内容行距，通过行距来支撑元素的高度；也无法通过width设置元素宽度，仅能由内容来决定而已）

当hasLayout为true时(就是所谓的"拥有布局")，相当于元素产生新BFC，元素自己对自身内容进行组织和尺寸计算;
当hasLayout为false时(就是所谓的"不拥有布局")，相当于元素不产生新BFC，元素由其所属的containing block进行组织和尺寸计算。
和产生新BFC的特性一样，hasLayout无法通过CSS属性直接设置，而是通过某些CSS属性间接开启这一特性。不同的是某些CSS属性是以不可逆方式间接开启hasLayout为true。并且默认产生新BFC的只有html元素，而默认hasLayout为true的元素就不只有html元素了。

默认hasLayout==true的元素：`<html>, <body>，<table>, <tr>, <th>, <td>，<img>,<hr>,<input>, <button>, <select>, <textarea>, <fieldset>, <legend>,<iframe>, <embed>, <object>, <applet>,<marquee>`

触发hasLayout==true的方式:

```css
display: inline-block
height: (除 auto 外任何值)
width: (除 auto 外任何值)
float: (left 或 right)
position: absolute
writing-mode: tb-rl
zoom: (除 normal 外任意值)(这可以也可以理解clearfix清楚浮动加的IE hack方法)
```

