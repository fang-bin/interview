#### html5新特性data_*自定义属性使用

这个自定义data属性的用法非常的简单, 就是你可以往HTML标签上添加任意以 "data-"开头的属性, 这些属性页面上是不显示的,它不会影响到你的页面布局和风格,但它却是可读可写的。

使用data-*可以解决自定义属性混乱无管理的现状。

使用（取值和赋值）主要通过 dataset 进行操作。

```html
<li id="getId" data-id="122" data-vice-id="11">获取id</li>
const getId = document.getElementById('getId');
```

```javascript
// //getAttribute()取值属性
console.log(getId.getAttribute("data-id"));//122
console.log(getId.getAttribute("data-vice-id"));//11
// //setAttribute()赋值属性
getId.setAttribute("data-id","48");
console.log(getId.getAttribute("data-id"));//48

//data-前缀属性可以在JS中通过dataset取值，更加方便
console.log(getId.dataset.id);//112
//data-vice-id连接取值使用驼峰命名法取值 
console.log(getId.dataset.viceId);//11

//赋值
getId.dataset.id = "113";//113
getId.dataset.viceId--;//10

//新增data属性
getId.dataset.id2 = "100";//100

//删除，设置成null，或者delete
getId.dataset.id2 = null;//null
delete getId.dataset.id2;//undefind
```

jq中的方法

```javascript
var id = $("#getId").data("id"); //122
var viceId = $("#getId").data("vice-id"); //11
//赋值
$("#getId").data("id","100");//100

var id = $("#getId").attr("data-id"); //122
var viceId = $("#getId").attr("data-vice-id"); //11
//赋值
$("#getId").attr("data-id","100");//100
```

#### 说说HTML5在标签、属性、存储、API上的新特性

**标签**：新增语义化标签（aside / figure / section / header / footer / nav等），增加多媒体标签video和audio，使得样式和结构更加分离

**属性**：增强表单，主要是增强了input的type属性；meta增加charset以设置字符集；script增加async以异步加载脚本

**存储**：增加localStorage、sessionStorage和indexedDB，引入了application cache对web和应用进行缓存

**API**：增加拖放API、地理定位、SVG绘图、canvas绘图、Web Worker、WebSocket

#### doctype的作用是什么

声明文档类型，告知浏览器用什么文档标准解析这个文档：

* 怪异模式：浏览器使用自己的模式解析文档，不加doctype时默认为怪异模式
* 标准模式：浏览器以W3C的标准解析文档

注意：doctype声明对大小写不敏感。其中HTML5声明为为`<!DOCTYPE html>`

#### href和src有什么区别

href（hyperReference）即超文本引用：当浏览器遇到href时，会并行的地下载资源，不会阻塞页面解析，例如我们使用<link>引入CSS，浏览器会并行地下载CSS而不阻塞页面解析. 因此我们在引入CSS时建议使用<link>而不是@import

```html
<link href="style.css" rel="stylesheet" />
```
复制代码src（resource）即资源，当浏览器遇到src时，会暂停页面解析，直到该资源下载或执行完毕，这也是script标签之所以放底部的原因

```html
<script src="script.js"></script>
```

#### meta有哪些属性，作用是什么
meta标签用于描述网页的元信息，如网站作者、描述、关键词，meta通过name=xxx和content=xxx的形式来定义信息，常用设置如下：

* charset: 定义HTML文档的字符集 `<meta charset="UTF-8" >`
* viewport: 视口，用于控制页面宽高及缩放比例 `<meta name="viewport", content="width=device-width, initial-scale=1, maximum-scale=1">`
* author: 作者
* description: 描述
* keywords: 关键字
* format-detection 格式化
  在浏览器中页面中出现的手机号码将不以拨号的超链接的形式出现: `<meta name="format-detection"content="telephone=no"/>`当你写了一串数字怎么就变成超链接了，点击还能拨打电话呢。因为iPhone会自动把你这个文字加链接样式，通过使用这个标签可以取消这一功能。默认是开启状态。

  不识别页面中出现的邮箱：`<meta name="format-detection"content="email=no"/>`遇上一条同理，告诉设备不识别邮箱，点击之后不自动发送
* http-equiv: 可用于模拟http请求头，可设置过期时间、缓存、刷新 `＜meta http-equiv="expires" content="Wed, 20 Jun 2019 22:33:00 GMT"＞`

针对苹果移动端的meta设置

* apple-mobile-web-app-capable: 是否启用 WebApp 全屏模式，删除苹果默认的工具栏和菜单栏 `<meta name="apple-mobile-web-app-capable"content="yes"/>`
* apple-mobile-web-app-status-bar-style: 设置苹果工具栏颜色 `<meta name="apple-mobile-web-app-status-bar-style" content="black" />`apple-mobile-web-app-status-bar-style的content默认值为default(白色)，可以定为black(黑色)和black-translucent(灰色半透明)。若值为“black-translucent”将会占据页面px位置，浮在页面上方（会覆盖页面20px高度–iphone4和itouch4的Retina屏幕为40px）
* apple-touch-fullscreen: 如果把一个web app添加到了主屏幕中，那么从主屏幕中打开这个web app则全屏显示 `<meta name="apple-touch-fullscreen" content="yes" />`

着重介绍一下**viewport**和**http-equive**

**viewport**
* width/height，宽高，默认宽度980px
* initial-scale，初始缩放比例，1~10
* maximum-scale/minimum-scale，允许用户缩放的最大/小比例
* user-scalable，用户是否可以缩放 (yes/no)

**http-equive**
* expires，指定过期时间
* progma，设置no-cache可以禁止缓存
* refresh，定时刷新
* set-cookie，可以设置cookie
* X-UA-Compatible，使用浏览器版本
* apple-mobile-web-app-status-bar-style，针对WebApp全屏模式，隐藏状态栏/设置状态栏颜色
