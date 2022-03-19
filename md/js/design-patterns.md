# 设计模式

设计模式可以说是在面向对象软件设计过程中针对特定问题的简洁而优雅的解决方案。

* 单例模式
* 工厂模式
* 适配器模式
* 观察者模式/发布-订阅模式

## 单例模式

保证一个类仅有一个实例，并提供访问此实例的全局访问点。

##### 用于解决

一个全局使用的类频繁地创建与销毁

```javascript
const Singleton = function() {};
Singleton.getInstance = (function() {
  // 由于es6没有静态类型,故闭包: 函数外部无法访问 instance
  let instance = null;
  return function(...args) {
    // 检查是否存在实例
    if (!instance) {
      instance = new Singleton(...args);
    }
    return instance;
  };
})();
let s1 = Singleton.getInstance();
let s2 = Singleton.getInstance();
console.log(s1 === s2);
```

## 工厂模式

就是把`new`对象的操作包裹一层，对外提供一个可以根据不同参数创建不同对象的函数。

##### 优点

优点显而易见，可以隐藏原始类，方便之后的代码迁移。调用者只需要记住类的代名词即可。

##### 缺点

由于多了层封装，会造成类的数目过多，系统复杂度增加。

## 适配器模式

适配器模式（Adapter）是将一个类（对象）的接口（方法或属性）转化成客户希望的另外一个接口（方法或属性），适配器模式使得原本由于接口不兼容而不能一起工作的那些类（对象）可以一些工作。

##### 桥接模式和其他模式的区别

1. 适配器和桥接模式虽然类似，但桥接的出发点不同，**桥接的目的是将接口部分和实现部分分离**，从而对他们可以更为容易也相对独立的加以改变。而**适配器则意味着改变一个已有对象的接口**。

2. 装饰者模式增强了其它对象的功能而同时又不改变它的接口，因此它对应程序的透明性比适配器要好，其结果是装饰者支持递归组合，而纯粹使用适配器则是不可能的。

3. 代理模式在不改变它的接口的条件下，为另外一个对象定义了一个代理。

## 桥接模式

桥接模式（Bridge）将抽象部分与它的实现部分分离，使它们都可以独立地变化。
其实就是函数的封装，比如要对某个DOM元素添加color和backgroundColor，可以封装个changeColor函数。

实现:

```javascript
class Speed {            // 运动模块
  constructor(x, y) {
    this.x = x
    this.y = y
  }
  run() {  console.log(`运动起来 ${this.x} + ${this.y}`)  }
}

class Color {            // 着色模块
  constructor(cl) {
    this.color = cl
  }
  draw() {  console.log(`绘制颜色 ${this.color}`)  }
}

class Speak {
  constructor(wd) {
    this.word = wd
  }
  say() {  console.log(`说话 ${this.word}`)  }
}

class Ball {                     // 创建球类，可以着色和运动
  constructor(x, y, cl) {
    this.speed = new Speed(x, y)
    this.color = new Color(cl)
  }
  init() {
    this.speed.run()
    this.color.draw()
  }
}

class Man {                    // 人类，可以运动和说话
  constructor(x, y, wd) {
    this.speed = new Speed(x, y)
    this.speak = new Speak(wd)
  }
  init() {
    this.speed.run()
    this.speak.say()
  }
}

const man = new Man(1, 2, 'hehe?')
man.init()                                // 运动起来 1 + 2      说话 hehe?
```

###### 优点：
* 分离接口和实现部分
* 提高可扩充性
* 对客户隐藏实现细节

###### 缺点
* 大量的类将导致开发成本的增加，同时在性能方面可能也会有所减少。


## 发布-订阅模式、观察者模式

观察者模式定义了对象间的一种一对多的依赖关系，当一个对象的状态发生改变时，所有依赖于它的对象都将得到通知，并自动更新。观察者模式属于行为型模式，行为型模式关注的是对象之间的通讯，观察者模式就是观察者和被观察者之间的通讯。

观察者模式 有一个别名叫 发布-订阅模式。

但是随着时间沉淀，发布-订阅模式已经独立于观察者模式，成为另外一种不同的设计模式。

在现在的发布订阅模式中，称为发布者的消息发送者不会将消息直接发送给订阅者，这意味着发布者和订阅者不知道彼此的存在。在发布者和订阅者之间存在第三个组件，称为调度中心或事件通道，它维持着发布者和订阅者之间的联系，过滤所有发布者传入的消息并相应地分发它们给订阅者。

举一个例子，你在微博上关注了A，同时其他很多人也关注了A，那么当A发布动态的时候，微博就会为你们推送这条动态。A就是发布者，你是订阅者，微博就是调度中心，你和A是没有直接的消息往来的，全是通过微博来协调的（你的关注，A的发布动态）。

