#### 1. 事件委托的好处

* 提高性能（减少事件注册，节省内存）
* 新添加的元素还会有之前的事件

#### 2. flex-basis 和 width 的区别

flex-basis 属性定义了在分配多余空间之前，项目占据的主轴空间（main size）。浏览器根据这个属性，计算主轴是否有多余空间。它的默认值为 auto ，即项目的本来大小。

`content –> width –> flex-basis (limted by max|min-width)`

* 如果没有设置 flex-basis 属性，那么 flex-basis 的大小就是项目的 width 属性的大小
* 如果没有设置 width 属性，那么 flex-basis 的大小就是项目内容 (content) 的大小

可以使用 max-width / min-width 来限制 flex-basis。

#### 3. ES6 为什么要引入 Symbol 值

ES5 的对象属性名都是字符串，这容易造成属性名的冲突。

如果有一种机制，保证每个属性的名字都是独一无二的就好了，这样就从根本上防止属性名的冲突。这就是 ES6 引入Symbol的原因。

ES6 引入了一种新的原始数据类型Symbol，表示独一无二的值。

#### 4. 文件上传的二进制怎么处理

可以使用 FormData 对象上传二进制文件，FormData 的最大优点就是我们可以异步上传二进制文件。

如果送出时的编码类型被设为 "multipart/form-data"，它会使用和表单一样的格式。

```javascript
var formData = new FormData();
formData.append("username", "Groucho");
formData.append("accountnum", 123456); // 数字 123456 会被立即转换成字符串 "123456"

// HTML 文件类型input，由用户选择
formData.append("userfile", fileInputElement.files[0]);

// JavaScript file-like 对象var content = '<a id="a"><b id="b">hey!</b></a>'; // 新文件的正文...var blob = new Blob([content], { type: "text/xml"});

formData.append("webmasterfile", blob);

var request = new XMLHttpRequest();
request.open("POST", "http://foo.com/submitform.php");
request.send(formData);
```

#### 5. 如何做一个上传图片预览效果

```html
<style>
.box{
  width: 300px;
  height: 300px;
  position: relative;
  background-color: #eee;
}
.image{
  width: 100%;
  height: 100%;
}
#file{
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  opacity: 0;
}
</style>
<div class="box">
  <img class="image" />
  <input id="file" type="file">
</div>
```
```javascript
document.querySelector('#file').addEventListener('change', function (e){
  document.querySelector('.image').setAttribute('src', URL.createObjectURL(e.target.files[0]));
})
```

###### URL.createObjectURL(object)

object 用于创建 URL 的 File 对象、Blob 对象或者 MediaSource 对象。​

URL.createObjectURL() 静态方法会创建一个 DOMString，其中包含一个表示参数中给出的对象的URL。

也可以用于创建文件内的web worker

```javascript
const createWorker = function (f, options){
  const blob = new Blob(['(' + f.toString() + ')()']);
  const url = window.URL.createObjectURL(blob);
  const worker = new Worker(url, options);
  return worker;
}
```

###### DOMString

DOMString 是一个UTF-16字符串。由于JavaScript已经使用了这样的字符串，所以DOMString 直接映射到 一个String。

###### Blob

