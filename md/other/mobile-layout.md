# 移动端适配布局

* 设备物理分辨率（物理像素）：实际分辨率 px（在同一个设备上，它的物理像素个数是固定的，这是厂商在出厂时就设置好了的————即一个设备的分辨率）
* 设备逻辑分辨率（逻辑像素）：肉眼实际感知的分辨率 pt/dp
* 设备像素比 dpr： 物理像素和设备独立像素的比值。

获取设备物理像素、逻辑像素和设备像素比的方法：

* 逻辑分辨率宽：window.screen.width
* 逻辑分辨率高：window.screen.height
* 设备像素比：window.devicePixelRatio
* 物理分辨率宽： window.screen.width * window.devicePixelRatio
* 物理分辨率高： window.screen.heiht * window.devicePixelRatio

#### 窗口适配

```html
<meta name="viewport" content="width=device-width; initial-scale=1; maximum-scale=1; minimum-scale=1; user-scalable=no;">
```

![avatar](https://user-gold-cdn.xitu.io/2020/3/17/170e782c3e72b843?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

#### rem适配
rem是CSS3新增的一个相对单位，这个单位引起了广泛关注。这个单位与em有什么区别呢？区别在于使用rem为元素设定字体大小时，仍然是相对大小，但相对的只是HTML根元素。这个单位可谓集相对大小和绝对大小的优点于一身，通过它既可以做到只修改根元素就成比例地调整所有字体大小，又可以避免字体大小逐层复合的连锁反应。目前，除了IE8及更早版本外，所有浏览器均已支持rem。对于不支持它的浏览器，应对方法也很简单，就是多写一个绝对单位的声明。这些浏览器会忽略用rem设定的字体大小

**flexible方案**是阿里早期开源的一个移动端适配解决方案。

```javascript
function setRemUnit () {
    var rem = docEl.clientWidth / 10
    docEl.style.fontSize = rem + 'px'
}
setRemUnit();
```
因为当年viewport在低版本安卓设备上还有兼容问题，而vw，vh还没能实现所有浏览器兼容，所以flexible方案用rem来模拟vmin来实现在不同设备等比缩放的“通用”方案，之所以说是通用方案,是因为他这个方案是根据设备大小去判断页面的展示空间大小即屏幕大小，然后根据屏幕大小去百分百还原设计稿，从而让人看到的效果(展示范围)是一样的，这样一来，苹果5 和苹果6p屏幕如果你按照设计稿还原的话，字体大小实际上不一样，而人们在一样的距离上希望看到的大小其实是一样的，本质上，**用户使用更大的屏幕，是想看到更多的内容，而不是更大的字**。

所以，这个用缩放来解决问题的方案是个过渡方案，注定时代所淘汰。

#### vw vh布局(现在主流大厂大部分采用这种方案)

vh、vw方案即将视觉视口宽度 window.innerWidth和视觉视口高度 window.innerHeight 等分为 100 份。

![avatar](https://user-gold-cdn.xitu.io/2020/3/17/170e82463b522ff6?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

vh和vw方案和rem类似也是相当麻烦需要做单位转化，而且px转换成vw不一定能完全整除，因此有一定的像素差。

不过在工程化的今天，webpack解析css 的时候用postcss-loader 有个postcss-px-to-viewport能自动实现px到vw的转化

```javascript
{
    loader: 'postcss-loader',
    options: {
    	plugins: ()=>[
        	require('autoprefixer')({
            browsers: ['last 5 versions'],
            // browsers: ['ios >= 8', 'Android >= 4.0'],
        	}),
        	require('postcss-px-to-viewport')({
        		viewportWidth: 375, //视口宽度（数字)
        		viewportHeight: 1334, //视口高度（数字）
        		unitPrecision: 3, //设置的保留小数位数（数字），5
        		viewportUnit: 'vw', //设置要转换的单位（字符串）
        		selectorBlackList: ['.ignore', '.hairlines'], //不需要进行转换的类名（数组）
                minPixelValue: 1, //设置要替换的最小像素值（数字）
                mediaQuery: false //允许在媒体查询中转换px（true/false）
        	})
    	]
}
```

#### px为主，vx和vxxx（vw/vh/vmax/vmin）为辅，搭配一些flex（推荐）

**用户之所以去买大屏手机，不是为了看到更大的字，而是为了看到更多的内容**

#### 1px边框问题

当我们css里写的1px的时候，由于它是逻辑像素，导致我们的逻辑像素根据这个设备像素比（dpr）去映射到设备上就为2px，或者3px，由于每个设备的屏幕尺寸不一样，就导致每个物理像素渲染出来的大小也不同（记得上面的知识点吗，设备的像素大小是不固定的），这样如果在尺寸比较大的设备上，1px渲染出来的样子相当的粗矿，这就是经典的一像素边框问题

**解决**

核心思路，就是
在web中，浏览器为我们提供了window.devicePixelRatio来帮助我们获取dpr。在css中，可以使用媒体查询min-device-pixel-ratio，区分dpr：
我们根据这个像素比，来算出他对应应该有的大小,但是暴露个非常大的兼容问题

![avatar](https://user-gold-cdn.xitu.io/2020/3/17/170e773ea67b01b4?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

其中Chrome把0.5px四舍五入变成了1px，而firefox/safari能够画出半个像素的边，并且Chrome会把小于0.5px的当成0，而Firefox会把不小于0.55px当成1px，Safari是把不小于0.75px当成1px，进一步在手机上观察iOS的Chrome会画出0.5px的边，而安卓(5.0)原生浏览器是不行的。所以直接设置0.5px不同浏览器的差异比较大，并且我们看到不同系统的不同浏览器对小数点的px有不同的处理。所以如果我们把单位设置成小数的px包括宽高等，其实不太可靠，因为不同浏览器表现不一样。

**transform: scale(0.5) 方案**
```css
div {
    height: 1px;
    background: #000;
    -webkit-transform: scaleY(0.5);
    -webkit-transform-origin: 0 0;
    overflow: hidden;
}
```

**css根据设备像素比媒体查询后的解决方案**
```css
/* 2倍屏 */
@media only screen and (-webkit-min-device-pixel-ratio: 2.0) {
    .border-bottom::after {
        -webkit-transform: scaleY(0.5);
        transform: scaleY(0.5);
    }
}

/* 3倍屏 */
@media only screen and (-webkit-min-device-pixel-ratio: 3.0) {
    .border-bottom::after {
        -webkit-transform: scaleY(0.33);
        transform: scaleY(0.33);
    }
}
```