#### 两种模式的区别

![Observer-pattern-and-Publish–subscribe-pattern](https://github.com/fang-bin/interview/blob/master/image/Observer-pattern-and-Publish–subscribe-pattern.png)

![Observer-pattern-and-Publish–subscribe-pattern](https://github.com/fang-bin/interview/blob/master/image/Observer-pattern-and-Publish–subscribe-pattern.jpeg)

可以看出，**发布订阅模式相比观察者模式多了个事件通道（也可以称之代理人(Broker)），事件通道作为调度中心，管理事件的订阅和发布工作，彻底隔绝了订阅者和发布者的依赖关系**。即订阅者在订阅事件的时候，只关注事件本身，而不关心谁会发布这个事件；发布者在发布事件的时候，只关注事件本身，而不关心谁订阅了这个事件。

观察者模式有两个重要的角色，即目标和观察者。在目标和观察者之间是没有事件通道的。一方面，观察者要想订阅目标事件，由于没有事件通道，因此必须将自己添加到目标(Subject) 中进行管理；另一方面，目标在触发事件的时候，也无法将通知操作(notify) 委托给事件通道，因此只能亲自去通知所有的观察者。

**观察者模式，其实就是为了实现松耦合(loosely coupled)。**（注：两个模块，A模块和B模块，当两者的关联非常多的时候，就叫紧耦合，反之，则是松耦合。）

**发布订阅模式里，发布者和订阅者，不是松耦合，而是完全解耦的。**

node.js中的event模块就是发布-订阅模式的应用

###### 发布-订阅模式
```javascript
class EventEmitter {
  #events = Object.create(null);
  on(type, listener) {
    this.#events[type] = this.#events[type] || [];
    this.#events[type].push(listener);
    return this;
  }
  off(type, listener) {
    if (!Reflect.has(this.#events, type)) return false;
    this.#events[type] = this.#events[type].filter(handle => handle !== listener);
    return true;
  }
  emit(type, ...args) {
    if (!Reflect.has(this.#events, type)) return false;
    this.#events[type].forEach(event => Reflect.apply(event, this, args));
    return true;
  }
  once(type, listener) {
    const callback = (...args) => {
      Reflect.apply(listener, this, args);
      this.off(type, callback);
    }
    this.on(type, callback);
  }
}

// 创建事件调度中心，为订阅者和发布者提供调度服务
let event = new EventEmitter();

// A订阅了SMS事件（A只关注SMS本身，而不关心谁发布这个事件）
event.on('SMS', console.log);

// B订阅了SMS事件
event.on('SMS', console.log);

// C发布了SMS事件（C只关注SMS本身，不关心谁订阅了这个事件）
event.emit('SMS','I published `SMS` event');
```

###### 观察者模式

```javascript
class Subject {
  #observers = [];
  add(observer) {
    this.#observers.push(observer);
  }
  notify(...args) {
    this.#observers.forEach(observer => observer.update(...args));
  }
}

class Observer {
  update(...args) {
    console.log(...args);
  }
}


// 创建观察者ob1
let ob1 =new Observer();
// 创建观察者ob2
let ob2 =new Observer();
// 创建目标sub
let sub =new Subject();
// 目标sub添加观察者ob1 （目标和观察者建立了依赖关系）
sub.add(ob1);
// 目标sub添加观察者ob2
sub.add(ob2);
// 目标sub触发SMS事件（目标主动通知观察者）
sub.notify('I fired `SMS` event');
```

**发布-订阅模式是面向调度中心编程的，而观察者模式则是面向目标和观察者编程的。前者用于解耦发布者和订阅者，后者用于耦合目标和观察者**。相对来说，发布-订阅模式能够根据不同主题来添加订阅者，从而实现更为颗粒度的控制。

#### 发布-订阅模式和观察者模式使用

* 发布订阅模式，更多的是一种跨应用的模式(cross-application pattern)，比如我们常用的消息中间件。 例如node的EventEmitter。
* 观察者模式，多用于单个应用内部

#### 发布-订阅模式和观察者模式的优缺点

* 发布-订阅模式有点事灵活，但是容易造成数据流混乱，不好维护
* 观察者模式最大的有点就是其响应式，但是不够灵活。

###### 发布-订阅模式也是自定义事件，它和DOM编程中的事件有什么区别
主要区别就是，发布-订阅模式的事件是手动触发的，而DOM编程的中的事件是自动触发的。
还有就是发布-订阅模式的事件可以自定义，而DOM编程中的事件是固定的几个(click, contextmenu, dbclick, mousedown, mouseup, mouseenter, mouseleave, mousemove, mouseout, mouseover, touch)
