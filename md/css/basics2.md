### 1. point-events

pointer-events CSS 属性指定在什么情况下 (如果有) 某个特定的图形元素可以成为鼠标事件的 target

| 可选值 | 含义 |
| --- | --- |
| auto | 与pointer-events属性未指定时的表现效果相同，对于SVG内容，该值与visiblePainted效果相同 |
| none | 元素永远不会成为鼠标事件的target。但是，当其后代元素的pointer-events属性指定其他值时，鼠标事件可以指向后代元素，在这种情况下，鼠标事件将在捕获或冒泡阶段触发父元素的事件侦听器。|
| visiblePainted | 只适用于SVG。元素只有在以下情况才会成为鼠标事件的目标： <br /> visibility属性值为visible，且鼠标指针在元素内部，且fill属性指定了none之外的值 <br/> visibility属性值为visible，鼠标指针在元素边界上，且stroke属性指定了none之外的值 |
| ...其他都是针对SVG | |

##### `pointer-events: none`的用处

元素应用了该 CSS 属性，链接啊，点击啊什么的都不管用了。作用是让元素实体 “虚化”。

例如一个应用 pointer-events: none 的按钮元素，则我们在页面上看到的这个按钮，只是一个虚幻的影子而已，您可以理解为海市蜃楼，幽灵的躯体。当我们用手触碰它的时候可以轻易地没有任何感觉地从中穿过去。

主要用于需要穿透点击的时候

### 2.