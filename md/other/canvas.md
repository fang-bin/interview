#### canvas绘制黑屏解决方案

###### hybird开发下的黑屏

某些安卓手机的WebView画布元素会呈现出黑屏，不论计算量大小，是否有动画，是否有触控事件，甚至可以认为所有安卓手机都有可能出现黑屏，解决方案不在H5前端，而在客户端。安卓某些版本的webview不支持canvas硬件加速，所以在很多情况下看到动画偶现黑屏。native增加了配置：

```javascript
  webView.setLayerType(View.LAYER_TYPE_SOFTWARE,null);
````

禁止canvas硬件加速，从而避免了类似黑屏、闪烁、动画停滞、跳帧和擦除不全一类的问题。取消硬件加速后，动画会更加平顺，但是整体速度会变慢，这个速度的快慢程度取决于手机的计算能力。所以在安卓手机上要想达到统一的动画效果是不可能的。所有安卓手机类似于动画黑屏闪烁的问题都是这个原因。

###### HTML5-canvas绘图黑屏
移动设备上canvas呈现出黑屏很大一部分原因就是绘图的计算量太大，导致设备显示出现问题。那么最直接的解决方案就是减少计算量（或者称为优化）。

正常情况下，为页面添加一个canvas元素，并且一边算各种图形坐标，一边一点点的绘制上去。在这个过程当中我们就损耗了相当大的性能，每执行一次context.fill()或者context.stroke()，图形都会被渲染一次。

**解决方案：双缓冲绘图机制**
为页面添加一个canvas元素，同时创建另一个一个一模一样的canvas元素，但是不添加到页面dom树（也就是存在内存中），这时我们在内存中的canvas元素上一边算图形坐标一边绘制，当绘制完成时直接调用canvas的复制方法（context.drawImage）一次性将“草稿画布”上的图形绘制到真正的页面canvas元素上，通过这样的技巧，我们将会节省大量页面绘制的性能损耗，不但能解决黑屏的问题，还能缩短图形绘制的时间。

```javascript
//获取页面中的canvas画布容器，通常为一个div
var container=document.getElementById("container");
//创建一个真实的画布，他将呈现给用户
var realCanvas=document.createElement("canvas");
//设置realCanvas的width/height属性，乘以二以防止像素点模糊问题
realCanvas.width=container.clientWidth*2;
realCanvas.height=container.clientHeight*2;
//设置realCanvas的样式width/height属性，填充满父元素
realCanvas.style.width=container.clientWidth+"px";
realCanvas.style.height=container.clientHeight+"px";
container.appendChild(realCanvas);
//创建一个虚拟节点cacheCanvas，我们不会将他添加到页面上
var cacheCanvas=document.createElement("canvas");
//设置cacheCanvas的width/height属性，和realCanvas的完全相同
cacheCanvas.width=container.clientWidth*2;
cacheCanvas.height=container.clientHeight*2;
//开始绘制，获取真实节点和虚拟节点的上下文context
var realContext=realCanvas.getContext("2d");
var cacheContext=cacheCanvas.getContext("2d");
/*
* 这次试用双缓冲技术，我们在cacheCanvas中绘制，
* 但此时什么都看不到，因为cacheCanvas没有添加到页面上
*/
cacheContext.fillRect(0,0,100,100);
/*
* 在缓冲区完成绘制之后，我们将缓冲区的内容一次性绘制到页面上
*/
realContext.drawImage(cacheCanvas, 0, 0);
```

###### HTML5-canvas绘图白屏

白屏产生可能原因：

* 绘图代码报错，导致绘图不执行，因此出现白屏
* 计算图形坐标的时候算错，比如计算出无穷大或者undefined或者超出画布边界的坐标值，导致你画的图看不见（HTML5-canvasAPI在执行绘图方法时传入的数据异常时不会抛出任何错误），也是白屏。
* 没有给画布canvas设置width或height或值为0，比如画布初始化的时候页面还没有撑开，导致画布canvas元素的高度或者宽度为零，也会出现白屏。
* canvas动画白屏闪烁问题（主要讲这个）

canvas动画白屏闪烁问题的原因：我们绘制动画，一般都是先将上一帧绘制的图像擦掉（清除画布），然后重新绘制，由于间隙时间非常小，给用户的感觉就很流畅，但是，一旦绘制的动画图形很复杂时，擦除画布之后，绘制新图形之间这个过程显示的是空白，由于某些设备的计算能力不足，就会看到很明显的canvas动画白屏闪烁。

此种情况也可以通过双缓冲绘图机制解决，将“重新计算坐标点”以及“重新绘制”两个耗时操作移到了虚拟画布上（也就是双缓冲机制），而在最后才通过realContext.drawImage()的方式一次性将虚拟画布上的图形呈现到页面上（这个操作通常小于1ms）。通过这种优化手段，即使再复杂的图形绘制，页面上也不会再看到动画帧之间的白屏问题，耗时操作都在虚拟节点上执行，真正的画布只会在虚拟画布准备好图形之后才擦除+重绘。