[更多Blob参考](https://www.jianshu.com/p/b322c2d5d778)

一个Blob对象就是一个包含有只读原始数据的类文件对象。Blob对象中的数据并不一定得是JavaScript中的原生形式。File接口基于Blob, 继承了Blob的功能，并且扩展支持了用户计算机上的本地文件。

创建Blob对象的方法有几种，可以调用Blob构造函数，还可以使用一个已有Blob对象上的slice()方法切出另一个Blob对象，还可以调用canvas对象上的toBlob方法。

实际上，Blob是计算机界通用术语之一，全称写作：BLOB (binary large object)，表示二进制大对象。MySql/Oracle数据库中，就有一种Blob类型，专门存放二进制数据。

在实际Web应用中，Blob更多是图片二进制形式的上传与下载，虽然其可以实现几乎任意文件的二进制传输。

构造函数 `Blob( [[Array parts], BlobPropertyBag properties] );`

```javascript
File.prototype instanceof Blob   //true   File也是继承自Blob
```

**parts** 一个数组，包含了将要添加到Blob对象中的数据。数组元素可以是任意多个的ArrayBuffer, ArrayBufferView(typed array), Blob, 或者DOMString对象。

**properties** 一个对象，设置Blob对象的一些属性。目前仅支持一个type属性，表示Blob的类型。


#### 6. `application/x-www-form-urlencoded` `multipart/form-data` `application/json` `text/xml` 的区别

###### `application/x-www-form-urlencoded`

最常见的 POST 提交数据的方式了。浏览器的原生 form 表单，如果不设置 enctype 属性，那么最终就会以 `application/x-www-form-urlencoded` 方式提交数据。

```javascript
POST http: //www .example.com HTTP /1 .1
Content-Type: application /x-www-form-urlencoded ;charset=utf-8
title= test &sub%5B%5D=1&sub%5B%5D=2&sub%5B%5D=3
```

提交的数据按照 `key1=val1&key2=val2` 的方式进行编码，key 和 val 都进行了 URL 转码。大部分服务端语言都对这种方式有很好的支持。

很多时候，我们用 Ajax 提交数据时，也是使用这种方式。例如 JQuery 的 Ajax，Content-Type 默认值都是`application/x-www-form-urlencoded;charset=utf-8`。

###### multipart/form-data

是一个常见的 POST 数据提交的方式。我们使用表单上传文件时，必须让 form 的 enctyped 等于这个值。

```javascript
POST http: //www .example.com HTTP /1 .1
Content-Type:multipart /form-data ; boundary=----WebKitFormBoundaryrGKCBY7qhFd3TrwA
------WebKitFormBoundaryrGKCBY7qhFd3TrwA
Content-Disposition: form-data; name= "text"
title
------WebKitFormBoundaryrGKCBY7qhFd3TrwA
Content-Disposition: form-data; name= "file" ; filename= "chrome.png"
Content-Type: image /png
PNG ... content of chrome.png ...
------WebKitFormBoundaryrGKCBY7qhFd3TrwA--
```

生成了一个 boundary 用于分割不同的字段，为了避免与正文内容重复，boundary 很长很复杂。然后 Content-Type 里指明了数据是以 mutipart/form-data 来编码，本次请求的 boundary 是什么内容。消息主体里按照字段个数又分为多个结构类似的部分，每部分都是以 –boundary 开始，紧接着内容描述信息，然后是回车，最后是字段具体内容（文本或二进制）。如果传输的是文件，还要包含文件名和文件类型信息。消息主体最后以 –boundary– 标示结束。

这种方式一般用来上传文件，各大服务端语言对它也有着良好的支持。

**上面提到的这两种 POST 数据的方式，都是浏览器原生支持的，而且现阶段原生 form 表单也只支持这两种方式。**

###### application/json

告诉服务端消息主体是序列化后的 JSON 字符串。由于 JSON 规范的流行，除了低版本 IE 之外的各大浏览器都原生支持 JSON.stringify，服务端语言也都有处理 JSON 的函数，使用 JSON 不会遇上什么麻烦。

```javascript
POST http: //www .example.com HTTP /1 .1
Content-Type: application /json ;charset=utf-8
{ "title" : "test" , "sub" :[1,2,3]}
```

JSON 格式支持比键值对复杂得多的结构化数据，这一点也很有用。

###### text/xml

它是一种使用 HTTP 作为传输协议，XML 作为编码方式的远程调用规范。

```javascript
POST http: //www .example.com HTTP /1 .1
Content-Type: text /xml
<?xml version= "1.0" ?>
<methodCall>
     <methodName>examples.getStateName< /methodName >
     <params>
         <param>
             <value><i4>41< /i4 >< /value >
         < /param >
     < /params >
< /methodCall >
```

#### 7. 事件处理函数中的 this 指向 currentTarget 还是 target.

target 在事件流的目标阶段，而 currentTareget 在事件流的捕获、目标、冒泡阶段。只有当事件流处于目标阶段的时候时候，两者才是一致。

target 指向真正的事件源，而 currentTarget 则是指向经过捕获、冒泡阶段的父级元素而触发事件的事件源（触发的事件在哪个元素上绑定，则指向哪个元素，当然它可能是真正的事件源，也可能不是）。

在事件处理函数中（匿名函数，而非箭头函数，因为箭头函数没有this）的this，指向 currentTarget。

#### 8. react的合成事件

React 自己实现了这么一套事件机制，它在 DOM 事件体系基础上做了改进，减少了内存的消耗，并且最大程度上解决了 IE 等浏览器的不兼容问题。

* React 上注册的事件最终会绑定在document这个 DOM 上，而不是 React 组件对应的 DOM(减少内存开销就是因为所有的事件都绑定在 document 上，其他节点没有绑定事件)（从React17开始，注册事件最终会绑定到root节点上,并且移除了事件池,事件池并不会提高现代浏览器的性能，甚至还会使经验丰富的开发者一头雾水,比如异步访问不到事件对象）
* ~~React 自身实现了一套事件冒泡机制，所以这也就是为什么我们 event.stopPropagation() 无效的原因。~~
* React 通过队列的形式，从触发的组件向父组件回溯，然后调用他们 JSX 中定义的 callback
* React 有一套自己的合成事件 SyntheticEvent，不是原生的，这个可以自己去看官网
* ~~React 通过对象池的形式管理合成事件对象的创建和销毁，减少了垃圾的生成和新对象内存的分配，提高了性能~~
* SyntheticEvent 是合并而来。~~这意味着 SyntheticEvent 对象可能会被重用，而且在事件回调函数被调用后，所有的属性都会无效。出于性能考虑，你不能通过异步访问事件。(不能setTimeout去获取event属性)~~

[关于在异步操作中访问React事件对象的问题](https://www.cnblogs.com/geek1116/p/13657822.html)

### 9. setState真的是异步的么

1. setState 只在合成事件和钩子函数中是“异步”的，在原生事件和setTimeout 中都是同步的。
2. setState 的“异步”并不是说内部由异步代码实现，其实本身执行的过程和代码都是同步的，只是合成事件和钩子函数的调用顺序在更新之前，导致在合成事件和钩子函数中没法立马拿到更新后的值，形成了所谓的“异步”，当然可以通过第二个参数 setState(partialState, callback) 中的callback拿到更新后的结果。
3. setState 的批量更新优化也是建立在“异步”（合成事件、钩子函数）之上的，在原生事件和setTimeout 中不会批量更新，在“异步”中如果对同一个值进行多次setState，setState的批量更新策略会对其进行覆盖，取最后一次的执行，如果是同时setState多个不同的值，在更新时会对其进行合并批量更新
4. **(React 18重大改动)** 批处理是指 React 将多个状态更新，聚合到一次 render 中执行，以提升性能。在 React 18 之前，React 只会在事件回调中使用批处理，而在 Promise、setTimeout、原生事件等场景下，是不能使用批处理的。**而在 React 18 中，所有的状态更新，都会自动使用批处理，不关心场景。**如果你在某种场景下不想使用批处理，你可以通过 flushSync来强制同步执行

### 10. webpack 中 hash chunkhash contenthash 区别

**hash**
如果都使用hash的话，因为这是工程级别的，即每次修改任何一个文件，所有文件名的hash至都将改变。所以一旦修改了任何一个文件，整个项目的文件缓存都将失效。

**chunkhash**

chunkhash根据不同的入口文件(Entry)进行依赖文件解析、构建对应的chunk，生成对应的哈希值。在生产环境里把一些公共库和程序入口文件区分开，单独打包构建，接着我们采用chunkhash的方式生成哈希值，那么只要我们不改动公共库的代码，就可以保证其哈希值不会受影响。并且webpack4中支持了异步import功能，固，chunkhash也作用于此.

但是这样又有一个问题，因为我们是将样式作为模块import到JavaScript文件中的，所以它们的chunkhash是一致的，如test1.js和test1.css

**contenthash**

contenthash是针对文件内容级别的，只有你自己模块的内容变了，那么hash值才改变，所以我们可以通过contenthash解决上诉问题。

### 11. webpack 中 module chunk bundle 的区别是什么

**module**

对于一份同逻辑的代码，当我们手写下一个一个的文件，它们无论是 ESM 还是 commonJS 或是 AMD，他们都是 module ；

**chunk**

当我们写的 module 源文件传到 webpack 进行打包时，webpack 会根据文件引用关系生成 chunk 文件，webpack 会对这个 chunk 文件进行一些操作；

**bundle**

webpack 处理好 chunk 文件后，最后会输出 bundle 文件，这个 bundle 文件包含了经过加载和编译的最终源文件，所以它可以直接在浏览器中运行。(chunk 和 bundle 也不是一一对应的关系)

module，chunk 和 bundle 其实就是同一份逻辑代码在不同转换场景下的取了三个名字:

我们直接写出来的是 module，webpack 处理时是 chunk，最后生成浏览器可以直接运行的 bundle。

### 12. 发布-订阅模式 VS 观察者模式

发布-订阅模式是面向调度中心编程的，而观察者模式则是面向目标和观察者编程的。前者用于解耦发布者和订阅者，后者用于耦合目标和观察者。相对来说，发布-订阅模式能够根据不同主题来添加订阅者，从而实现更为颗粒度的控制。

```javascript
/*
* 发布-订阅
*/
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

/*
* 目标
*/
class Subject {
  #observers = new Set();
  add(observer) {
    this.#observers.add(observer);
  }
  notify(...args) {
    this.#observers.forEach(observer => observer.update(...args));
  }
}
// 观察者
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

### 13. webpack 相关问题总结

[webpack相关问题](https://mp.weixin.qq.com/s/2-zNlGrKUngWdQNvlcgESw)

### 14. AST

抽象语法树（Abstract Syntax Tree，AST），或简称语法树（Syntax tree），**是源代码语法结构的一种抽象表示**。它以树状的形式表现编程语言的语法结构，树上的每个节点都表示源代码中的一种结构。

就是按照某种约定好的规范，以树形的数据结构把我们的代码描述出来，让js引擎和转译器能够理解。

举个例子：就好比现在框架会利用虚拟dom这种方式把真实dom结构描述出来再进行操作一样，而对于更底层的代码来说，AST就是用来描述代码的好工具。

当然AST不是JS特有的，每个语言的代码都能转换成对应的AST, 并且AST结构的规范也有很多， js里所使用的规范大部分是 estree。

[在线生成AST网站](https://astexplorer.net/)

其中Babel就是利用AST对代码进行编译，过程大概是 **code转换为AST -> 处理AST -> AST转换为code**  也就是 **解析 -> 转换 -> 生成**

##### 解析过程

通过 parser 把源码转成抽象语法树（AST），分为两个阶段：词法分析(分词)、语法分析。

语法分析阶段的任务就是根据 tokens 生成 AST。它会对 tokens 进行遍历，最终按照特定的结构生成一个 tree 这个 tree 就是 AST。

### 15. JSBridge实现原理

[JSBridge实现原理](jianshu.com/p/43b45b687593)

### 16. React 相关问题总结

[react相关问题](https://mp.weixin.qq.com/s/Os4U_AZGJiflkVe1WRBt0Q)

### 17. git 操作

![git图片](https://github.com/fang-bin/interview/blob/master/image/git.jpg)

* `git reflog` 查看本地 git 命令历史

###### `git log` 查看提交历史

* `--oneline`
* `--grep="关键字"`

###### `git checkout` 撤销工作区的修改

##### `git reset` 可以撤销工作区/暂存区的修改

![git-reset图片](https://github.com/fang-bin/interview/blob/master/image/git-reset.jpg)

![git-reset-head图片](https://github.com/fang-bin/interview/blob/master/image/git-reset-head.jpg)

##### `git reset` 和 `git revert` 的区别

* `git revert` 后会多出一条commit，这里可进行回撤操作
* `git reset` 直接把之前 commit 删掉，非 `git reset --hard` 的操作是不会删掉修改代码，如果远程已经有之前代码，需要强推 `git push -f`

##### `git rebase` 和 `git merge` 的区别

###### git reabse

* 合并多次提交记录
* 分支合并

[git rebase 和 git merge 的区别](http://jartto.wang/2018/12/11/git-rebase/)

`git rebase` 合并 commit 记录，保持分支整洁.

`git rebase` 相比 `git merge` 来说会减少分支合并的记录，但多人开发的时候，也会有丢失提交纪录的风险，因为它改变了历史，应该谨慎使用。

只要你的分支上需要 `rebase` 的所有 commits 历史还没有被 push 过，就可以安全地使用 `git-rebase` 来操作。
### 18. typescript 中面向对象的 public private protected

TypeScript里，成员都默认为 public。

* public 没有限制
* protected 不能在声明它的类的外部访问，但是在派生类中仍然可以访问(可以被继承)
* private 不能在声明它的类的外部访问，不能在派生类中访问(不可被继承)

注意: 

TypeScript使用的是结构性类型系统。 当我们比较两种不同的类型时，并不在乎它们从何处而来，如果所有成员的类型都是兼容的，我们就认为它们的类型是兼容的。

然而，当我们比较带有 private或 protected成员的类型的时候，情况就不同了。 如果其中一个类型里包含一个 private成员，那么只有当另外一个类型中也存在这样一个 private成员， 并且**它们都是来自同一处声明时**，我们才认为这两个类型是兼容的。 对于 protected成员也使用这个规则。

### 19. super的作用

super这个关键字，既可以当作函数使用，也可以当作对象使用。

###### super作为函数调用时，代表父类的构造函数

```javascript
class A {}

class B extends A {
  constructor() {
    super();
  }
}
```

super虽然代表了父类A的构造函数，但是返回的是子类B的实例，即super内部的this指的是B的实例，因此super()在这里相当于A.prototype.constructor.call(this)。

```javascript
class A {
  constructor() {
    console.log(new.target.name);
  }
}
class B extends A {
  constructor() {
    super();
  }
}
new A() // A
new B() // B
```

作为函数时，super()只能用在子类的构造函数之中，用在其他地方就会报错。

###### super作为对象时，在普通方法中，指向父类的原型对象；在静态方法中，指向父类。

```javascript
class A {
  p() {
    return 2;
  }
}

class B extends A {
  constructor() {
    super();
    console.log(super.p()); // 2
  }
}

let b = new B();
```

这里需要注意，由于super指向父类的原型对象，所以定义在父类实例上的方法或属性，是无法通过super调用的。

```javascript
class A {
  constructor() {
    this.p = 2;
  }
}

class B extends A {
  get m() {
    return super.p;
  }
}

let b = new B();
b.m // undefined
```

ES6 规定，在子类普通方法中通过super调用父类的方法时，方法内部的this指向当前的子类实例。

由于this指向子类实例，所以如果通过super对某个属性赋值，这时super就是this，赋值的属性会变成子类实例的属性。

```javascript
class A {
  constructor() {
    this.x = 1;
  }
}

class B extends A {
  constructor() {
    super();
    this.x = 2;
    super.x = 3;
    console.log(super.x); // undefined
    console.log(this.x); // 3
  }
}

let b = new B();
```

如果super作为对象，用在静态方法之中，这时super将指向父类，而不是父类的原型对象。

```javascript
class Parent {
  static myMethod(msg) {
    console.log('static', msg);
  }

  myMethod(msg) {
    console.log('instance', msg);
  }
}

class Child extends Parent {
  static myMethod(msg) {
    super.myMethod(msg);
  }

  myMethod(msg) {
    super.myMethod(msg);
  }
}

Child.myMethod(1); // static 1

var child = new Child();
child.myMethod(2); // instance 2
```

另外，在子类的静态方法中通过super调用父类的方法时，方法内部的this指向当前的子类，而不是子类的实例。

```javascript
class A {
  constructor() {
    this.x = 1;
  }
  static print() {
    console.log(this.x);
  }
}

class B extends A {
  constructor() {
    super();
    this.x = 2;
  }
  static m() {
    super.print();
  }
}

B.x = 3;
B.m() // 3
```

使用super的时候，必须显式指定是作为函数、还是作为对象使用，否则会报错。

```javascript
class A {}

class B extends A {
  constructor() {
    super();
    console.log(super); // 报错
  }
}
```

### 20. postMessage

`otherWindow.postMessage(message, targetOrigin, [transfer])`

* otherWindow 其他窗口的一个引用
* message 发送的数据 它会被序列化(JSON.stringify，自动被序列化)
* targetOrigin 通过窗口的origin属性来指定哪些窗口能接收到消息事件，其值可以是字符串"*"（表示无限制）或者一个URI。
* transfer 可选，是一串和message 同时传递的 Transferable 对象. 这些对象的所有权将被转移给消息的接收方，而发送一方将不再保有所有权。

`window.addEventListener('message', receiveMessage, false)`

接收到的message对象的属性有：
* data 从其他 window 中传递过来的对象
* origin 调用 postMessage  时消息发送方窗口的 origin . 这个字符串由 协议、“://“、域名、“ : 端口号”拼接而成。
* source 对发送消息的窗口对象的引用; 您可以使用此来在具有不同origin的两个窗口之间建立双向通信。

##### 安全问题

如果不希望从其他网站接收message，不要为message事件添加任何事件侦听器。 这是一个完全万无一失的方式来避免安全问题。

如果确实希望从其他网站接收message，请始终使用origin和source属性验证发件人的身份。

当使用postMessage将数据发送到其他窗口时，始终指定精确的目标origin，而不是*。

### 21. 通信

##### 跨域iframe通信

* `postMessage`
* 片段标识符
* `document.domain` (一级域名相同情况下，设置)
* `window.name`

补充同域iframe通信，还有`sessionStorage` `localStorage`

##### 同域跨窗口通信

* `postMessage`
* 片段标识符
* Shared Worker
* Service Worker
* `localStorage`

##### 跨域跨窗口

* `postMessage`
* 片段标识符


