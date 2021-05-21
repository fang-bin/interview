## Web Components

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