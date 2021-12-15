### 1. `content-visibility: hidden;` `display:none;` `visibility: hidden`的区别
* `display: none`：隐藏元素并破坏其渲染状态。这意味着取消隐藏元素与渲染具有相同内容的新元素一样昂贵。渲染树中不包括 `display: none` 的元素。

* `visibility: hidden`：隐藏元素并保持其渲染状态。这并不能真正从文档中删除该元素，因为它（及其子树）仍占据页面上的几何空间，并且仍然可以单击。它也可以在需要时随时更新渲染状态，即使隐藏也是如此。其元素仍然存在于渲染树中，该属性为继承属性。

* `content-visibility: hidden`：隐藏元素并保留其渲染状态。这意味着该元素隐藏时行为和`display: none`一样，但再次显示它的成本要低得多。content-visibility可跳过不在屏幕上的内容渲染，包括布局和渲染，直到真正需要布局渲染的时候为止。不过由于其没有其他设置高度为0，可能会造成滚动问题。