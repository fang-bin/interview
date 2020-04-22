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