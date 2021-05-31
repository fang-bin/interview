# Web Components

```html
<user-card src="https://semantic-ui.com/images/avatar2/large/kristy.png" name="方斌" age="28">
  <p class="email" slot="email">123@qq.com</p>
</user-card>
<template id="userCardTemplate">
  <style>
    .wrap{
      width: 500px;
      height: 300px;
      background-color: #ccc;
      border-radius: 24px;
      display: flex;
      justify-content: space-between;
      align-items: center;
    }
    .img{
      width: 200px;
      height: 200px;
      border-radius: 12px;
      margin-left: 25px;
    }
    .info{
      width: 200px;
      height: 500px;
      margin-right: 25px;
      display: flex;
      flex-direction: column;
      justify-content: center;
      align-items: center;
    }
    .name,.age{
      color: #000;
      font-size: 30px;
      height: 45px;
      margin-bottom: 10px;
      text-align: center;
    }
  </style>
  <div class="wrap">
    <img class="img" />
    <div class="info">
      <p class="name"></p>
      <p class="age"></p>
      <slot name="email"></slot>
    </div>
  </div>
</template>
```

```javascript
class UserCard extends HTMLElement {
  constructor() {
    super();
    const shadow = this.attachShadow({mode: 'closed'});
    const userCardTem = document.querySelector('#userCardTemplate');
    const content = userCardTem.content.cloneNode(true);
    content.querySelector('.img').setAttribute('src', this.getAttribute('src'));
    content.querySelector('.name').innerText = this.getAttribute('name');
    content.querySelector('.age').innerText = this.getAttribute('age');
    content.querySelector('.img').addEventListener('click', this.toConsole);
    shadow.appendChild(content);
  }
  toConsole() {
    console.log('hahah');
  }
}

window.customElements.define('user-card', UserCard);
```

## Shadow DOM

Web components 的一个重要属性是**封装**——可以**将标记结构、样式和行为隐藏起来，并与页面上的其他代码相隔离，保证不同的部分不会混在一起，可使代码更加干净、整洁**。其中，Shadow DOM 接口是关键所在，它可以将一个隐藏的、独立的 DOM 附加到一个元素上。

以上面为例，不希望用户能够看到`<user-card>`的内部代码，Web Component 允许内部代码隐藏起来，这叫做 Shadow DOM，即这部分 DOM 默认与外部 DOM 隔离，内部任何代码都无法影响外部。

`Element.attachShadow()` 方法给指定的元素挂载一个Shadow DOM，并且返回对 ShadowRoot 的引用。

自定义元素的`this.attachShadow()`方法开启 Shadow DOM。

`this.attachShadow()`方法的参数`{ mode: 'closed' }`，表示 Shadow DOM 是封闭的，不允许外部访问。

**Shadow DOM 允许将隐藏的 DOM 树附加到常规的 DOM 树中**

##### Shadow DOM 和 Virtual DOM

###### Shadow DOM
Shadow DOM是浏览器提供的一个可以允许将隐藏的DOM树添加到常规的DOM树中——它以shadow root为起始根节点，在这个根节点的下方，可以是任意元素，和普通的DOM元素一样。

###### Virtual DOM
虚拟DOM是由js实现的避免DOM树频繁更新，通过js的对象模拟DOM中的节点，然后通过特定的render方法将它渲染成真实的节点，数据更新时，渲染得到新的 Virtual DOM，与上一次得到的 Virtual DOM 进行 diff，得到所有需要在 DOM 上进行的变更，然后在 patch 过程中应用到 DOM 上实现UI的同步更新