# 设计模式

## 观察者模式(订阅-发布者模式)
观察者模式实现：

1. 消息容器
2. 订阅消息方法
3. 取消订阅的消息方法
4. 发送订阅的消息方法

观察者模式主要用来解决模块间通信问题，这是模块间解耦的一种可行方案。

观察者模式最主要的作用是解决类或对象之间的耦合，解耦两个相互依赖的对象，使其依赖于观察者的消息机制。

node.js中的event模块就是观察者模式的应用

```javascript
const Observer = (function (){
  let _event = {};
  return {
    register: function (type, fn){
      if (typeof _event[type] === 'undefined'){
        _event[type] = [fn];
      }else {
        _event[type].push(fn);
      }
      return this;
    },
    remove: function (type, fn){
      if (typeof _event[type] === 'undefined') return;
      const index = _event[type].findIndex(e => e === fn);
      _event[type].splice(index, 1);
      return this;
    },
    fire: function (type, ...args){
      if (typeof _event[type] === 'undefined') return;
      const len = _event[type].length;
      let i = 0;
      for (; i < len; i++){
        let fn = _event[type][i];
        typeof fn === 'function' && fn.apply(this, args)
      }
      return this;
    }
  }
})();

const yyy = (y) => {
  console.log(`你好 ${y}`);
}
Observer.register('fangbin', (x) => {
  console.log(`方斌 ${x}`);
  Observer.fire('ha', x);
}).register('ha', yyy);

setTimeout(() => {
  Observer.fire('fangbin', '呀呀呀呀').remove('ha', yyy);
  setTimeout(() => {
    Observer.fire('fangbin', 'ffff')
  }, 1000);
}, 2000);
```


###### 订阅者模式也是自定义事件，它和DOM编程中的事件有什么区别
主要区别就是，订阅者模式的事件是手动触发的，而DOM编程的中的事件是自动触发的。
还有就是订阅者模式的事件可以自定义，而DOM编程中的事件是固定的几个(click, contextmenu, dbclick, mousedown, mouseup, mouseenter, mouseleave, mousemove, mouseout, mouseover, touch)
